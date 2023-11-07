<template>
  <Page actionBarHidden="true">
    <FlexBoxLayout class="content">
      <Label>&nbsp;</Label>
      <StackLayout>
        <Image src="~/assets/recycleIconBig.png" class="logo"></Image>
        <Label class="logoText">TrashTracer</Label>
      </StackLayout>
      <Label class="logText">{{ logText }}</Label>
    </FlexBoxLayout>
  </Page>
</template>

<script lang="ts">
import { Http, HttpResponse } from '@nativescript/core';
import Vue from 'nativescript-vue';
import * as AppSettings from '@nativescript/core/application-settings';

/* import page components */
import HomePage from '../Home.vue';
import LoginPage from './login.vue';

export default Vue.extend({
  data() {
    return {
      logText: "Connecting to TrashTracer's servers..."
    };
  },
  mounted() {
    this.checkSessionValid();
  },
  methods: {
    async checkSessionValid() {
      const token = AppSettings.getString('token');
      Http.request({
        url: 'https://api.trashtracer.lol/u/',
        method: 'GET',
        headers: {
          auth: token
        }
      }).then(
        (response: HttpResponse) => {
          if (response.statusCode === 200) {
            const responseData: any = JSON.stringify(response.content);
            AppSettings.setString('userdata', responseData);
            this.logText = 'Authenticated with success';
            this.$navigateTo(HomePage, {
              clearHistory: true
            });
          } else {
            this.logText = 'Authentication failed';
            this.$navigateTo(LoginPage, {
              clearHistory: true
            });
          }
        },
        (e) => {
          console.log(e);
        }
      );
    }
  }
});
</script>

<style scoped>
@import url('../../styles/loader.scss');
</style>
