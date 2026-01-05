<script setup lang="ts">
import { ref, onMounted, computed } from 'vue'
import { useI18n } from 'vue-i18n'

const { t, locale } = useI18n()

interface Asset {
  name: string
  browser_download_url: string
}

interface Release {
  published_at: string
  tag_name: string
  name: string
  body: string
  assets: Asset[]
}

interface Commit {
  sha: string
  commit: {
    author: {
      date: string
    }
    message: string
  }
}

const CONFIG = {
  repo: "ShardLauncher/ShardLauncher",
  workflow: "development.yml",
  artifacts: {
    "all": "com.lanrhyme.shardlauncher-all-debug",
    "arm64-v8a": "com.lanrhyme.shardlauncher-arm64-v8a-debug",
    "armeabi-v7a": "com.lanrhyme.shardlauncher-armeabi-v7a-debug",
    "x86_64": "com.lanrhyme.shardlauncher-x86_64-debug",
    "x86": "com.lanrhyme.shardlauncher-x86-debug"
  }
}

const stableRelease = ref<Release | null>(null)
const latestCommit = ref<Commit | null>(null)
const currentArch = ref('all')
const currentSource = ref('nightly')
const selectedAssetUrl = ref('')
const showModal = ref(false)
const detectedArch = ref('all')
const archOpen = ref(false)
const sourceOpen = ref(false)
const assetOpen = ref(false)
const isRateLimited = ref(false)

const timeAgo = computed(() => {
  if (!latestCommit.value) return '...'
  const date = new Date(latestCommit.value.commit.author.date)
  const now = new Date()
  const diffMs = now.getTime() - date.getTime()
  const diffHours = Math.floor(diffMs / (1000 * 60 * 60))
  const diffDays = Math.floor(diffHours / 24)

  if (locale.value === 'zh') {
    return diffDays > 0 ? `${diffDays} 天前` : `${diffHours} 小时前`
  }
  return diffDays > 0 ? `${diffDays} days ago` : `${diffHours} hours ago`
})

const releaseDate = computed(() => {
  if (!stableRelease.value) return '...'
  const date = new Date(stableRelease.value.published_at)
  return date.toLocaleDateString(locale.value === 'zh' ? 'zh-CN' : 'en-US')
})

const nightlyDownloadUrl = computed(() => {
  if (currentSource.value === 'actions') {
    return `https://github.com/${CONFIG.repo}/actions`
  }
  const artifactName = (CONFIG.artifacts as any)[currentArch.value] || CONFIG.artifacts['all']
  return `https://nightly.link/${CONFIG.repo}/workflows/${CONFIG.workflow}/master/${artifactName}.zip`
})

const fetchLatestRelease = async () => {
  const cacheKey = `gh_release_${CONFIG.repo}`
  const cached = localStorage.getItem(cacheKey)
  
  if (cached) {
    try {
      const { data, timestamp } = JSON.parse(cached)
      // Use cache if it's less than 1 hour old
      if (Date.now() - timestamp < 3600000) {
        stableRelease.value = data
        if (data.assets && data.assets.length > 0) {
          selectedAssetUrl.value = data.assets[0].browser_download_url
        }
      }
    } catch (e) {
      localStorage.removeItem(cacheKey)
    }
  }

  try {
    const res = await fetch(`https://api.github.com/repos/${CONFIG.repo}/releases/latest`)
    if (res.status === 403) {
      isRateLimited.value = true
      console.warn("GitHub API rate limit exceeded, using cached data if available.")
      return
    }
    if (!res.ok) throw new Error("API Error")
    const data = await res.json()
    stableRelease.value = data
    isRateLimited.value = false
    if (data.assets && data.assets.length > 0) {
      selectedAssetUrl.value = data.assets[0].browser_download_url
    }
    localStorage.setItem(cacheKey, JSON.stringify({ data, timestamp: Date.now() }))
  } catch (e) {
    console.error("Failed to fetch release info:", e)
  }
}

const fetchLatestCommit = async () => {
  const cacheKey = `gh_commit_${CONFIG.repo}`
  const cached = localStorage.getItem(cacheKey)
  
  if (cached) {
    try {
      const { data, timestamp } = JSON.parse(cached)
      if (Date.now() - timestamp < 3600000) {
        latestCommit.value = data
      }
    } catch (e) {
      localStorage.removeItem(cacheKey)
    }
  }

  try {
    const res = await fetch(`https://api.github.com/repos/${CONFIG.repo}/commits/master`)
    if (res.status === 403) {
      isRateLimited.value = true
      console.warn("GitHub API rate limit exceeded, using cached data if available.")
      return
    }
    if (!res.ok) throw new Error("API Error")
    const data = await res.json()
    latestCommit.value = data
    isRateLimited.value = false
    localStorage.setItem(cacheKey, JSON.stringify({ data, timestamp: Date.now() }))
  } catch (e) {
    console.error("Failed to fetch commit info:", e)
  }
}

const detectArchitecture = () => {
  const ua = navigator.userAgent.toLowerCase()
  let detected = 'all'
  if (ua.includes('aarch64') || ua.includes('arm64') || ua.includes('armv8')) {
    detected = 'arm64-v8a'
  } else if (ua.includes('armv7') || ua.includes('armeabi')) {
    detected = 'armeabi-v7a'
  } else if (ua.includes('x86_64') || ua.includes('amd64')) {
    detected = 'x86_64'
  } else if (ua.includes('x86') || ua.includes('i686')) {
    detected = 'x86'
  }
  detectedArch.value = detected
  currentArch.value = detected
}

const selectAsset = (url: string) => {
  selectedAssetUrl.value = url
  assetOpen.value = false
}

const selectArch = (arch: string) => {
  currentArch.value = arch
  archOpen.value = false
}

const selectSource = (source: string) => {
  currentSource.value = source
  sourceOpen.value = false
}

onMounted(() => {
  fetchLatestRelease()
  fetchLatestCommit()
  detectArchitecture()
})
</script>

<template>
  <div class="download-view">
    <nav class="sub-nav">
      <router-link to="/" class="back-link">
        <i class="ri-arrow-left-line"></i> <span>{{ t('nav.back') }}</span>
      </router-link>
    </nav>

    <div class="download-wrapper">
      <!-- Stable Release Card -->
      <div class="download-card release-card">
        <div v-if="isRateLimited && !stableRelease" class="api-warning">
          <i class="ri-error-warning-line"></i>
          <span>{{ t('msg.rate_limit') }}</span>
        </div>
        <div class="card-header">
          <span class="card-badge">{{ t('dl.stable') }}</span>
          <h1 class="card-title">{{ t('dl.stable_title') }}</h1>
          <p class="card-desc">{{ t('dl.stable_desc') }}</p>
        </div>

        <div class="build-info">
          <div class="info-row">
            <span>{{ t('info.updated') }}</span>
            <span>{{ releaseDate }}</span>
          </div>
          <div class="info-row">
            <span>{{ t('info.version') }}</span>
            <span>{{ stableRelease?.tag_name || '...' }}</span>
          </div>
          <div class="commit-msg">{{ stableRelease?.name || t('msg.fetching') }}</div>
        </div>

        <button class="secondary-btn" @click="showModal = true">
          <i class="ri-file-list-3-line"></i> <span>{{ t('btn.changelog') }}</span>
        </button>

        <div class="control-group">
          <label class="label">{{ t('lbl.release_assets') }}</label>
          <div class="c-select" :class="{ 'open': assetOpen }">
            <div class="c-select-trigger" @click="assetOpen = !assetOpen">
              <span class="selected-text">{{ stableRelease?.assets?.find(a => a.browser_download_url === selectedAssetUrl)?.name || t('opt.loading') }}</span>
              <i class="ri-arrow-down-s-line"></i>
            </div>
            <div class="c-options">
              <div v-for="asset in stableRelease?.assets" :key="asset.name" 
                   class="c-option" :class="{ 'selected': selectedAssetUrl === asset.browser_download_url }"
                   @click="selectAsset(asset.browser_download_url)">
                <i class="ri-file-zip-line"></i><span>{{ asset.name }}</span>
              </div>
            </div>
          </div>
        </div>

        <a :href="selectedAssetUrl" class="dl-btn">
          <i class="ri-rocket-2-fill"></i>
          <span>{{ t('btn.download_stable') }}</span>
        </a>

        <div class="github-link">
          <a :href="`https://github.com/${CONFIG.repo}/releases`" target="_blank">
            <span>{{ t('msg.all_releases') }}</span> <i class="ri-external-link-line"></i>
          </a>
        </div>
      </div>

      <!-- Nightly Build Card -->
      <div class="download-card">
        <div class="card-header">
          <span class="card-badge snapshot">{{ t('dl.snapshot') }}</span>
          <h1 class="card-title">{{ t('dl.title') }}</h1>
          <p class="card-desc">{{ t('dl.desc') }}</p>
        </div>

        <div class="build-info">
          <div class="info-row">
            <span>{{ t('info.updated') }}</span>
            <span>{{ timeAgo }}</span>
          </div>
          <div class="info-row">
            <span>{{ t('info.version') }}</span>
            <span>{{ latestCommit?.sha.substring(0, 7) || '...' }}</span>
          </div>
          <div class="commit-msg">{{ latestCommit?.commit.message || t('msg.fetching') }}</div>
        </div>

        <div class="control-group">
          <label class="label">{{ t('lbl.source') }}</label>
          <div class="c-select" :class="{ 'open': sourceOpen }">
            <div class="c-select-trigger" @click="sourceOpen = !sourceOpen">
              <span class="selected-text">{{ t(`opt.src.${currentSource}`) }}</span>
              <i class="ri-arrow-down-s-line"></i>
            </div>
            <div class="c-options">
              <div class="c-option" :class="{ 'selected': currentSource === 'nightly' }" @click="selectSource('nightly')">
                {{ t('opt.src.nightly') }}
              </div>
              <div class="c-option" :class="{ 'selected': currentSource === 'actions' }" @click="selectSource('actions')">
                {{ t('opt.src.actions') }}
              </div>
            </div>
          </div>
        </div>

        <div class="control-group">
          <label class="label">{{ t('lbl.arch') }}</label>
          <div class="c-select" :class="{ 'open': archOpen, 'disabled': currentSource === 'actions' }">
            <div class="c-select-trigger" @click="archOpen = !archOpen">
              <span class="selected-text">{{ t(`opt.arch.${currentArch === 'arm64-v8a' ? 'arm64' : currentArch === 'armeabi-v7a' ? 'armv7' : currentArch === 'x86_64' ? 'x64' : currentArch}`) }}</span>
              <i class="ri-arrow-down-s-line"></i>
            </div>
            <div class="c-options">
              <div class="c-option" :class="{ 'selected': currentArch === 'all' }" @click="selectArch('all')">{{ t('opt.arch.all') }}</div>
              <div class="c-option" :class="{ 'selected': currentArch === 'arm64-v8a' }" @click="selectArch('arm64-v8a')">{{ t('opt.arch.arm64') }}</div>
              <div class="c-option" :class="{ 'selected': currentArch === 'armeabi-v7a' }" @click="selectArch('armeabi-v7a')">{{ t('opt.arch.armv7') }}</div>
              <div class="c-option" :class="{ 'selected': currentArch === 'x86_64' }" @click="selectArch('x86_64')">{{ t('opt.arch.x64') }}</div>
              <div class="c-option" :class="{ 'selected': currentArch === 'x86' }" @click="selectArch('x86')">{{ t('opt.arch.x86') }}</div>
            </div>
          </div>
          <div class="detected-badge" :class="{ 'visible': detectedArch !== 'all' && currentSource !== 'actions' }">
            <i class="ri-magic-line"></i> <span>{{ t('msg.detected') }}</span>&nbsp;<span>{{ detectedArch.toUpperCase() }}</span>
          </div>
        </div>

        <a :href="nightlyDownloadUrl" class="dl-btn" :target="currentSource === 'actions' ? '_blank' : '_self'">
          <i :class="currentSource === 'actions' ? 'ri-external-link-line' : 'ri-download-cloud-2-fill'"></i>
          <span>{{ currentSource === 'actions' ? t('btn.go_actions') : t('btn.download') }}</span>
        </a>

        <div class="disclaimer">
          {{ t('msg.disclaimer') }}
        </div>
      </div>
    </div>

    <!-- Modal -->
    <div class="modal-overlay" :class="{ 'open': showModal }" @click.self="showModal = false">
      <div class="modal-content">
        <div class="modal-header">
          <span class="modal-title">{{ t('modal.title') }}</span>
          <button class="modal-close" @click="showModal = false">&times;</button>
        </div>
        <div class="modal-body">
          {{ stableRelease?.body || 'No release notes provided.' }}
        </div>
      </div>
    </div>
  </div>
</template>


