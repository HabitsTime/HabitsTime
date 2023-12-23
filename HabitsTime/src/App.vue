<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
  <header>
    <div class="wrapper">
      <!-- As a link -->
      <nav class="navbar bg-body-tertiary">
        <div class="container-fluid">
          <a class="navbar-brand" href="#">Habits</a>
          <div class="d-flex justify-content-end">
            <RouterLink class="RouterLink" to="/">Home</RouterLink>
            <RouterLink class="RouterLink" to="/about">About</RouterLink>
          </div>
        </div>
      </nav>
    </div>
  </header>

  <RouterView />
</template>

<style scoped>
.RouterLink {
  text-decoration: none;
  margin-left: 1rem;
}
</style>
