<template>
    <page actionBarHidden="true" :class="{ 'dark': darkModeToggle }">
        <StackLayout>
            <FlexboxLayout class="actionBar">
                <FlexboxLayout @tap="goBack()" class="goBackIcon">
                    <Label col="0" text.decode="&#xf060;" class="fas"></Label>
                </FlexboxLayout>
                <Label>Account</Label>
            </FlexboxLayout>
            <FlexboxLayout class="inputOption option">
                <Label class="inputTitle">Nome utente</Label>
                <TextField v-model="name" class="input" returnKeyType="done"></TextField>
            </FlexboxLayout>

            <FlexboxLayout class="inputOption option">
                <Label class="inputTitle">E-mail</Label>
                <TextField v-model="email" class="input" returnKeyType="done"></TextField>
            </FlexboxLayout>

            <FlexboxLayout class="doneButton" @tap="pushSettings">
                <Label>Salva</Label>
            </FlexboxLayout>
            <FlexboxLayout class="resultText" :class="{ 'green': success }" v-if="success && !error">
                <Label col="0" text.decode="&#xf00c;" class="fas"></Label>
                <Label>Informazioni salvate con successo</Label>
            </FlexboxLayout>
            <FlexboxLayout class="resultText" :class="{ 'red': error }" v-if="!success && error">
                <Label col="0" text.decode="&#xf00d;" class="fas"></Label>
                <Label>c'è stato un errore...</Label>
            </FlexboxLayout>
        </StackLayout>
    </page>
</template>

<script lang="ts">
import Vue from 'vue';
import { Http, HttpResponse } from "@nativescript/core";
import * as AppSettings from "@nativescript/core/application-settings";
const { Application, Color } = require("@nativescript/core");
export default Vue.extend({
    data() {
        return {
            name: "",
            email: "",
            success: false,
            error: false,
            darkModeToggle: AppSettings.getBoolean("darkMode")
        }
    },
    mounted() {
        this.getSettings();
        setInterval(() => {
            this.darkModeToggle = AppSettings.getBoolean("darkMode");
        }, 200)
    },
    methods: {
        goBack() {
            this.$navigateBack();
        },
        async getSettings() {
            try {
                console.log("test");
                const response = await Http.request({
                    url: "http://192.168.1.16:8080/s",
                    method: "GET",
                    headers: {
                        "Content-Type": "application/json",
                        auth: AppSettings.getString('token')
                    }
                });

                if (response.statusCode === 200) {
                    const responseData = JSON.parse(response.content);
                    this.name = responseData.user.name;
                    this.email = responseData.user.email;
                } else {
                    console.error("Error fetching user information");
                }
            } catch (error) {
                console.error("Error:", error);
            }

        },
        async pushSettings() {
            try {
                console.log("test")
                const response = await Http.request({
                    url: "http://192.168.1.16:8080/s",
                    method: "POST",
                    headers: {
                        "Content-Type": "application/json",
                        auth: AppSettings.getString('token')
                    },
                    content: JSON.stringify({
                        name: this.name,
                        email: this.email,
                    }),
                });
                if (response.statusCode === 200) {
                    this.success = true;
                    setTimeout(() => {
                        this.success = false;
                    }, 3000)
                } else {
                    this.error = true;
                    setTimeout(() => {
                        this.error = false;
                        this.getSettings();
                    }, 3000)
                }
            } catch (error) {
                this.error = true;
                setTimeout(() => {
                    this.error = false;
                }, 3000)
            }
        },
    }
})
</script>

<style scoped>
@import url('../../../styles/subPages/settingsPage.scss');
</style>