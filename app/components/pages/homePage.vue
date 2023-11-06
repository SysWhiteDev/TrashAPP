<template>
  <StackLayout>
    <Label class="homeTitle">Benvenuto, {{ userdata.name }}</Label>
    <StackLayout class="homeTotalAmount">
      <Label class="homeTotalAmountTitle">Questa settimana hai riciclato</Label>
      <Label class="homeTotalAmountValue">{{ calculatedTotalAmount }} g</Label>
      <Label class="homeTotalAmountLabel">di rifiuti totali</Label>
    </StackLayout>
    <StackLayout class="homeChart">
      <Label class="chartTitle">Grafico di questa settimana</Label>
      <RadCartesianChart>
        <CategoricalAxis v-tkCartesianHorizontalAxis />
        <LinearAxis v-tkCartesianVerticalAxis label="test" />
        <LineSeries v-tkCartesianSeries :items="chartDataMASS" categoryProperty="Day" valueProperty="Amount" />
      </RadCartesianChart>
    </StackLayout>
  </StackLayout>
</template>

<script lang="ts">
import Vue from 'vue';
import * as AppSettings from '@nativescript/core/application-settings';
import { Http, HttpResponse } from '@nativescript/core';

import loaderPage from '../auth/loader.vue';
export default Vue.extend({
  data() {
    return {
      userdata: '',
      chartDataMASS: [],
      calculatedTotalAmount: 0
    };
  },
  mounted() {
    this.userdata = JSON.parse(AppSettings.getString('userdata'));
    setInterval(() => {
      this.getGraphData();
    }, 200)
  },
  methods: {
    logOut() {
      console.log('logout');
      AppSettings.setString('token', '');
      this.$navigateTo(loaderPage, {
        clearHistory: true
      });
    },
    async getGraphData() {
      Http.request({
        "url": "http://192.168.1.16:8080/g",
        method: "GET",
        headers: {
          auth: AppSettings.getString("token")
        }
      }).then((res: HttpResponse) => {
        this.chartDataMASS = [];
        this.chartDataMASS = JSON.parse(res.content);
        this.calculatedTotalAmount = 0;
        for (const entry of this.chartDataMASS) {
          this.calculatedTotalAmount += entry.Amount;
        }
      })
    }
  }
});
</script>

<style scoped>
@import url('../../styles/pages/homePage.scss');
</style>
