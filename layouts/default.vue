<template>
  <!-- <div v-if="web"> -->
  <div v-if="true">
    <NavBar />
    <div
      v-if="userProfile && (!userProfile.phone || !userProfile.lineId)"
      class="bg-red-300 text-red-700 p-2 text-sm"
    >
      請記得更新你的資料，才能夠配對得更準確，更容易應徵成功喔！
    </div>
    <Nuxt />
    <Footer />
  </div>
  <div
    v-else
    class="flex flex-col w-screen justify-center items-center p-10"
  >
    <img src="~/assets/line_account_qrcode.png" />
    <p class="text-gray-600 mt-3">
      請加入「<span class="text-green-500">{{ process.env.prjname }}</span
      >」LINE好友
    </p>
    <p class="text-green-500">LINE ID: @832gobav</p>
  </div>
</template>

<script>
const LIFF_ID = process.env.LIFF_ID;
export default {
  head: {
    title: process.env.prjname,
    script: [
      { src: "https://static.line-scdn.net/liff/edge/2/sdk.js", body: true },
      {
        src: "https://kit.fontawesome.com/d1105f8ef3.js",
        crossorigin: "anonymous",
      },
    ],
    meta: [
      { name: "viewport", content: "initial-scale=1.0, width=device-width" },
    ],
  },
  computed: {
    isWeb() {
      if (process.env.dev) return false;
      return this.$store.state.OS === "web";
    },
    userProfile() {
      return this.$store.state.userProfile;
    },
  },
  mounted() {
    this.$store.commit("setOS", liff);
    // initialize LIFF
    liff
      .init({
        liffId: LIFF_ID,
      })
      .then(async () => {
        // handle login
        if (liff.isLoggedIn()) {
          this.$store.commit("setUserIdToken", await liff.getProfile());
          if (!this.$store.state.userProfile) {
            this.$store.commit(
              "setUserProfile",
              await fetch("/api/crud_profile", {
                method: "POST",
                body: JSON.stringify({
                  action: "create",
                  userIdToken: this.$store.state.userIdToken.userId,
                  displayName: this.$store.state.userIdToken.displayName,
                  image: this.$store.state.userIdToken.pictureUrl
                    ? this.$store.state.userIdToken.pictureUrl
                    : "",
                }),
              }).then((res) => res.json())
            );
          }
        }
      })
      .catch((err) => {
        // do something when error is catched
      });
  },
};
</script>

<style>
body {
  background-color: #e9ecf0;
}
</style>