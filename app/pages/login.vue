<template>
  <div>
    <v-container class="fill-height d-flex align-center justify-center">
 <v-card width="400" class="pa-6 rounded-xl">
 <v-card-title class="text-center text-h5">
 Login
 </v-card-title>
 <v-card-text>
 <v-btn
 block
 color="primary"
 size="large"
 prepend-icon="mdi-google"
 @click="loginWithGoogle"

 
 >
 Sign in with Google
 </v-btn>
 </v-card-text>
 </v-card>
 </v-container>
  </div>
</template>

<script lang="ts" setup>

// @ts-nocheck
const config = useRuntimeConfig()
declare global {
 interface Window {
 google: any
 }
}
const loginWithGoogle = () => {
 const client = window.google.accounts.oauth2.initTokenClient({
 client_id: config.public.googleClientId,
 scope: 'openid email profile',
 callback: async (response: any) => {
 const userInfo = await $fetch(
 'https://www.googleapis.com/oauth2/v3/userinfo',
 {
 headers: {
 Authorization: `Bearer ${response.access_token}`
 }
 }
 )
 localStorage.setItem(
 'google_user',
 JSON.stringify(userInfo)
 )
 localStorage.setItem(
 'google_token',
 response.access_token
 )
 navigateTo('/')
 }
 })
 client.requestAccessToken()
}



</script>

<style>

</style>