<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <v-icon>mdi-calculator</v-icon>
      <v-icon>mdi-blank</v-icon>
      <v-toolbar-title>pacr.run</v-toolbar-title>
      <v-spacer></v-spacer>

      <v-toolbar-title>Calculate your running pace, time or distance</v-toolbar-title>
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
  created() {
    document.title = "pacr.run - Calculate your running pace!";
  },
  components: { Pacr, PacrFooter },

  data: () => ({
    //
  })
};
</script>
