<script setup lang="ts">
import { ref } from 'vue'
import { useI18n } from 'vue-i18n'
import ProjectStats from '../components/ProjectStats.vue'
import { useTilt } from '../composables/useTilt'

const { t } = useI18n()
const tiltCard = ref(null)
useTilt(tiltCard)
</script>

<template>
  <div class="home-view">
    <header class="hero">
      <div class="hero-visual">
        <div class="visual-glass-card" ref="tiltCard">
          <div class="ring"></div>
          <div class="breathing-glow"></div>
          <img src="/logo.png" alt="Shard" class="main-logo">
        </div>
      </div>
      <div class="hero-content">
        <div class="badge stagger-animate">{{ t('hero.badge') }}</div>
        <i18n-t keypath="hero.title" tag="h1" class="stagger-animate delay-1">
          <template #br>
            <br>
          </template>
          <template #span>
            <span class="highlight">{{ t('hero.title_span') }}</span>
          </template>
        </i18n-t>
        <p class="stagger-animate delay-2">{{ t('hero.desc') }}</p>
        <div class="btn-group stagger-animate delay-3">
          <router-link to="/download" class="btn btn-pri">
            <i class="ri-download-cloud-2-fill"></i> <span>{{ t('btn.download') }}</span>
          </router-link>
          <a href="https://github.com/ShardLauncher/ShardLauncher" class="btn btn-sec">
            <i class="ri-github-fill"></i> {{ t('btn.source') }}
          </a>
        </div>
      </div>
    </header>

    <ProjectStats />

    <section class="features">
      <div class="grid-container">
        <div class="feature-card span-2">
          <div class="f-icon"><i class="ri-layout-masonry-fill"></i></div>
          <h3>{{ t('ft.1.t') }}</h3>
          <p>{{ t('ft.1.d') }}</p>
        </div>
        <div class="feature-card">
          <div class="f-icon"><i class="ri-cpu-line"></i></div>
          <h3>{{ t('ft.2.t') }}</h3>
          <p>{{ t('ft.2.d') }}</p>
        </div>
        <div class="feature-card">
          <div class="f-icon"><i class="ri-equalizer-fill"></i></div>
          <h3>{{ t('ft.3.t') }}</h3>
          <p>{{ t('ft.3.d') }}</p>
        </div>
        <div class="feature-card span-2">
          <div class="f-icon"><i class="ri-settings-4-fill"></i></div>
          <h3>{{ t('ft.4.t') }}</h3>
          <p>{{ t('ft.4.d') }}</p>
        </div>
      </div>
    </section>
  </div>
</template>


