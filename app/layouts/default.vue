<template>
  <v-app>
      <v-navigation-drawer
        v-model="drawer"
        permanent
        color="#10102e"
        @click="drawer = true"
      >
        <v-list>
          <v-list-item
            prepend-avatar="/hazen.png"
            subtitle="hazenaguelo02@gmail.com"
            title="Hazen Aguelo"
          ></v-list-item>
        </v-list>

        <v-divider></v-divider>

        <v-list density="compact" nav>
          <v-list-item prepend-icon="mdi mdi-view-dashboard" title="Dashboard" to="/"></v-list-item>
          <v-list-item prepend-icon="mdi mdi-qrcode-scan" title="QR Scanner" to="qr-scanner"></v-list-item>
          <v-list-item prepend-icon="mdi mdi-weather-partly-cloudy" title="Weather" to="weather"></v-list-item>
        </v-list>
      </v-navigation-drawer>

      <v-app-bar color="#10102e">
    <template v-slot:prepend>
    <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
    </template>

    <v-app-bar-title>PROFILE</v-app-bar-title>
  </v-app-bar>

         <slot />
      <v-main style="height: 250px"></v-main>
  </v-app>
</template>

<script>
export default {
  data: () => ({
    drawer: true,
  }),
}
</script>