<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
  <nav>
    <RouterLink to="/">Home</RouterLink>
    <RouterLink to="/login">Log In</RouterLink>
  </nav>

  <RouterView />
</template>

<style scoped lang="scss"></style>
