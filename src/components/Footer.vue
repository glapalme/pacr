<template>
  <v-footer height="50" dark>
    <v-row align="center" class="ml-2">
      <div class="overline text-color:deep-purple ">
        <i>More than just a finish line</i> - Rush
      </div>
      <v-spacer></v-spacer>
      <v-btn
        v-for="(icon, index) in icons"
        :key="index"
        target="_blank"
        :href="icon.link"
        class="mx-4"
        icon
      >
        <v-icon size="24px">{{ icon.name }}</v-icon>
      </v-btn>
    </v-row>
  </v-footer>
</template>

<script>
export default {
  name: "PacrFooter",
  data: () => ({
    icons: [{ name: "mdi-github", link: "https://github.com/glapalme/pacr" }]
    //icons: ["mdi-github", "mdi-linkedin", "mdi-twitter"]
  })
};
</script>
