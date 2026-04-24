<template>
  <div class="app" :data-theme="isDark ? 'dark' : ''">

    <!-- ===== MOBILE: Player View ===== -->
    <div class="view player-view" :class="{ active: mobileView === 'player' }">
      <div class="pv-bg"></div>
      <div class="pv-top">
        <button @click="mobileView = 'library'"><i class="icon-chevron-down"></i></button>
        <div style="display:flex;gap:4px">
          <button @click="toggleTheme" title="明/暗切換"><i :class="isDark ? 'icon-moon' : 'icon-sun'"></i></button>
          <button @click="mobileView = 'library'"><i class="icon-list-music"></i></button>
        </div>
      </div>
      <div class="pv-content">
        <div class="pv-cover" :class="{ spin: isPlaying }">
          <img v-if="isDark" src="/logo_blue_clean.png" alt="HeyBoss" />
          <img v-else src="/logo_blue_clean.png" alt="HeyBoss" />
        </div>
        <div class="pv-title">{{ currentTrack?.title || 'HeyBoss Audio' }}</div>
        <div class="pv-artist">{{ currentTrack ? currentTrack.artist : '選擇歌曲開始播放' }}</div>
        <div class="pv-actions">
          <button class="pv-act" @click="mobileView = 'library'"><i class="icon-plus"></i><span>加入</span></button>
          <button class="pv-act" :class="{ active: isFav }" @click="isFav = !isFav"><i class="icon-heart"></i><span>收藏</span></button>
          <button class="pv-act"><i class="icon-download"></i><span>下載</span></button>
          <button class="pv-act" @click="mobileView = 'lyrics'"><i class="icon-text"></i><span>歌詞</span></button>
        </div>
        <div class="pv-progress">
          <div class="pv-prog-bar" @click="seek($event)">
            <div class="pv-prog-fill" :style="{ width: progress + '%' }"></div>
          </div>
          <div class="pv-times"><span>{{ fmt(currentTime) }}</span><span>{{ fmt(duration) }}</span></div>
        </div>
        <div class="pv-controls">
          <button class="pv-ctrl" :class="{ active: playMode > 0 }" @click="cycleMode"><i :class="playMode === 2 ? 'icon-repeat-1' : 'icon-repeat'"></i></button>
          <button class="pv-ctrl" @click="prevTrack"><i class="icon-skip-back"></i></button>
          <button class="pv-ctrl main-play" @click="togglePlay"><i :class="isPlaying ? 'icon-pause' : 'icon-play'"></i></button>
          <button class="pv-ctrl" @click="nextTrack"><i class="icon-skip-forward"></i></button>
          <button class="pv-ctrl" :class="{ active: isShuffle }" @click="isShuffle = !isShuffle"><i class="icon-shuffle"></i></button>
        </div>
        <div class="pv-vol">
          <button @click="toggleMute"><i :class="volIcon"></i></button>
          <input type="range" min="0" max="1" step="0.01" :value="volume" @input="setVol($event.target.value)" />
        </div>
      </div>
    </div>

    <!-- ===== MOBILE: Lyrics View ===== -->
    <div class="view lyrics-fullview" :class="{ active: mobileView === 'lyrics' }">
      <div class="lv-bg"></div>
      <div class="lv-header">
        <button @click="mobileView = 'player'"><i class="icon-chevron-down"></i></button>
        <div><div class="song">{{ currentTrack?.title || '歌詞' }}</div><div class="artist">{{ currentTrack?.artist || '' }}</div></div>
        <button @click="mobileView = 'library'"><i class="icon-list-music"></i></button>
      </div>
      <div class="lv-scroll" ref="lyricsScrollM">
        <div style="height:40vh"></div>
        <div v-for="(line, i) in currentLyrics" :key="i" class="lv-line"
          :class="{ active: i === activeLyricIdx, passed: i < activeLyricIdx }"
          @click="seekToLyric(i)" ref="lyricLinesM">
          {{ line.text }}
        </div>
        <div style="height:40vh"></div>
      </div>
    </div>

    <!-- ===== MOBILE: Library View ===== -->
    <div class="view library-view" :class="{ active: mobileView === 'library' }">
      <div class="lib-header">
        <h2>音樂庫</h2>
        <div class="lib-tabs">
          <button class="lib-tab active">全部</button>
          <button class="lib-tab">專輯</button>
          <button class="lib-tab">播放清單</button>
          <button class="lib-tab">下載</button>
        </div>
      </div>
      <div class="lib-list">
        <div v-for="(track, i) in tracks" :key="i" class="lib-item" :class="{ active: i === curIdx }" @click="playTrack(i)">
          <div class="lib-cover">
            <img src="/logo_blue_clean.png" alt="cover" />
          </div>
          <div class="lib-info">
            <div class="lib-name">{{ track.title }}</div>
            <div class="lib-meta">{{ track.artist }} · {{ track.duration }}</div>
          </div>
          <button class="lib-more"><i class="icon-more-horizontal"></i></button>
        </div>
      </div>
    </div>

    <!-- ===== MOBILE: Bottom Mini Player ===== -->
    <div class="bottom-mini" v-if="mobileView === 'library' && curIdx >= 0" @click="mobileView = 'player'">
      <div class="bm-cover"><img src="/logo_blue_clean.png" alt="" /></div>
      <div class="bm-info">
        <div class="t">{{ currentTrack?.title }}</div>
        <div class="s">{{ currentTrack?.artist }}</div>
      </div>
      <button class="bm-play" @click.stop="togglePlay"><i :class="isPlaying ? 'icon-pause' : 'icon-play'"></i></button>
    </div>

    <!-- ===== MOBILE: Bottom Nav ===== -->
    <div class="bottom-nav" v-if="mobileView === 'library'">
      <button class="nav-item" @click="mobileView = 'player'"><i class="icon-home"></i><span>首頁</span></button>
      <button class="nav-item"><i class="icon-search"></i><span>探索</span></button>
      <button class="nav-item active"><i class="icon-library"></i><span>音樂庫</span></button>
    </div>

    <!-- ===== DESKTOP LAYOUT ===== -->
    <div class="sidebar-desk">
      <div class="sd-header">
        <img src="/logo.png" alt="HeyBoss" />
        <div style="flex:1"><span class="brand">HeyBoss Audio</span><span class="sub">Music Player</span></div>
        <button @click="toggleTheme" class="sd-theme-btn" title="明/暗切換"><i :class="isDark ? 'icon-moon' : 'icon-sun'"></i></button>
      </div>
      <div class="sd-section">🎼 音樂庫</div>
      <div class="sd-list">
        <div v-for="(track, i) in tracks" :key="i" class="qi" :class="{ active: i === curIdx }" @click="playTrack(i)">
          <div class="qi-cover"><img src="/logo_blue_clean.png" alt="" /></div>
          <div class="qi-info">
            <div class="qi-name">{{ track.title }}</div>
            <div class="qi-meta">{{ track.artist }} · {{ track.duration }}</div>
          </div>
        </div>
      </div>
    </div>
    <div class="desk-main">
      <div class="desk-lyrics">
        <div class="lv-bg"></div>
        <div class="lv-scroll" ref="lyricsScrollD" style="height:100%">
          <template v-if="curIdx >= 0">
            <div style="height:40vh"></div>
            <div v-for="(line, i) in currentLyrics" :key="i" class="lv-line"
              :class="{ active: i === activeLyricIdx, passed: i < activeLyricIdx }"
              @click="seekToLyric(i)" ref="lyricLinesD">
              {{ line.text }}
            </div>
            <div style="height:40vh"></div>
          </template>
          <div v-else style="display:flex;align-items:center;justify-content:center;height:100%;color:var(--text3);font-size:18px">
            <div style="text-align:center"><i class="icon-music" style="font-size:48px;display:block;margin-bottom:12px;color:var(--gold);opacity:0.5"></i>選擇歌曲開始播放</div>
          </div>
        </div>
      </div>
      <div class="desk-player">
        <div class="dp-info">
          <div class="dp-cover"><img src="/logo_blue_clean.png" alt="" /></div>
          <div class="dp-text">
            <div class="t">{{ currentTrack?.title || 'HeyBoss Audio' }}</div>
            <div class="s">{{ currentTrack?.artist || '選擇歌曲' }}</div>
          </div>
        </div>
        <div class="dp-center">
          <div class="dp-btns">
            <button class="dp-btn" :class="{ active: isShuffle }" @click="isShuffle = !isShuffle"><i class="icon-shuffle"></i></button>
            <button class="dp-btn" @click="prevTrack"><i class="icon-skip-back"></i></button>
            <button class="dp-btn play" @click="togglePlay"><i :class="isPlaying ? 'icon-pause' : 'icon-play'"></i></button>
            <button class="dp-btn" @click="nextTrack"><i class="icon-skip-forward"></i></button>
            <button class="dp-btn" :class="{ active: playMode > 0 }" @click="cycleMode"><i :class="playMode === 2 ? 'icon-repeat-1' : 'icon-repeat'"></i></button>
          </div>
          <div class="dp-prog">
            <span class="dp-time">{{ fmt(currentTime) }}</span>
            <div class="dp-bar" @click="seekDesk($event)"><div class="dp-fill" :style="{ width: progress + '%' }"></div></div>
            <span class="dp-time">{{ fmt(duration) }}</span>
          </div>
        </div>
        <div class="dp-right">
          <button @click="toggleMute"><i :class="volIcon"></i></button>
          <input type="range" min="0" max="1" step="0.01" :value="volume" @input="setVol($event.target.value)" />
        </div>
      </div>
    </div>

    <audio ref="audioEl"></audio>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch, nextTick } from 'vue'

const isDark = ref(false)
const mobileView = ref('player') // player | lyrics | library
const isFav = ref(false)
const isShuffle = ref(false)
const playMode = ref(0) // 0=normal, 1=repeat-all, 2=repeat-one
const isPlaying = ref(false)
const currentTime = ref(0)
const duration = ref(0)
const volume = ref(0.8)
const isMuted = ref(false)
const curIdx = ref(-1)
const activeLyricIdx = ref(-1)

const audioEl = ref(null)
const lyricsScrollM = ref(null)
const lyricsScrollD = ref(null)
const lyricLinesM = ref([])
const lyricLinesD = ref([])

const tracks = ref([
  { title: '全新生活 馬上腰瘦', artist: 'HeyBoss Audio', file: '/audio/01-全新生活馬上腰瘦.mp3', lrcFile: '/audio/01-全新生活馬上腰瘦.lrc', duration: '2:10', album: '全新生活專輯' },
  { title: '動起來 燃燒吧', artist: 'HeyBoss Audio', file: '/audio/02-動起來燃燒吧.mp3', lrcFile: '/audio/02-動起來燃燒吧.lrc', duration: '1:59', album: '全新生活專輯' },
  { title: '跑出新自己', artist: 'HeyBoss Audio', file: '/audio/03-跑出新自己.mp3', lrcFile: '/audio/03-跑出新自己.lrc', duration: '1:45', album: '全新生活專輯' },
  { title: '瑜伽時光', artist: 'HeyBoss Audio', file: '/audio/04-瑜伽時光.mp3', lrcFile: '/audio/04-瑜伽時光.lrc', duration: '2:17', album: '全新生活專輯' }
])

const currentTrack = computed(() => curIdx.value >= 0 ? tracks.value[curIdx.value] : null)
const progress = computed(() => duration.value > 0 ? (currentTime.value / duration.value) * 100 : 0)
const volIcon = computed(() => {
  if (isMuted.value || volume.value === 0) return 'icon-volume-x'
  if (volume.value < 0.5) return 'icon-volume-1'
  return 'icon-volume-2'
})

// Lyrics storage
const lyricsCache = ref({})
const currentLyrics = computed(() => {
  if (curIdx.value < 0) return []
  return lyricsCache.value[curIdx.value] || []
})

function parseLRC(text) {
  const lines = []
  text.split('\n').forEach(line => {
    const match = line.match(/\[(\d{2}):(\d{2})\.(\d{2})\](.*)/)
    if (match) {
      const time = parseInt(match[1]) * 60 + parseInt(match[2]) + parseInt(match[3]) / 100
      const txt = match[4].trim()
      if (txt) lines.push({ t: time, text: txt })
    }
  })
  return lines
}

async function loadLyrics(idx) {
  if (lyricsCache.value[idx]) return
  try {
    const res = await fetch(tracks.value[idx].lrcFile)
    const text = await res.text()
    lyricsCache.value[idx] = parseLRC(text)
  } catch (e) {
    lyricsCache.value[idx] = []
  }
}

function fmt(s) {
  if (isNaN(s)) return '0:00'
  return Math.floor(s / 60) + ':' + String(Math.floor(s % 60)).padStart(2, '0')
}

function toggleTheme() {
  isDark.value = !isDark.value
  localStorage.setItem('theme', isDark.value ? 'dark' : 'light')
}

async function playTrack(i) {
  curIdx.value = i
  activeLyricIdx.value = -1
  await loadLyrics(i)
  const audio = audioEl.value
  audio.src = tracks.value[i].file
  audio.play()
  mobileView.value = 'player'
}

function togglePlay() {
  const audio = audioEl.value
  if (audio.paused || !audio.src) {
    if (curIdx.value < 0) playTrack(0)
    else audio.play()
  } else {
    audio.pause()
  }
}

function prevTrack() {
  if (curIdx.value <= 0) playTrack(tracks.value.length - 1)
  else playTrack(curIdx.value - 1)
}

function nextTrack() {
  if (isShuffle.value) {
    let n
    do { n = Math.floor(Math.random() * tracks.value.length) } while (n === curIdx.value && tracks.value.length > 1)
    playTrack(n)
  } else {
    playTrack((curIdx.value + 1) % tracks.value.length)
  }
}

function cycleMode() { playMode.value = (playMode.value + 1) % 3 }

function setVol(v) {
  volume.value = parseFloat(v)
  audioEl.value.volume = volume.value
  isMuted.value = false
}

function toggleMute() {
  isMuted.value = !isMuted.value
  audioEl.value.muted = isMuted.value
}

function seek(e) {
  const rect = e.currentTarget.getBoundingClientRect()
  const pct = Math.max(0, Math.min(1, (e.clientX - rect.left) / rect.width))
  if (!isNaN(duration.value)) audioEl.value.currentTime = pct * duration.value
}

function seekDesk(e) { seek(e) }

function seekToLyric(i) {
  if (curIdx.value >= 0 && currentLyrics.value[i]) {
    audioEl.value.currentTime = currentLyrics.value[i].t
    if (audioEl.value.paused) audioEl.value.play()
  }
}

function scrollToActiveLyric() {
  nextTick(() => {
    ;[lyricsScrollM.value, lyricsScrollD.value].forEach(container => {
      if (!container) return
      const lines = container.querySelectorAll('.lv-line')
      const activeLine = lines[activeLyricIdx.value]
      if (activeLine) {
        container.scrollTo({
          top: activeLine.offsetTop - container.clientHeight / 2 + activeLine.clientHeight / 2,
          behavior: 'smooth'
        })
      }
    })
  })
}

onMounted(() => {
  const saved = localStorage.getItem('theme')
  if (saved === 'dark' || (!saved && window.matchMedia('(prefers-color-scheme: dark)').matches)) {
    isDark.value = true
  }

  const audio = audioEl.value
  audio.volume = volume.value

  audio.addEventListener('timeupdate', () => {
    currentTime.value = audio.currentTime
    duration.value = audio.duration
    // Update active lyric
    const lyrics = currentLyrics.value
    let ai = -1
    for (let i = lyrics.length - 1; i >= 0; i--) {
      if (audio.currentTime >= lyrics[i].t) { ai = i; break }
    }
    if (ai !== activeLyricIdx.value) {
      activeLyricIdx.value = ai
      scrollToActiveLyric()
    }
  })

  audio.addEventListener('play', () => { isPlaying.value = true })
  audio.addEventListener('pause', () => { isPlaying.value = false })
  audio.addEventListener('ended', () => {
    if (playMode.value === 2) { audio.currentTime = 0; audio.play() }
    else if (playMode.value === 1 || curIdx.value < tracks.value.length - 1) nextTrack()
  })

  // Preload all lyrics
  tracks.value.forEach((_, i) => loadLyrics(i))

  if ('serviceWorker' in navigator) navigator.serviceWorker.register('/sw.js')
})
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;600;700;800&display=swap');
*{margin:0;padding:0;box-sizing:border-box}
:root{--blue:#005FDF;--gold:#EDC626;--dark-blue:#003E8F;--bg:#f5f7fa;--white:#ffffff;--card:#ffffff;--border:#e5e7eb;--text1:#111827;--text2:#4b5563;--text3:#9ca3af;--cover-bg:#ffffff;--lyrics-bg:#003E8F;--prog-color:#005FDF;--play-bg:#005FDF;--play-color:#fff}
[data-theme="dark"]{--bg:#0a0a0a;--white:#1a1a1a;--card:#1a1a1a;--border:#2a2a2a;--text1:#f0f0f0;--text2:#a0a0a0;--text3:#555;--cover-bg:#ffffff;--lyrics-bg:#0a0a14;--prog-color:#EDC626;--play-bg:#EDC626;--play-color:#111}
body{font-family:'Noto Sans TC',-apple-system,sans-serif;background:var(--bg);color:var(--text1);min-height:100vh;overflow:hidden}
.app{height:100vh;display:flex;flex-direction:column}

/* ===== VIEWS ===== */
.view{display:none;flex:1;overflow:hidden;flex-direction:column}
.view.active{display:flex}

/* ===== PLAYER VIEW ===== */
.player-view{align-items:center;justify-content:center;padding:20px;position:relative}
.pv-bg{position:absolute;inset:0;background:linear-gradient(180deg,var(--white) 0%,var(--bg) 100%)}
.pv-top{position:absolute;top:env(safe-area-inset-top,12px);left:16px;right:16px;display:flex;justify-content:space-between;align-items:center;z-index:2}
.pv-top button{background:none;border:none;color:var(--text2);font-size:20px;padding:8px;cursor:pointer}
.pv-content{z-index:1;text-align:center;width:100%;max-width:400px}
.pv-cover{width:280px;height:280px;border-radius:50%;margin:0 auto 24px;background:var(--cover-bg);display:flex;align-items:center;justify-content:center;box-shadow:0 16px 50px rgba(0,0,0,0.15);overflow:hidden;transition:box-shadow 0.3s}
[data-theme="dark"] .pv-cover{box-shadow:0 0 30px rgba(237,198,38,0.25);background:#ffffff}
.pv-cover.spin{animation:coverSpin 8s linear infinite}
[data-theme="dark"] .pv-cover.spin{box-shadow:0 0 40px rgba(237,198,38,0.5),0 0 80px rgba(237,198,38,0.15)}
.pv-cover img{width:65%;height:65%;object-fit:contain}
@keyframes coverSpin{from{transform:rotate(0)}to{transform:rotate(360deg)}}
.pv-title{font-size:22px;font-weight:800;margin-bottom:4px}
.pv-artist{font-size:14px;color:var(--text2)}
.pv-actions{display:flex;justify-content:center;gap:32px;margin:20px 0 16px}
.pv-act{background:none;border:none;color:var(--text2);font-size:20px;cursor:pointer;display:flex;flex-direction:column;align-items:center;gap:4px;transition:color 0.2s}
.pv-act span{font-size:10px}
.pv-act:hover,.pv-act.active{color:var(--gold)}
.pv-progress{width:100%;margin-bottom:4px}
.pv-prog-bar{width:100%;height:4px;background:var(--border);border-radius:2px;cursor:pointer;position:relative}
.pv-prog-bar:hover{height:6px}
.pv-prog-fill{height:100%;background:var(--prog-color);border-radius:2px;transition:width 0.1s}
.pv-times{display:flex;justify-content:space-between;font-size:11px;color:var(--text3);margin-top:6px;font-variant-numeric:tabular-nums}
.pv-controls{display:flex;align-items:center;justify-content:center;gap:24px;margin-top:16px}
.pv-ctrl{background:none;border:none;color:var(--text2);font-size:22px;cursor:pointer;padding:8px;transition:all 0.2s}
.pv-ctrl:hover{color:var(--text1)}
.pv-ctrl.active{color:var(--gold)}
.pv-ctrl.main-play{font-size:28px;background:var(--play-bg);color:var(--play-color);width:60px;height:60px;border-radius:50%;display:flex;align-items:center;justify-content:center}
.pv-ctrl.main-play:hover{transform:scale(1.06)}
.pv-vol{display:flex;align-items:center;justify-content:center;gap:8px;margin-top:16px}
.pv-vol button{background:none;border:none;color:var(--text3);font-size:16px;cursor:pointer}
.pv-vol input{-webkit-appearance:none;width:100px;height:3px;border-radius:2px;background:var(--border);outline:none}
.pv-vol input::-webkit-slider-thumb{-webkit-appearance:none;width:10px;height:10px;border-radius:50%;background:var(--text1);cursor:pointer}

/* ===== LYRICS VIEW ===== */
.lyrics-fullview{background:var(--lyrics-bg);position:relative}
.lv-bg{position:absolute;inset:0;background:radial-gradient(circle at 50% 40%,rgba(0,95,223,0.2),transparent 70%)}
.lv-header{position:relative;z-index:2;padding:16px 20px;display:flex;justify-content:space-between;align-items:center}
.lv-header button{background:none;border:none;color:rgba(255,255,255,0.6);font-size:18px;cursor:pointer}
.lv-header .song{text-align:center;font-size:14px;font-weight:700;color:#fff}
.lv-header .artist{text-align:center;font-size:11px;color:rgba(255,255,255,0.5)}
.lv-scroll{position:relative;z-index:1;flex:1;overflow-y:auto;padding:0 24px;
  mask-image:linear-gradient(transparent 3%,black 20%,black 80%,transparent 97%);
  -webkit-mask-image:linear-gradient(transparent 3%,black 20%,black 80%,transparent 97%)}
.lv-scroll::-webkit-scrollbar{display:none}
.lv-line{padding:14px 16px;text-align:center;font-size:20px;font-weight:500;color:rgba(255,255,255,0.2);transition:all 0.5s cubic-bezier(0.4,0,0.2,1);cursor:pointer;line-height:1.6;border-radius:8px}
.lv-line:hover{color:rgba(255,255,255,0.4)}
.lv-line.active{color:#fff;font-size:26px;font-weight:800;transform:scale(1.03);text-shadow:0 0 30px rgba(237,198,38,0.5)}
.lv-line.passed{color:rgba(255,255,255,0.12)}

/* ===== LIBRARY VIEW ===== */
.library-view{background:var(--bg)}
.lib-header{padding:20px 20px 0}
.lib-header h2{font-size:24px;font-weight:800;margin-bottom:16px}
.lib-tabs{display:flex;gap:8px;margin-bottom:12px}
.lib-tab{padding:8px 16px;border-radius:20px;border:none;background:var(--card);color:var(--text2);font-size:13px;font-weight:600;cursor:pointer;transition:all 0.2s}
.lib-tab.active{background:var(--blue);color:#fff}
[data-theme="dark"] .lib-tab.active{background:var(--gold);color:#111}
.lib-list{flex:1;overflow-y:auto;padding:8px 12px}
.lib-list::-webkit-scrollbar{display:none}
.lib-item{display:flex;align-items:center;gap:12px;padding:12px 8px;border-radius:12px;cursor:pointer;transition:background 0.2s}
.lib-item:hover{background:var(--card)}
.lib-item.active{background:rgba(0,95,223,0.08)}
[data-theme="dark"] .lib-item.active{background:rgba(237,198,38,0.08)}
.lib-cover{width:52px;height:52px;border-radius:12px;background:var(--cover-bg);display:flex;align-items:center;justify-content:center;flex-shrink:0;overflow:hidden}
.lib-cover img{width:65%;height:65%;object-fit:contain}
.lib-info{flex:1;min-width:0}
.lib-name{font-size:15px;font-weight:600;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.lib-item.active .lib-name{color:var(--blue)}
[data-theme="dark"] .lib-item.active .lib-name{color:var(--gold)}
.lib-meta{font-size:12px;color:var(--text3);margin-top:2px}
.lib-more{background:none;border:none;color:var(--text3);font-size:18px;cursor:pointer;padding:4px}

/* ===== BOTTOM MINI PLAYER ===== */
.bottom-mini{display:flex;background:var(--white);border-top:1px solid var(--border);padding:8px 12px;align-items:center;gap:10px;cursor:pointer}
.bm-cover{width:44px;height:44px;border-radius:12px;background:var(--cover-bg);display:flex;align-items:center;justify-content:center;flex-shrink:0;overflow:hidden}
.bm-cover img{width:65%;height:65%;object-fit:contain}
.bm-info{flex:1;min-width:0}
.bm-info .t{font-size:13px;font-weight:600;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.bm-info .s{font-size:10px;color:var(--text3)}
.bm-play{background:none;border:none;color:var(--text1);font-size:22px;cursor:pointer;padding:4px}

/* ===== BOTTOM NAV ===== */
.bottom-nav{display:flex;background:var(--white);border-top:1px solid var(--border);padding:8px 0 env(safe-area-inset-bottom,8px)}
.nav-item{flex:1;display:flex;flex-direction:column;align-items:center;gap:2px;background:none;border:none;color:var(--text3);font-size:18px;cursor:pointer;padding:4px}
.nav-item span{font-size:10px}
.nav-item.active{color:var(--blue)}
[data-theme="dark"] .nav-item.active{color:var(--gold)}

/* ===== DESKTOP SIDEBAR ===== */
.sidebar-desk{display:none;flex-direction:column;width:320px;min-width:320px;border-right:1px solid var(--border);background:var(--white)}
.sd-header{padding:16px 20px;display:flex;align-items:center;gap:10px;border-bottom:1px solid var(--border)}
.sd-header img{height:32px;border-radius:6px}
.sd-header .brand{font-size:15px;font-weight:800;color:var(--blue)}
[data-theme="dark"] .sd-header .brand{color:var(--gold)}
.sd-header .sub{font-size:10px;color:var(--text3);display:block}
.sd-theme-btn{background:none;border:none;color:var(--text2);font-size:18px;cursor:pointer;padding:4px}
.sd-section{padding:14px 20px 6px;font-size:11px;color:var(--text3);letter-spacing:1px;font-weight:600}
.sd-list{flex:1;overflow-y:auto;padding:4px 8px}
.sd-list::-webkit-scrollbar{width:4px}
.sd-list::-webkit-scrollbar-thumb{background:var(--border);border-radius:2px}
.qi{display:flex;align-items:center;gap:12px;padding:10px 8px;border-radius:10px;cursor:pointer;transition:background 0.2s}
.qi:hover{background:rgba(0,0,0,0.03)}
[data-theme="dark"] .qi:hover{background:rgba(255,255,255,0.03)}
.qi.active{background:rgba(0,95,223,0.1)}
[data-theme="dark"] .qi.active{background:rgba(237,198,38,0.1)}
.qi-cover{width:40px;height:40px;border-radius:8px;background:var(--cover-bg);display:flex;align-items:center;justify-content:center;flex-shrink:0;overflow:hidden}
.qi-cover img{width:60%;height:60%;object-fit:contain}
.qi-info{flex:1;min-width:0}
.qi-name{font-size:14px;font-weight:600;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.qi.active .qi-name{color:var(--blue)}
[data-theme="dark"] .qi.active .qi-name{color:var(--gold)}
.qi-meta{font-size:11px;color:var(--text3);margin-top:2px}

/* ===== DESKTOP MAIN ===== */
.desk-main{display:none;flex:1;flex-direction:column}
.desk-lyrics{flex:1;overflow:hidden;position:relative;background:var(--lyrics-bg)}
.desk-player{border-top:1px solid var(--border);background:var(--white);padding:12px 24px;display:flex;align-items:center;gap:16px}
.dp-info{min-width:200px;display:flex;align-items:center;gap:12px}
.dp-cover{width:48px;height:48px;border-radius:10px;background:var(--cover-bg);display:flex;align-items:center;justify-content:center;overflow:hidden}
.dp-cover img{width:60%;height:60%;object-fit:contain}
.dp-text .t{font-size:13px;font-weight:700}
.dp-text .s{font-size:11px;color:var(--text3)}
.dp-center{flex:1;display:flex;flex-direction:column;align-items:center;gap:6px;max-width:600px;margin:0 auto}
.dp-btns{display:flex;align-items:center;gap:14px}
.dp-btn{background:none;border:none;color:var(--text2);font-size:18px;cursor:pointer;padding:4px;transition:all 0.2s}
.dp-btn:hover{color:var(--text1)}
.dp-btn.active{color:var(--gold)}
.dp-btn.play{font-size:22px;background:var(--play-bg);color:var(--play-color);width:36px;height:36px;border-radius:50%;display:flex;align-items:center;justify-content:center}
.dp-prog{display:flex;align-items:center;gap:8px;width:100%}
.dp-time{font-size:11px;color:var(--text3);min-width:36px;text-align:center;font-variant-numeric:tabular-nums}
.dp-bar{flex:1;height:4px;background:var(--border);border-radius:2px;cursor:pointer}
.dp-bar:hover{height:5px}
.dp-fill{height:100%;background:var(--prog-color);border-radius:2px;transition:width 0.1s}
.dp-right{min-width:140px;display:flex;align-items:center;gap:6px;justify-content:flex-end}
.dp-right button{background:none;border:none;color:var(--text3);font-size:16px;cursor:pointer}
.dp-right input{-webkit-appearance:none;width:80px;height:3px;border-radius:2px;background:var(--border);outline:none}
.dp-right input::-webkit-slider-thumb{-webkit-appearance:none;width:10px;height:10px;border-radius:50%;background:var(--text1);cursor:pointer}

/* ===== RESPONSIVE ===== */
@media(min-width:769px){
  .app{flex-direction:row}
  .sidebar-desk{display:flex}
  .desk-main{display:flex}
  /* hide mobile elements */
  .player-view,.lyrics-fullview,.library-view,.bottom-mini,.bottom-nav{display:none !important}
}
@media(max-width:768px){
  .sidebar-desk,.desk-main{display:none !important}
}
</style>
