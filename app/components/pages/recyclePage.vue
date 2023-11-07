<template>
  <StackLayout>
    <AbsoluteLayout height="100%" width="100%">
      <StackLayout width="100%" height="100%" top="0">
        <ScrollView class="scrollView">
          <StackLayout class="content">
            <Label class="recycleTitle">Riciclo</Label>
            <Label class="recycleSection">Carta</Label>
            <FlexBoxLayout class="item paper" @tap="selectObject('Foglio', 5, 0)">
              <Label text.decode="&#xf1d8;" class="fas"></Label>
              <FlexboxLayout class="itemDescr">
                <Label>Foglio • 5g</Label>
              </FlexboxLayout>
            </FlexBoxLayout>
            <FlexBoxLayout class="item paper" @tap="selectObject('Volantino', 15, 0)">
              <Label text.decode="&#xf518;" class="fas"></Label>
              <FlexboxLayout class="itemDescr">
                <Label>Volantino • 15g</Label>
              </FlexboxLayout>
            </FlexBoxLayout>
            <Label class="recycleSection">Plastica</Label>
            <FlexBoxLayout class="item plastic" @tap="selectObject('Busta', 7, 1)">
              <Label text.decode="&#xf290;" class="fas"></Label>
              <FlexboxLayout class="itemDescr">
                <Label>Busta • 7g</Label>
              </FlexboxLayout>
            </FlexBoxLayout>
            <FlexBoxLayout class="item plastic" @tap="selectObject('Bottiglia', 17, 1)">
              <Label text.decode="&#xf72f;" class="fas"></Label>
              <FlexboxLayout class="itemDescr">
                <Label>Bottiglia • 17g</Label>
              </FlexboxLayout>
            </FlexBoxLayout>
            <FlexBoxLayout class="item plastic" @tap="selectObject('Posate usa e getta', 25, 1)">
              <Label text.decode="&#xf2e7;" class="fas"></Label>
              <FlexboxLayout class="itemDescr">
                <Label>Posate usa e getta • 25g</Label>
              </FlexboxLayout>
            </FlexBoxLayout>
            <Label class="recycleSection">Cartone</Label>
            <FlexBoxLayout class="item cardboard" @tap="selectObject('Scatola', 30, 2)">
              <Label text.decode="&#xf466;" class="fas"></Label>
              <FlexboxLayout class="itemDescr">
                <Label>Scatola • 30g</Label>
              </FlexboxLayout>
            </FlexBoxLayout>
            <FlexBoxLayout class="item cardboard" @tap="selectObject('Scatola della pizza', 65, 2)">
              <Label text.decode="&#xf818;" class="fas"></Label>
              <FlexboxLayout class="itemDescr">
                <Label>Scatola della pizza • 65g</Label>
              </FlexboxLayout>
            </FlexBoxLayout>
            <!-- <Label class="recycleSection" @tap="selectObject('Altro', 0, 3)">Altro</Label>
            <FlexBoxLayout class="item other">
              <Label text.decode="&#xf059;" class="fas"></Label>
              <FlexboxLayout class="itemDescr">
                <Label>Altro</Label>
              </FlexboxLayout>
            </FlexBoxLayout> -->
          </StackLayout>
        </ScrollView>
      </StackLayout>
      <FlexboxLayout class="popupWrapper" top="0" v-if="selectedObject.selected">
        <StackLayout class="popupContainer">
          <FlexboxLayout class="popupTopBar">
            <Label>{{ selectedObject.name }}</Label>
            <FlexboxLayout class="popupCloseButton" @tap="closePopup">
              <Label text.decode="&#xf00d;" class="fas"></Label>
            </FlexboxLayout>
          </FlexboxLayout>
          <FlexboxLayout class="popupAmmountButtons">
            <FlexboxLayout class="popupAmmountButton" style="background-color: #ff696973;" @tap="rmAmmount()">
              <Label>-</Label>
            </FlexboxLayout>
            <FlexboxLayout class="popupAmmountButton" style="background-color: #6565652c;">
              <Label>{{ selectedObject.ammount }}</Label>
            </FlexboxLayout>
            <FlexboxLayout class="popupAmmountButton" style="background-color: #50ae485d;" @tap="addAmmount()">
              <Label>+</Label>
            </FlexboxLayout>
          </FlexboxLayout>
          <FlexboxLayout class="ammountData">
            <Label>Peso totale: {{ selectedObject.value * selectedObject.ammount }}g</Label>
            <Label> • </Label>
            <Label>Materiale: {{ materials[selectedObject.type] }}</Label>
          </FlexboxLayout>
          <FlexboxLayout class="confirmButton" @tap="pushRecycle()">
            <Label v-if="!loading && !success">Conferma</Label>
            <ActivityIndicator busy="true" v-else-if="loading" style="color: black;" />
            <Label text.decode="&#xf00c;" class="fas" v-else-if="success" style="font-size: 22;"></Label>
          </FlexboxLayout>
        </StackLayout>
      </FlexboxLayout>
    </AbsoluteLayout>
  </StackLayout>
</template>

<script lang="ts">
import Vue from 'vue';
import { Http, HttpResponse } from '@nativescript/core';
import * as AppSettings from '@nativescript/core/application-settings';
export default Vue.extend({
  data() {
    return {
      loading: false,
      success: false,
      selectedObject: {
        selected: false,
        name: '',
        value: 0,
        ammount: 0,
        type: 0,
      },
      materials: {
        0: 'carta',
        1: 'plastica',
        2: 'cartone'
      }
    }
  },
  methods: {
    selectObject(name: string, value: number, type: number) {
      this.selectedObject = {
        selected: true,
        name: name,
        value: value,
        type: type,
        ammount: 1,
      }
      this.loading = false;
      this.success = false;
    },
    closePopup() {
      this.selectedObject.selected = false;
    },
    addAmmount() {
      if (this.loading) {
        return
      }
      this.selectedObject.ammount += 1;
    },
    rmAmmount() {
      if (this.loading) {
        return
      }
      if (this.selectedObject.ammount <= 1) {
        this.selectedObject.ammount = 1
      } else {
        this.selectedObject.ammount -= 1;
      }
    },
    async pushRecycle() {
      if (this.loading || this.selectedObject.ammount <= 1) {
        return
      }

      this.loading = true;
      Http.request({
        url: "https://api.trashtracer.lol/r",
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          auth: AppSettings.getString('token')
        },
        content: JSON.stringify({
          value: this.selectedObject.value * this.selectedObject.ammount,
          type: this.selectedObject.type,
        })
      }).then((response) => {
        console.log(response.statusCode == 200)
        if (response.statusCode == 200) {
          this.success = true;
          this.loading = false;
          setTimeout(() => {
            this.selectedObject.selected = false;
          }, 1500)
        }
      })
    }
  }
});
</script>

<style scoped>
@import url('../../styles/pages/recyclePage.scss');

.popupWrapper {
  flex-direction: column;
  justify-content: flex-end;
  align-items: flex-end;
  height: 100%;
  width: 100%;
}

.popupContainer {
  padding: 50px;
  background-color: var(--secondary);
  border-radius: 30px 30px 0px 0px;
  width: 100%;
}

/* topbar */
.popupTopBar {
  justify-content: space-between;
  font-size: 25px;
}

.popupCloseButton {
  background: rgba(255, 255, 255, 0.269);
  border-radius: 50%;
  height: 120px;
  width: 120px;
  justify-content: center;
  align-items: center;
}

/* ammount buttons */
.popupAmmountButtons {
  justify-content: space-between;
  align-items: center;
  width: 100%;
  margin-top: 40px;
}

.popupAmmountButtons>* {
  height: 250px;
  width: 250px;
  justify-content: center;
  align-items: center;
  border-radius: 30px;
  font-size: 40px;
}

/* ammount data */
.ammountData {
  margin-top: 40px;
  width: 100%;
  justify-content: center;
  font-size: 14px;
}

/* confirm button */
.confirmButton {
  justify-content: center;
  align-items: center;
  border-radius: 30px;
  background-color: var(--primary);
  margin-top: 30px;
  padding: 20px;
  font-size: 18px;
  margin-bottom: 0;
}
</style>
