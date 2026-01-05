<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import { useRouter } from 'vue-router'
import { useI18n } from 'vue-i18n'

const { locale } = useI18n()
const scrolled = ref(false)
const theme = ref(localStorage.getItem('theme') || 'dark')
const lang = ref(locale.value)
const langOpen = ref(false)

const router = useRouter()

const handleScroll = () => {
  scrolled.value = window.scrollY > 50
}

const toggleTheme = () => {
  theme.value = theme.value === 'dark' ? 'light' : 'dark'
  document.documentElement.setAttribute('data-theme', theme.value)
  localStorage.setItem('theme', theme.value)
}

const setLang = (l: string) => {
  lang.value = l
  locale.value = l
  localStorage.setItem('lang', l)
  langOpen.value = false
}

onMounted(() => {
  window.addEventListener('scroll', handleScroll)
  document.documentElement.setAttribute('data-theme', theme.value)
})

onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll)
})
</script>

<template>
  <nav class="main-nav" :class="{ 'scrolled': scrolled }">
    <router-link to="/" class="brand">
      <img src="/logo.png" alt="ShardLauncher">
      <span>ShardLauncher</span>
    </router-link>

    <div class="nav-tools">
      <div class="custom-select-wrapper" :class="{ 'open': langOpen }">
        <div class="custom-select-trigger" @click="langOpen = !langOpen">
          <span class="custom-select-label">{{ lang === 'zh' ? '简体中文' : 'English' }}</span>
        </div>
        <div class="custom-options">
          <div class="custom-option" :class="{ 'selected': lang === 'zh' }" @click="setLang('zh')">简体中文</div>
          <div class="custom-option" :class="{ 'selected': lang === 'en' }" @click="setLang('en')">English</div>
        </div>
      </div>

      <button class="tool-btn" @click="toggleTheme">
        <i :class="theme === 'dark' ? 'ri-sun-line' : 'ri-moon-line'"></i>
      </button>

      <router-link to="/download" class="tool-btn">
        <i class="ri-download-line"></i>
      </router-link>
    </div>
  </nav>
</template>


