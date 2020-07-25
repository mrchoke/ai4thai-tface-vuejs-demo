<template>
  <v-app>
    <v-app-bar app color="#043c3b" dark dense>
      <div class="d-flex align-center">
        <v-img
          alt="AI4Thai"
          class="shrink mr-2"
          contain
          src="@/assets/logo.png"
          transition="scale-transition"
          width="40"
        />
        T-Face
      </div>

      <v-spacer></v-spacer>

      <v-btn href="https://aiforthai.in.th" target="_blank" text>
        <span class="mr-2">Read More</span>
        <v-icon>mdi-open-in-new</v-icon>
      </v-btn>
    </v-app-bar>

    <v-main>
      <TFace />
    </v-main>
    <v-footer padless height="45">
      <v-col class="text-center" cols="12">
        <p>Copyright {{ Copyright }}. AI For Thai</p>
      </v-col>
    </v-footer>
  </v-app>
</template>

<script>
import TFace from "./components/TFace";

export default {
  name: "App",

  components: {
    TFace
  },

  data: () => ({
    //
  }),
  computed: {
    Copyright() {
      return new Date().getFullYear();
    }
  }
};
</script>
