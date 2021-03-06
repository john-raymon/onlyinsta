<template>
  <div class="py-6 mx-auto text-white w-full px-2 md:px-4">
    <div v-if="creator" class="flex justify-between items-center py-2 pb-4 mb-4 mx-auto space-x-2">
      <div class="flex items-center flex-wrap md:space-x-4">
        <router-link
          :to="`/creator/${creator.id}`"
          class="relative w-32 h-auto rounded-full mb-2 mr-2"
        >
          <div class="relative w-full padding-bottom-full rounded-full overflow-hidden">
            <img
              v-if="creator.imageFile"
              class="absolute w-full h-full object-cover"
              :src="creator.imageFile.url"
            />
            <span
              v-else
              class="absolute uppercase flex items-center justify-center w-full h-full text-black bg-red-200"
            >
              {{ creator.username[0] }}
              {{ creator.username[1] }}
            </span>
          </div>
        </router-link>
        <p class="text-red-200 text-sm md:text-lg">
          @<span class="capitalize">{{ creator.username }}</span>
        </p>
      </div>
      <div>
        <div class="flex space-x-3 text-red-100 text-sm md:text-lg justify-end py-2">
          <p v-if="postCount !== null" class="text-4xl leading-none font-bold text-center">
            {{ postCount }}
            <span class="text-xs text-center block font-light">
              Posts
            </span>
          </p>
          <template v-if="stats">
            <router-link to="/subscribers" class="text-4xl leading-none font-bold text-center">
              {{ stats.followers.length }}
              <span class="text-xs text-center block font-light">
                Subscribers
              </span>
            </router-link>
            <router-link to="/following" class="text-4xl leading-none font-bold text-center">
              {{ stats.following.length }}
              <span class="text-xs text-center block font-light">
                Following
              </span>
            </router-link>
          </template>
        </div>
        <p v-if="stats" class="text-xs text-center py-2 opacity-50">
          (only you can see this)
        </p>
        <p v-if="following && subscription" class="text-red-300 text-sm md:text-base">
          <template v-if="following && subscription">
            Your monthly subscription expires on:
            <span class="capitalize">{{ formattedExpDate }}</span>
          </template>
        </p>
      </div>
    </div>
    <template v-if="creatorNotFound">
     <p class="text-white text-3xl text-center my-4">
        Sorry, this creator page isn't available.
      </p>
    </template>
    <template v-else-if="!userAuth.user">
      <div class="flex flex-col max-w-screen-md mx-auto">
        <p class="text-white text-3xl text-center mb-2">
          <router-link
          :to="signInRoute"
          class="underline">Sign in</router-link>
          or
          <router-link class="underline" :to="signUpRoute">Sign up</router-link>
          to follow {{ creator && creator.username }}'s page.
          <span class="block text-white opacity-75 text-sm">
            (We'll redirect you back here after)
          </span>
        </p>
      </div>
    </template>
    <template v-else>
      <ul v-if="following" class="flex flex-wrap w-full">
        <li
          class="w-1/3 cursor-pointer hover:opacity-75"
          v-for="post in computedPosts"
          :key="post._id"
        >
          <div class="p-1 md:p-4">
            <router-link
              :to="`/posts/${post._id}`"
              class="block relative padding-bottom-full overflow-hidden"
            >
              <div
                v-if="post.file.contentType.split('/')[0] === 'image'"
                class="w-full h-full absolute top-0 left-0 flex items-center justify-center"
              >
                <img
                  class="absolute w-full object-cover object-center opacity-75"
                  :src="post.url"
                />
              </div>
              <div
                v-else
                class="w-full h-full absolute top-0 left-0 flex items-center justify-center"
              >
                <img
                  class="absolute h-full w-full object-cover object-center opacity-75"
                  :src="post.thumbnailUrl"
                />
                <div
                  class="absolute left-0 top-0 h-full w-full mx-auto flex flex-col items-center justify-center opacity-75"
                >
                  <div class="w-1/4 h-auto fill-current cursor-pointer hover:opacity-50">
                    <PlayIconSvg />
                  </div>
                </div>
              </div>
            </router-link>
          </div>
        </li>
      </ul>
      <div v-else-if="userAuth.isAuth" class="flex flex-col max-w-screen-md mx-auto">
        <p class="text-white text-3xl text-center mb-2">
          Pay to unlock {{ creator && creator.username }}'s page
        </p>
        <!-- Stripe.js handles populating card-element-->
        <div class="p-5 bg-gray-900 rounded-lg mb-4">
          <div id="card-element"></div>
        </div>
        <button
          @click="subscribeToCreator"
          v-show="monthlySubscriptionPrice"
          class="text-red-800 font-medium text-md bg-white py-3 px-6 outline-none focus:outline-none focus:bg-red-100 hover:bg-red-100 mx-auto rounded-full"
        >
          {{ `Subscribe now for ${(monthlySubscriptionPrice / 100).toFixed(2)}/month` }}
        </button>
        <span class="text-xs text-white text-center block my-4 leading-none"
          >(no automatic renewals)</span
        >
      </div>
    </template>
  </div>
</template>
<script>
import { mapActions, mapState } from "vuex";
import { loadStripe } from "@stripe/stripe-js";
import PlayIconSvg from "@/assets/svgs/play-icon-svg.svg";

export default {
  name: "CreatorProfilePage",
  data() {
    return {
      following: false,
      creator: null,
      posts: [],
      postCount: null,
      subscribeError: "",
      subscription: null,
      stats: null,
      creatorNotFound: false,
      monthlySubscriptionPrice: null,
    };
  },
  components: {
    PlayIconSvg
  },
  // handles setting Stripe.js #card-element when mounted on DOM
  mounted() {
    this.setUpStripeElementCard();
  },
  created() {
    // fetch post feed
    this.fetchPost();
  },
  computed: {
    ...mapState(["userAuth"]),
    signInRoute() {
      return {
        name: "sign-in",
        query: { ...this.$route.query, redirect: this.$route.name, redirectParams: JSON.stringify(this.$route.params) }
      };
    },
    signUpRoute() {
      return {
        name: "sign-up",
        query: { ...this.$route.query, redirect: this.$route.name, redirectParams: JSON.stringify(this.$route.params) }
      };
    },
    formattedExpDate() {
      const options = { year: "numeric", month: "long", day: "numeric" };
      return new Date(this.subscription.expires).toLocaleDateString(undefined, options);
    },
    computedPosts() {
      const posts = [...this.posts];
      return posts.reverse();
    }
  },
  methods: {
    ...mapActions(["logout"]),
    fetchPost() {
      // fetch post feed
      this.$http
        ._get(`/users/${this.$route.params.id}/posts`)
        .then(
          function(res) {
            if (res.following) {
              this.following = res.following;
              this.creator = res.creator;
              this.subscription = res.subscription;
              this.monthlySubscriptionPrice = res.monthlySubscriptionPriceInCents;
              this.posts = res.posts;
              this.postCount = res.postCount;
              // TODO: store this date globally, so that we can reuse the other computed data dervived
              // from this for other views, instead of making the same request against
              // check if logged-in user's creator-profile, if so, fetch follower, following count
              if (this.userAuth.user && this.userAuth.user.id === res.creator.id) {
                // fetch follower count;
                return this.$http._get("/users/followers-following").then(res => {
                  this.stats = {
                    following: res.following,
                    followers: res.followers
                  };
                });
              }
              return;
            }
            // not following, so set creator data, and allow UI to show
            // subscribe form to follow
            this.creator = res.creator;
            this.following = res.following;
            this.postCount = res.postCount;
            this.monthlySubscriptionPrice = res.monthlySubscriptionPriceInCents || '0';
          }.bind(this)
        )
        .catch(error => {
          this.creatorNotFound = true;
          // catch unauthenticated error logging out user
          console.log("error while fetch post feed");
        });
    },
    subscribeToCreator() {
      this.subscribeError = "";
      this.$stripe
        .createToken(this.$stripeCard)
        .then(result => {
          if (result.error) {
            throw result.error;
          }
          const stripeCardToken = result.token.id;
          const createUserId = this.$route.params.id;
          return this.$http
            ._post(`/users/${createUserId}/subscribe`, { stripeCardToken })
            .then(res => {
              if (res.success) {
                return this.fetchPost();
              }
            });
          // continue and make request to subscribe endpoint
        })
        .catch(error => {
          if (error && error.response && error.response.status === 401) {
            this.logout();
          }
          console.log("there was an error", error);
        });
    },
    // set up Stripe.js card element
    setUpStripeElementCard() {
      loadStripe(process.env.VUE_APP_STRIPE_PUBLISHABLE_KEY).then(stripe => {
        this.$stripe = stripe;
        // Create an instance of Elements
        const elements = stripe.elements();

        // Custom styling can be passed to options when creating an Element.
        // (Note that this demo uses a wider set of styles than the guide below.)
        const style = {
          base: {
            color: "white",
            lineHeight: "18px",
            fontFamily: '"El Messiri"',
            fontSmoothing: "antialiased",
            fontSize: "1.2rem",
            "::placeholder": {
              color: "#aab7c4"
            }
          },
          invalid: {
            color: "#fa755a",
            iconColor: "#fa755a"
          }
        };

        // Create an instance of the card Element
        this.$stripeCard = elements.create("card", { style });

        // Add an instance of the card Element into the `card-element` <div>
        this.$stripeCard.mount("#card-element");
      });
    }
  }
};
</script>
