<template>
 <v-container>
 <v-card v-if="user" class="pa-6">
 <div class="d-flex align-center ga-4">
 <v-avatar size="64">
 <v-img :src="user.picture" />
 </v-avatar>
 <div>
 <h2>{{ user.name }}</h2>
 <p>{{ user.email }}</p>
 </div>
 </div>
 </v-card>

<v-btn color="error" @click="logout">
 Logout
</v-btn>


 </v-container>

</template>

<script lang="ts" setup>

const user = ref<any>(null)
onMounted(() => {
 const savedUser = localStorage.getItem('google_user')
 if (savedUser) {
 user.value = JSON.parse(savedUser)
 }
})

const logout = () => {
 localStorage.removeItem('google_user')
 localStorage.removeItem('google_token')
 navigateTo('/login')
}



</script>

<style>

</style>