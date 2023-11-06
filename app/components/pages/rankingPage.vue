<template>
  <StackLayout class="content">
    <Label class="rankingTitle">Classifiche</Label>
    <Label class="rankingSection">Tipo di classifica</Label>
    <FlexboxLayout class="lbTypes">
      <FlexboxLayout class="lbType" :class="{ active: lbType === 'friends' }" @tap="changeLbType('friends')">
        <Label>Amici</Label>
      </FlexboxLayout>
      <FlexboxLayout class="lbType" :class="{ active: lbType === 'month' }" @tap="changeLbType('month')">
        <Label>Mensile</Label>
      </FlexboxLayout>
      <FlexboxLayout class="lbType" :class="{ active: lbType === 'allTime' }" @tap="changeLbType('allTime')">
        <Label>Tutti i tempi</Label>
      </FlexboxLayout>
    </FlexboxLayout>
    <Label class="rankingSection" style="margin-top: 20px"><Span>Classifica </Span> <Span
        style="text-decoration: underline">{{ lbTypeNames[lbType] }}</Span></Label>
    <StackLayout class="lbContainer">
      <FlexBoxLayout class="lbLegend">
        <FlexBoxLayout>
          <Label class="lbRank" style="font-weight: normal">Rank</Label>
          <Label class="lbName">Name</Label>
        </FlexBoxLayout>
        <FlexBoxLayout>
          <Label col="0" text.decode="&#xf1ea;" class="fas lbStat"
            style="background-color: rgba(161, 165, 42, 0.226)"></Label>
          <Label col="0" text.decode="&#xe4c5;" class="fas lbStat"
            style="background-color: rgba(42, 147, 165, 0.226)"></Label>
          <Label col="0" text.decode="&#xf468;" class="fas lbStat"
            style="background-color: rgba(165, 42, 42, 0.163); margin-right: 0;"></Label>
        </FlexBoxLayout>
      </FlexBoxLayout>
      <FlexBoxLayout class="lbPlayer" v-for="(player, index) in filteredLbData" :key="player.user.id"
        @tap="openUserPage(player.user.id)">
        <FlexBoxLayout>
          <Label class="lbRank">#{{ index + 1 }}</Label>
          <Label class="lbName">{{ player.user.name }}</Label>
        </FlexBoxLayout>
        <FlexBoxLayout>
          <Label class="lbStat">{{ player.stats.paper.value }}g</Label>
          <Label class="lbStat">{{ player.stats.plastic.value }}g</Label>
          <Label class="lbStat">{{ player.stats.cardboard.value }}g</Label>
        </FlexBoxLayout>
      </FlexBoxLayout>
    </StackLayout>
  </StackLayout>
</template>

<script lang="ts">
import Vue from 'vue';
import { Http, HttpContent, HttpResponse } from '@nativescript/core';
import * as AppSettings from "@nativescript/core/application-settings";
import externalAccountPageVue from './externalAccountPage.vue';
export default Vue.extend({
  data() {
    return {
      lbType: 'month',
      lbTypeNames: {
        friends: 'degli amici',
        month: 'mensile',
        allTime: 'di tutti i tempi'
      },
      lbData: {},
      filteredLbData: {},
      localUserData: '',
    };
  },
  mounted() {
    this.localUserData = JSON.parse(AppSettings.getString('userdata'));
    setInterval(() => {
      this.getLeaderboardData();
    }, 200)
  },
  methods: {
    changeLbType(to: string) {
      this.lbType = to;
    },
    async getLeaderboardData() {
      Http.request({
        url: "http://192.168.1.16:8080/lb",
        method: "GET",
        headers: {
          auth: AppSettings.getString("token"),
          type: this.lbType
        }
      }).then((res: HttpResponse) => {
        this.lbData = JSON.parse(res.content);
        if (this.lbType === 'friends') {
          this.filteredLbData = this.lbData.filter(player => player.friend === "TRUE" || player.user.id === this.localUserData.id);
        } else {
          this.filteredLbData = this.lbData;
        }
      })
    },
    openUserPage(id: number) {
      this.$navigateTo(externalAccountPageVue, {
        props: {
          userid: id
        }
      });
    }
  }
});
</script>

<style scoped>
@import url('../../styles/pages/rankingPage.scss');
</style>
