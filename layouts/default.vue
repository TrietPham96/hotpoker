<template>
  <v-app dark>
    <v-app-bar :clipped-left="clipped" fixed app>
      <v-btn text @click="goHome">{{ title }}</v-btn>
      <v-spacer />
      <v-toolbar-title v-text="currentPath" v-if="isDisplayPath" />
      <v-btn
        color="info"
        @click="copyURLToClipBoard"
        v-if="isDisplayPath"
        class="ma-2 white--text"
      >
        <v-icon dark> mdi-content-copy </v-icon>
        Copy URL
      </v-btn>
    </v-app-bar>
    <v-main>
      <v-container>
        <Nuxt />
      </v-container>
    </v-main>
    <v-footer :absolute="!fixed" app>
      <span>&copy; {{ new Date().getFullYear() }}</span>
    </v-footer>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      clipped: false,
      fixed: false,
      title: "Poker Planning",
      currentPath: "",
      isDisplayPath: false,
    };
  },
  created() {
    this.$nuxt.$on("roomInfo", (data) => {
      this.currentPath = window.location.href + "room/" + data.RoomId;
      this.isDisplayPath = true;
    });
  },
  mounted() {
    this.currentPath = window.location.href;
    if (!(this.currentPath.indexOf("room") == -1)) {
      this.isDisplayPath = true;
    }
  },

  methods: {
    async copyURLToClipBoard() {
      await navigator.clipboard.writeText(this.currentPath);
      // this.copyText = "Copied";
    },
    goHome() {
      window.location = "/";
    },
  },
};
</script>
