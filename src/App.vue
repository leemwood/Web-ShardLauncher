<script setup lang="ts">
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import Navbar from './components/Navbar.vue'
import AppFooter from './components/AppFooter.vue'
import AppAtmosphere from './components/AppAtmosphere.vue'

const route = useRoute()
const showFooter = computed(() => route.path !== '/generator')
</script>

<template>
  <div class="app-wrapper">
    <AppAtmosphere />
    <Navbar />
    <main>
      <router-view v-slot="{ Component }">
        <transition name="fade" mode="out-in">
          <component :is="Component" />
        </transition>
      </router-view>
    </main>
    <AppFooter v-if="showFooter" />
  </div>
</template>


