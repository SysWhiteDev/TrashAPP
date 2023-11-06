<template>
  <page actionBarHidden="true">
    <FlexboxLayout class="content">
      <Label />
      <StackLayout class="form">
        <Label class="formTitle">Log in</Label>
        <label class="formSubTitle">{{ logText }}</label>
        <TextField
          hint="Username"
          v-model="username"
          secure="false"
          autocorrect="false"
          maxLength="16"
        ></TextField>
        <TextField
          hint="Password"
          v-model="passwd"
          secure="true"
          returnKeyType="done"
          autocorrect="false"
        ></TextField>
        <FlexBoxLayout class="buttons">
          <Button text="Log in" class="button" @tap="tryLogIn"></Button>
          <Label style="font-size: 15">o</Label>
          <Button text="Registrati" class="minimalButton" @tap="goToRegister"></Button>
        </FlexBoxLayout>
      </StackLayout>
    </FlexboxLayout>
  </page>
</template>

<script lang="ts">
import { Http, HttpResponse } from "@nativescript/core";
import * as AppSettings from "@nativescript/core/application-settings";
import Vue from "vue";

/* import page components */
import loaderPage from "./loader.vue";
import registerPage from "./register.vue"

interface ResponseContent {
  token?: string; // Assuming token is of string type; adjust accordingly
}

export default Vue.extend({
  data() {
    return {
      username: "",
      passwd: "",
      logText: "Please login into your TrashTracer account",
    };
  },
  methods: {
    async tryLogIn() {
      try {
        const response = await Http.request({
          url: "http://192.168.1.16:8080/auth/l",
          method: "POST",
          headers: { "Content-Type": "application/json" },
          content: JSON.stringify({
            name: this.username,
            passwd: this.passwd,
          }),
        });
        if (response.statusCode === 200) {
          const content: ResponseContent = JSON.parse(response.content);
          if (content.token) {
            this.logText = "Authenticated with success";
            AppSettings.setString("token", content.token);
            this.$navigateTo(loaderPage, {
              clearHistory: true,
            });
          } else {
            this.logText = "Authentication failed";
          }
        } else {
          this.logText = "Authentication failed";
        }
      } catch (error) {}
    },
    goToRegister() {
      this.$navigateTo(registerPage)
    }
  },
});
</script>
<style scoped>
@import url("../../styles/auth/login.scss");
</style>
