<template>
  <div class="pb-6 w-full flex flex-col items-center mt-4">
    <ul class="space-y-6 max-w-lg w-full" v-if="postFeed && postFeed.length">
      <li v-for="post in postFeed" :key="post._id" class="w-full">
        <!-- <router-link
          :to="`/posts/${post._id}`"
          class="card-post flex flex-col bg-red-100 rounded-lg w-full"
        >
          <div class="w-full overflow-hidden rounded-lg bg-black shadow-lg">
            <img
              v-if="post.file.contentType.split('/')[0] === 'image'"
              :src="post.url"
              width="100%"
            />
            <div v-else class="w-full h-40 relative">
              <div class="w-full h-full absolute top-0 left-0 flex items-center justify-center">
                <img
                  class="absolute h-full w-full object-cover object-center opacity-50"
                  :src="post.thumbnailUrl"
                />
                <div
                  class="absolute left-0 top-0 h-full w-full mx-auto flex flex-col items-center text-white justify-center opacity-75"
                >
                  <div class="w-1/6 h-auto fill-current cursor-pointer hover:opacity-50">
                    <PlayIconSvg />
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="flex items-center justify-between px-6 md:px-8 py-2 w-full">
            <p class="text-black text-sm pr-4">
              {{ post.description }}
            </p>
            <div class="flex items-center self-end">
              <p class="mr-2 font-light text-xs">
                Created by
              </p>
              <router-link
                :to="`/creator/${post.user._id}`"
                class="relative w-12 h-auto bg-red-200 rounded-full"
              >
                <div
                  class="relative w-full padding-bottom-full rounded-full bg-red-200 overflow-hidden"
                >
                  <img
                    v-if="post.user.imageFile"
                    class="absolute w-full h-full object-cover"
                    :src="post.user.imageFile.url"
                  />
                  <span
                    v-else
                    class="absolute uppercase flex items-center justify-center w-full h-full text-black"
                  >
                    {{ post.user.username[0] }}
                    {{ post.user.username[1] }}
                  </span>
                </div>
              </router-link>
            </div>
          </div>
        </router-link> -->
        <TimelinePost :post="post" />
      </li>
    </ul>
    <div v-else class='mx-auto text-center text-white flex flex-col items-center py-6'>
      <div class='w-10 h-10 text-white fill-current'>
        <InfoIconSvg />
      </div>
      <h2>
        No posts available.
      </h2>
    </div>
  </div>
</template>
<script>
import { mapActions } from "vuex";

import TimelinePost from "@/components/TimelinePost";
import InfoIconSvg from "@/assets/svgs/info-icon-svg.svg";

export default {
  name: "TimelineHomepage",
  data() {
    return {
      postFeed: []
    };
  },
  components: {
    TimelinePost,
    InfoIconSvg,
  },
  created() {
    // fetch post feed
    this.$http
      ._get("/posts/feed")
      .then(res => {
        if (res.success) {
          this.postFeed = res.postFeed.filter(p => !p.deleted);
        }
      })
      .catch(
        function(error) {
          if (error && error.response && error.response.status === 401) {
            this.logout();
          }
          // this.$emit('log-out');
          // catch unauthenticated error logging out user
          console.log("error while fetch post feed");
        }.bind(this)
      );
  },
  methods: {
    ...mapActions(["logout"])
  }
};
</script>
<style lang="scss">
.homepage {
  max-height: 80vh;
}
</style>
