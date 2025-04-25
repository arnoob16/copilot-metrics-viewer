<template>
  <v-app>
    <v-toolbar color="primary" dark>
      <v-toolbar-title>My Application</v-toolbar-title>
      <v-spacer></v-spacer>

      <!-- Theme Toggle -->
      <v-btn icon @click="toggleTheme">
        <v-icon>{{ currentTheme === 'dark' ? 'mdi-weather-sunny' : 'mdi-moon-waning-crescent' }}</v-icon>
      </v-btn>
    </v-toolbar>

    <!-- Main Content -->
    <router-view />
  </v-app>
</template>

<script>
import { useTheme } from 'vuetify'

export default {
  name: 'App',
  setup() {
    const theme = useTheme()

    const toggleTheme = () => {
      theme.global.name.value = theme.global.name.value === 'dark' ? 'light' : 'dark'
    }

    return {
      currentTheme: theme.global.name,
      toggleTheme,
    }
  },
}
</script>

<style>
/* Add any global styles if needed */
</style>