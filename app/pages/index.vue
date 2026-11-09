<template>
     <v-container class="fill-height d-flex align-center justify-center">
     <v-card v-if="user" width="400" class="pa-6 rounded-xl">
      <div class="d-flex align-center ga-4 mb-4">
        <v-avatar size="64">
          <v-img :src="user.picture"/>
        </v-avatar>
        <div>
          <h2>{{ user.name }}</h2>
          <p class="text-caption text-grey">{{ user.email }}</p>
        </div>
      </div>

      <v-btn color="error" @click="logout">
      Logout
      </v-btn>

    </v-card>
  </v-container>
</template>

<script setup lang="ts">
//@ts-nocheck
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