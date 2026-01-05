<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { useI18n } from 'vue-i18n'

const { t } = useI18n()

const stats = ref({
  stars: '--',
  commits: '--',
  prs: '--',
  issues: '--',
  contributors: '--'
})

const animateValue = (key: keyof typeof stats.value, start: number, end: number, duration: number) => {
  let startTimestamp: number | null = null
  const step = (timestamp: number) => {
    if (!startTimestamp) startTimestamp = timestamp
    const progress = Math.min((timestamp - startTimestamp) / duration, 1)
    const easeProgress = 1 - Math.pow(1 - progress, 4)
    const current = Math.floor(easeProgress * (end - start) + start)
    stats.value[key] = current.toString()
    if (progress < 1) {
      window.requestAnimationFrame(step)
    } else {
      stats.value[key] = end.toString()
    }
  }
  window.requestAnimationFrame(step)
}

const fetchGitHubStats = async () => {
  const repo = 'ShardLauncher/ShardLauncher'
  try {
    const repoRes = await fetch(`https://api.github.com/repos/${repo}`)
    if (!repoRes.ok) throw new Error('API Rate Limit')
    const repoData = await repoRes.json()

    animateValue('stars', 0, repoData.stargazers_count, 2000)
    animateValue('issues', 0, repoData.open_issues_count, 2000)

    const prRes = await fetch(`https://api.github.com/search/issues?q=repo:${repo}+type:pr`)
    const prData = await prRes.json()
    if (prData.total_count) {
      animateValue('prs', 0, prData.total_count, 2000)
    }

    const contribRes = await fetch(`https://api.github.com/repos/${repo}/contributors?per_page=1&anon=true`)
    const contribLink = contribRes.headers.get('link')
    if (contribLink) {
      const match = contribLink.match(/&page=(\d+)>; rel="last"/)
      if (match) {
        animateValue('contributors', 0, parseInt(match[1]), 2000)
      }
    } else {
      const contribData = await contribRes.json()
      animateValue('contributors', 0, contribData.length, 2000)
    }

    const commitRes = await fetch(`https://api.github.com/repos/${repo}/commits?per_page=1`)
    const commitLink = commitRes.headers.get('link')
    if (commitLink) {
      const match = commitLink.match(/&page=(\d+)>; rel="last"/)
      if (match) {
        animateValue('commits', 0, parseInt(match[1]), 2500)
      }
    }
  } catch (error) {
    console.warn('GitHub Stats Fetch Failed:', error)
  }
}

onMounted(() => {
  setTimeout(fetchGitHubStats, 500)
})
</script>

<template>
  <section class="stats-section">
    <div class="stats-container stagger-animate delay-3">
      <div class="stat-item">
        <i class="ri-star-smile-line stat-icon"></i>
        <div class="stat-number">{{ stats.stars }}</div>
        <div class="stat-label">{{ t('stat.stars') }}</div>
      </div>
      <div class="stat-item">
        <i class="ri-git-commit-line stat-icon"></i>
        <div class="stat-number">{{ stats.commits }}</div>
        <div class="stat-label">{{ t('stat.commits') }}</div>
      </div>
      <div class="stat-item">
        <i class="ri-git-pull-request-line stat-icon"></i>
        <div class="stat-number">{{ stats.prs }}</div>
        <div class="stat-label">{{ t('stat.prs') }}</div>
      </div>
      <div class="stat-item">
        <i class="ri-bug-line stat-icon"></i>
        <div class="stat-number">{{ stats.issues }}</div>
        <div class="stat-label">{{ t('stat.issues') }}</div>
      </div>
      <div class="stat-item">
        <i class="ri-user-heart-line stat-icon"></i>
        <div class="stat-number">{{ stats.contributors }}</div>
        <div class="stat-label">{{ t('stat.contributors') }}</div>
      </div>
    </div>
  </section>
</template>


