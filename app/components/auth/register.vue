<template>
  <page actionBarHidden="true">
    <FlexboxLayout class="content">
      <Label />
      <StackLayout class="form">
        <Label class="formTitle">Registrati</Label>
        <label class="formSubTitle">{{ logText }}</label>
        <TextField hint="Username" v-model="username" secure="false" autocorrect="false" maxLength="16"></TextField>
        <TextField hint="E-mail" v-model="email" secure="false" autocorrect="false"></TextField>
        <TextField hint="Password" v-model="passwd" secure="true" returnKeyType="done" autocorrect="false"></TextField>
        <FlexBoxLayout class="buttons">
          <Button text="Registrati" class="button" @tap="tryRegister"></Button>
          <Label style="font-size: 15">o</Label>
          <Button text="Log in" class="minimalButton" @tap="goToLogin"></Button>
        </FlexBoxLayout>
      </StackLayout>
    </FlexboxLayout>
  </page>
</template>

<script lang="ts">
import { Http, HttpResponse } from '@nativescript/core';
import * as AppSettings from '@nativescript/core/application-settings';
import Vue from 'vue';

/* import page components */
import loaderPage from './loader.vue';
import loginPage from './login.vue';

interface ResponseContent {
  token?: string; // Assuming token is of string type; adjust accordingly
}

export default Vue.extend({
  data() {
    return {
      username: '',
      passwd: '',
      email: '',
      logText: 'Benvenuto su TrashTracer'
    };
  },
  methods: {
    async tryRegister() {
      try {
        const response = await Http.request({
          url: 'http://192.168.1.16:8080/auth/r',
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          content: JSON.stringify({
            name: this.username,
            passwd: this.passwd,
            email: this.email
          })
        });
        if (response.statusCode === 200) {
          const content: ResponseContent = JSON.parse(response.content);
          this.logText = 'Ti sei registrato con successo';
          this.$navigateTo(loginPage);
        } else {
          if (JSON.parse(response.content).message == 'NAME_EXISTS') {
            this.logText = "Un altro utente ha gia' questo nome";
          } else {
            this.logText = 'Ho riscontrato un problema durante la registrazione...';
          }
        }
      } catch (error) {}
    },
    goToLogin() {
      this.$navigateTo(loginPage);
    }
  }
});
</script>
<style scoped>
@import url('../../styles/auth/login.scss');
</style>
