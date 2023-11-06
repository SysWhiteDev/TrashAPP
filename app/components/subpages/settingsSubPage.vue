<template>
    <page actionBarHidden="true" :class="{'dark': darkModeToggle}">
        <StackLayout>
            <FlexboxLayout class="actionBar">
                <FlexboxLayout @tap="goBack()" class="goBackIcon">
                    <Label col="0" text.decode="&#xf062;" class="fas"></Label>
                </FlexboxLayout>
                <Label>Impostazioni</Label>
            </FlexboxLayout>
            <FlexboxLayout class="option" @tap="openAccountSettings">
                <Label>Account</Label>
                <Label col="0" text.decode="&#xf054;" class="fas optionIcon"></Label>
            </FlexboxLayout>
            <FlexboxLayout class="option" @tap="toggleDarkMode">
                <Label>Modalitá scura</Label>
                <Label v-if="darkModeToggle" class="optionIcon">ON</Label>
                <Label v-else class="optionIcon">OFF</Label>
            </FlexboxLayout>
            <FlexboxLayout class="option" @tap="logOut()" style="color: rgb(255, 100, 100);">
                <Label style="color: rgb(255, 100, 100);">Log out</Label>
                <Label col="0" text.decode="&#xf52b;" class="fas optionIcon" style="color: rgb(255, 100, 100);"></Label>
            </FlexboxLayout>
        </StackLayout>
    </page>
</template>

<script lang="ts">
import Vue from 'vue';
import * as AppSettings from '@nativescript/core/application-settings';
import accountSetting from './settings/accountSetting.vue';
import loaderPage from '../auth/loader.vue';
export default Vue.extend({
    data() {
        return {
            darkModeToggle: false,
        }
    },
    mounted() {
        this.darkModeToggle = AppSettings.getBoolean("darkMode");
    },
    methods: {
        goBack() {
            this.$navigateBack();
        },
        openAccountSettings() {
            this.$navigateTo(accountSetting, {
                transition: { name: 'slide' }
            })
        },
        toggleDarkMode() {
            AppSettings.setBoolean("darkMode", !this.darkModeToggle)
            this.darkModeToggle = !this.darkModeToggle
        },
        logOut() {
            console.log('logout');
            AppSettings.setString('token', '');
            this.$navigateTo(loaderPage, {
                clearHistory: true
            });
        }
    }
})
</script>

<style scoped>
@import url('../../styles/subPages/settingsPage.scss');
</style>