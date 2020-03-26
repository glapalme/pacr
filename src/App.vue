<template>
  <v-app>
    <v-app-bar app color="primary" dark height="55">
      <div class="d-flex align-center">
        <!--<v-img
          alt="Vuetify Logo"
          class="shrink mr-2"
          contain
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-logo-dark.png"
          transition="scale-transition"
          width="40"
        />-->
        <!--
        <v-img
          alt="Vuetify Name"
          class="shrink mt-1 hidden-sm-and-down"
          contain
          min-width="100"
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-name-dark.png"
          width="100"
        />-->
        <h2>Calculate your running pace, time or distance!</h2>
      </div>

      <v-spacer></v-spacer>
      <!-- 
        <v-btn href="https://github.com/vuetifyjs/vuetify/releases/latest" target="_blank" text>
        <span class="mr-2">Latest Release</span>
        <v-icon>mdi-open-in-new</v-icon>
      </v-btn>
      -->
    </v-app-bar>
    <v-content>
      <Pacr />
    </v-content>
    <PacrFooter />
  </v-app>
</template>

<script>
import Pacr from "./components/Pacr";
import PacrFooter from "./components/Footer";

export default {
  name: "App",

  components: {
    Pacr,
    PacrFooter
  },

  data: () => ({
    //
  })
};
</script>
