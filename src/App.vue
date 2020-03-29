<template>
  <v-app>
    <v-img
      :src="require('@/assets/img/bk-road-1.jpg')"
      :lazy-src="require('@/assets/img/bk-road-1.jpg')"
      :height="bgHeight"
    >
      <!-- <v-card
        color="transparent"
        class="overflow-y-auto"

      >-->
      <v-toolbar color="white" dense flat>
        <v-icon>mdi-calculator</v-icon>
        <v-icon>mdi-blank</v-icon>
        <v-toolbar-title>pacr.run</v-toolbar-title>
        <v-spacer></v-spacer>

        <v-toolbar-title>
          Calculate your running pace, time or distance</v-toolbar-title
        >
        <!-- 
        <v-btn href="https://github.com/vuetifyjs/vuetify/releases/latest" target="_blank" text>
        <span class="mr-2">Latest Release</span>
        <v-icon>mdi-open-in-new</v-icon>
      </v-btn>
      -->
      </v-toolbar>
      <v-content>
        <Pacr />
      </v-content>
    </v-img>
    <PacrFooter />
  </v-app>
</template>

<script>
import Pacr from "./components/Pacr";
import PacrFooter from "./components/Footer";

export default {
  name: "App",

  created() {
    document.title = "pacr.run - Calculate your running pace!";
  },

  components: {
    Pacr,
    PacrFooter
  },

  data: () => ({}),
  computed: {
    bgHeight() {
      return this.$vuetify.breakpoint.height;
    }
  }
};
</script>
