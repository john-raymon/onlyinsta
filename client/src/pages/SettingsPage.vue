<template>
  <div class="settings-page">
    <h1 class="text-white">
      Settings
    </h1>
    <ul class="w-full space-y-4 text-base tracking-wider text-white">
      <li tabIndex="0">
        <router-link to="/settings/pay" class="settings-item">
          <div class="flex items-center">
            <p>
              {{
                userAuth.user.hasConnectedToStripe
                  ? "Manage your revenue & payments"
                  : "Set-up your account for pay-outs"
              }}
            </p>
          </div>
          <div class="w-6 h-6">
            <RightArrowIcon />
          </div>
        </router-link>
      </li>
      <li tabIndex="0">
        <router-link to="/settings/set-subscription-price" class="settings-item">
          <div class="flex items-center">
            <div class="w-5 h-5 mr-4">
              <MoneyIconSvg />
            </div>
            <p>
              Set your subscription price
            </p>
          </div>
          <div class="w-6 h-6">
            <RightArrowIcon />
          </div>
        </router-link>
      </li>
      <li tabIndex="0">
        <router-link to="/settings/password" class="settings-item">
          <div class="flex items-center">
            <div class="w-5 h-5 mr-4">
              <KeyIconSvg />
            </div>
            <p>
              Change your password
            </p>
          </div>
          <div class="w-6 h-6">
            <RightArrowIcon />
          </div>
        </router-link>
      </li>
      <li tabIndex="0">
        <router-link to="/settings/account" class="settings-item">
          <div class="flex items-center">
            <div class="w-5 h-5 mr-4">
              <HammerIconSvg />
            </div>
            <p>
              Update your account information
            </p>
          </div>
          <div class="w-6 h-6">
            <RightArrowIcon />
          </div>
        </router-link>
      </li>
      <li tabIndex="0">
        <div class="settings-item">
          <div class="flex items-center">
            <div class="w-6 h-6 mr-4">
              <SadFaceIconSvg class="fill-current text-white" />
            </div>
            <p>
              Deactivate your account
            </p>
          </div>
          <div class="w-6 h-6">
            <RightArrowIcon />
          </div>
        </div>
      </li>
    </ul>
  </div>
</template>
<script>
import { mapState } from "vuex";
import KeyIconSvg from "@/assets/svgs/key-icon-svg.svg";
import RightArrowIcon from "@/assets/svgs/right-arrow-icon-svg.svg";
import HammerIconSvg from "@/assets/svgs/hammer-icon-svg.svg";
import SadFaceIconSvg from "@/assets/svgs/sad-face-icon.svg";
import MoneyIconSvg from "@/assets/svgs/money-icon-svg.svg";

export default {
  name: "SettingsPage",
  components: {
    KeyIconSvg,
    RightArrowIcon,
    HammerIconSvg,
    SadFaceIconSvg,
    MoneyIconSvg,
  },
  computed: {
    ...mapState(["userAuth"])
  }
};
</script>
<style lang="scss">
.settings-page {
  @apply w-full;

  .settings-item {
    @apply flex justify-between items-center border-b border-gray-600 w-full py-6 cursor-pointer;
  }
}
</style>
