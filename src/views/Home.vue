<template>
  <v-container>
    <v-card :loading="submitting">
      <v-card-title> Login </v-card-title>
      <v-card-text>
        <v-container>
          <v-row align="center" class="mx-0">
            <validation-observer ref="observer" v-slot="{ invalid }">
              <form @submit.prevent="submit">
                <v-row>
                  <validation-provider
                    v-slot="{ errors }"
                    name="Username"
                    rules="required|email"
                  >
                    <v-text-field
                      v-model="username"
                      :error-messages="errors"
                      label="Username"
                      :disabled="submitting"
                      required
                      clearable
                      prepend-icon="mdi-account"
                    ></v-text-field>
                  </validation-provider>
                </v-row>
                <v-row>
                  <validation-provider
                    v-slot="{ errors }"
                    name="Password"
                    rules="required"
                  >
                    <v-text-field
                      v-model="password"
                      :append-icon="showPassword ? 'mdi-eye' : 'mdi-eye-off'"
                      :type="showPassword ? 'text' : 'password'"
                      label="Password"
                      required
                      :disabled="submitting"
                      :error-messages="errors"
                      prepend-icon="mdi-lock"
                      @click:append="showPassword = !showPassword"
                    ></v-text-field>
                  </validation-provider>
                </v-row>
                <v-btn :loading="submitting" type="submit" :disabled="invalid"
                  >Login</v-btn
                >
              </form>
            </validation-observer>
          </v-row>
        </v-container>
      </v-card-text>
      <v-divider></v-divider>
      <v-card-text>
        <v-row align="center" justify="space-around">
          <v-btn :href="loginGithub" outlined
            ><v-icon left> mdi-github-circle </v-icon>Login with Github</v-btn
          >
          <v-btn :href="loginGoogle" outlined
            ><v-icon left> mdi-google </v-icon>Login with Google</v-btn
          >
          <v-btn :href="loginTwitter" outlined
            ><v-icon left> mdi-twitter </v-icon>Login with Twitter</v-btn
          >
        </v-row>
      </v-card-text>
    </v-card>
    <v-snackbar v-model="snackbar" color="red">
      {{ errorMessage }}
      <template v-slot:action="{ attrs }">
        <v-btn color="black" text v-bind="attrs" @click="snackbar = false">
          Close
        </v-btn>
      </template>
    </v-snackbar>
  </v-container>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { required, email } from "vee-validate/dist/rules";
import axios from "axios";
import {
  extend,
  ValidationObserver,
  ValidationProvider,
  setInteractionMode
} from "vee-validate";

setInteractionMode("eager");

extend("required", {
  ...required,
  message: "{_field_} can not be empty"
});

extend("email", {
  ...email,
  message: "Email must be valid"
});

@Component({
  components: {
    ValidationProvider,
    ValidationObserver
  }
})
export default class Home extends Vue {
  $refs!: {
    observer: HTMLFormElement;
  };
  baseURL = "https://spms-berv.herokuapp.com";
  loginGithub = `${this.baseURL}/oauth/github`;
  loginGoogle = `${this.baseURL}/oauth/google`;
  loginTwitter = `${this.baseURL}/oauth/twitter`;
  loginPassword = `${this.baseURL}/authentication`;
  showPassword = false;
  password = "";
  username = "";
  valid = true;
  submitting = false;
  snackbar = false;
  errorMessage = "";
  submit() {
    const result = this.$refs.observer.validate();
    if (!result) {
      console.log("Can't login");
      return;
    }
    this.submitting = true;
    axios
      .post(this.loginPassword, {
        strategy: "local",
        email: this.username,
        password: this.password
      })
      .then(result => {
        this.submitting = false;
        console.log(result.data);
      })
      .catch(error => {
        this.submitting = false;
        console.log(error);
        const statusCode = error.response.status;
        console.log(statusCode);
        if (statusCode === 401) {
          const dataResponse = error.response.data;
          const message = dataResponse.message;
          console.error(message);
          this.errorMessage = message;
        } else {
          this.errorMessage = "Failed to call server";
        }
        this.snackbar = true;
      });
  }
}
</script>
