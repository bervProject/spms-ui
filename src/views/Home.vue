<template>
  <v-container>
    <v-btn :href="loginGithub" outlined
      ><v-icon left> mdi-github-circle </v-icon>Login with Github</v-btn
    >
    <v-btn :href="loginGoogle" outlined
      ><v-icon left> mdi-google </v-icon>Login with Google</v-btn
    >
  </v-container>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";

@Component({})
export default class Home extends Vue {
  baseURL = "https://spms-berv.herokuapp.com";
  loginGithub = `${this.baseURL}/oauth/github`;
  loginGoogle = `${this.baseURL}/oauth/google`;
}
</script>
