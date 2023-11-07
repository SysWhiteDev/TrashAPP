<template>
  <StackLayout>
    <ActivityIndicator busy="true" v-if="loading" style="margin-top: 100px;" />
    <FlexboxLayout class="accountCard" v-if="!loading && !error">
      <Label class="accountName">{{ userdata.name }}</Label>
      <FlexBoxLayout class="accountStats">
        <StackLayout class="accountStat" style="background-color: rgba(161, 165, 42, 0.226)">
          <Label col="0" text.decode="&#xf1ea;" class="fas"></Label>
          <Label>{{ userdata.stats.paper.value }}{{ userdata.stats.unit }}</Label>
          <Label>di carta</Label>
        </StackLayout>
        <StackLayout class="accountStat" style="background-color: rgba(42, 147, 165, 0.226)">
          <Label col="0" text.decode="&#xe4c5;" class="fas"></Label>
          <Label>{{ userdata.stats.plastic.value }}{{ userdata.stats.unit }}</Label>
          <Label>di plastica</Label>
        </StackLayout>
        <StackLayout class="accountStat" style="background-color: rgba(165, 42, 42, 0.163)">
          <Label col="0" text.decode="&#xf468;" class="fas"></Label>
          <Label>{{ userdata.stats.cardboard.value }}{{ userdata.stats.unit }}</Label>
          <Label>di cartone</Label>
        </StackLayout>
      </FlexBoxLayout>
    </FlexboxLayout>
    <FlexboxLayout class="actionButton" @tap="addFriend" v-if="!loading && !error && localUserData.id != userid">
      <Label col="0" text.decode="&#xf234;" class="fas" style="margin-right: 15px"></Label>
      <Label>Aggiungi {{ userdata.name }} come amico</Label>
    </FlexboxLayout>
    <FlexboxLayout class="actionButton gray" @tap="goToSettings" v-else-if="!loading && !error && userdata.id == localUserData.id">
      <Label col="0" text.decode="&#xf013;" class="fas" style="margin-right: 15px"></Label>
      <Label>Impostazioni</Label>
    </FlexboxLayout>
  </StackLayout>
</template>

<script lang="ts">
import { Http, HttpResponse } from '@nativescript/core';
import Vue from 'vue';
import * as AppSettings from '@nativescript/core/application-settings';
import settingsSubPageVue from '../subpages/settingsSubPage.vue';
export default Vue.extend({
  data() {
    return {
      userdata: {},
      localUserData: '',
      loading: true,
      error: false
    };
  },
  props: {
    userid: Number
  },
  mounted() {
    this.localUserData = JSON.parse(AppSettings.getString('userdata'));
    setInterval(() => {
      this.getUserData()
    }, 2000);

  },
  methods: {
    async getUserData() {
      Http.request({
        url: `https://api.trashtracer.lol/u/${this.userid}`,
        method: 'GET',
        headers: {
          auth: AppSettings.getString('token')
        }
      }).then(
        (response: HttpResponse) => {
          if (response.statusCode == 200) {
            this.userdata = JSON.parse(response.content);
            this.loading = false;
            this.error = false;
          } else {
            this.error = true;
            this.loading = false;
          }
        },
        (e) => {
          console.log(e);
        }
      );
    },
    async addFriend() { },
    goToSettings() {
      this.$navigateTo(settingsSubPageVue, {
        transition: { name: 'slideTop' }
      })
    }
  }
});
</script>

<style scoped>
@import url('../../styles/pages/accountPage.scss');
</style>
