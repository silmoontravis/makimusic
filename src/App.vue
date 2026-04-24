<template>
  <div class="app" :class="{ dark: isDark }">
    <header class="header">
      <div class="brand">
        <img :src="isDark ? '/logo_white_clean.png' : '/logo_blue_clean.png'" alt="HeyBoss" class="brand-logo" />
        <div>
          <div class="brand-name">HeyBoss Audio</div>
          <div class="brand-sub">全新生活專輯</div>
        </div>
      </div>
      <button class="theme-btn" @click="toggleTheme">
        {{ isDark ? '☀️' : '🌙' }}
      </button>
    </header>

    <div class="player-wrap">
      <div ref="playerEl" id="aplayer"></div>
    </div>

    <footer class="footer">
      HeyBoss Audio &copy; 2026 | Powered by KIE AI (Suno V5.5)
    </footer>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import APlayer from 'aplayer'
import 'aplayer/dist/APlayer.min.css'

const isDark = ref(false)
const playerEl = ref(null)
let ap = null

const audioList = [
  {
    name: '全新生活 馬上腰瘦',
    artist: 'HeyBoss Audio',
    url: '/audio/01-全新生活馬上腰瘦.mp3',
    cover: '/logo_blue_clean.png',
    lrc: '/audio/01-全新生活馬上腰瘦.lrc'
  },
  {
    name: '動起來 燃燒吧',
    artist: 'HeyBoss Audio',
    url: '/audio/02-動起來燃燒吧.mp3',
    cover: '/logo_blue_clean.png',
    lrc: '/audio/02-動起來燃燒吧.lrc'
  },
  {
    name: '跑出新自己',
    artist: 'HeyBoss Audio',
    url: '/audio/03-跑出新自己.mp3',
    cover: '/logo_blue_clean.png',
    lrc: '/audio/03-跑出新自己.lrc'
  },
  {
    name: '瑜伽時光',
    artist: 'HeyBoss Audio',
    url: '/audio/04-瑜伽時光.mp3',
    cover: '/logo_blue_clean.png',
    lrc: '/audio/04-瑜伽時光.lrc'
  }
]

function toggleTheme() {
  isDark.value = !isDark.value
  localStorage.setItem('theme', isDark.value ? 'dark' : 'light')
  updateThemeColor()
}

function updateThemeColor() {
  if (ap) {
    ap.theme(isDark.value ? '#EDC626' : '#005FDF', 0)
  }
}

onMounted(() => {
  const saved = localStorage.getItem('theme')
  if (saved === 'dark' || (!saved && window.matchMedia('(prefers-color-scheme: dark)').matches)) {
    isDark.value = true
  }

  ap = new APlayer({
    container: playerEl.value,
    audio: audioList,
    theme: isDark.value ? '#EDC626' : '#005FDF',
    lrcType: 3,
    listFolded: false,
    order: 'list',
    volume: 0.8,
    autoplay: false
  })

  if ('serviceWorker' in navigator) navigator.serviceWorker.register('/sw.js')
})
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;600;700;800&display=swap');
* { margin: 0; padding: 0; box-sizing: border-box; }
body { font-family: 'Noto Sans TC', -apple-system, sans-serif; min-height: 100vh; }

.app { min-height: 100vh; background: #f5f7fa; color: #111827; transition: all 0.3s; }
.app.dark { background: #0a0a0a; color: #f0f0f0; }

.header { display: flex; align-items: center; justify-content: space-between; padding: 16px 24px; background: #fff; border-bottom: 1px solid #e5e7eb; transition: all 0.3s; }
.dark .header { background: #1a1a1a; border-color: #2a2a2a; }
.brand { display: flex; align-items: center; gap: 12px; }
.brand-logo { height: 40px; }
.brand-name { font-size: 18px; font-weight: 800; color: #005FDF; }
.dark .brand-name { color: #EDC626; }
.brand-sub { font-size: 11px; color: #9ca3af; }
.theme-btn { background: none; border: 1px solid #e5e7eb; border-radius: 8px; padding: 8px 12px; font-size: 18px; cursor: pointer; transition: all 0.2s; }
.dark .theme-btn { border-color: #333; }

.player-wrap { max-width: 720px; margin: 24px auto; padding: 0 16px; }

/* APlayer overrides */
.aplayer { border-radius: 16px !important; box-shadow: 0 4px 20px rgba(0,0,0,0.08) !important; border: 1px solid #e5e7eb !important; }
.dark .aplayer { background: #1a1a1a !important; border-color: #2a2a2a !important; box-shadow: 0 4px 20px rgba(0,0,0,0.3) !important; color: #f0f0f0 !important; }
.dark .aplayer .aplayer-info { background: #1a1a1a !important; border-color: #2a2a2a !important; }
.dark .aplayer .aplayer-info .aplayer-music .aplayer-title { color: #f0f0f0 !important; }
.dark .aplayer .aplayer-info .aplayer-music .aplayer-author { color: #9ca3af !important; }
.dark .aplayer .aplayer-list ol li { border-color: #2a2a2a !important; color: #f0f0f0 !important; }
.dark .aplayer .aplayer-list ol li:hover { background: rgba(255,255,255,0.05) !important; }
.dark .aplayer .aplayer-lrc { background: #1a1a1a !important; }
.dark .aplayer .aplayer-lrc p { color: rgba(255,255,255,0.3) !important; }
.dark .aplayer .aplayer-lrc p.aplayer-lrc-current { color: #fff !important; }
.dark .aplayer .aplayer-controller .aplayer-time { color: #9ca3af !important; }
.dark .aplayer .aplayer-controller .aplayer-time .aplayer-icon path { fill: #9ca3af !important; }
.dark .aplayer .aplayer-list ol li .aplayer-list-author { color: #666 !important; }
/* Cover: circular + spin like turntable */
.aplayer .aplayer-pic { border-radius: 50% !important; transition: box-shadow 0.3s; }
.aplayer .aplayer-pic.aplayer-pause { animation: none !important; }
@keyframes coverSpin { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }
.aplayer .aplayer-pic { animation: coverSpin 8s linear infinite; animation-play-state: paused; }
.aplayer.aplayer-playing .aplayer-pic { animation-play-state: running; }
.dark .aplayer .aplayer-pic { background: #ffffff !important; box-shadow: 0 0 30px rgba(237,198,38,0.3); }
.dark .aplayer.aplayer-playing .aplayer-pic { box-shadow: 0 0 40px rgba(237,198,38,0.5), 0 0 80px rgba(237,198,38,0.15); }
.aplayer .aplayer-lrc p { font-size: 14px !important; }
.aplayer .aplayer-lrc p.aplayer-lrc-current { font-size: 16px !important; font-weight: 700 !important; }

.footer { text-align: center; padding: 20px; font-size: 11px; color: #9ca3af; }

@media (max-width: 600px) {
  .header { padding: 12px 16px; }
  .brand-logo { height: 32px; }
  .brand-name { font-size: 15px; }
  .player-wrap { margin: 12px auto; padding: 0 8px; }
}
</style>
