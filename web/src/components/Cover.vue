<template>
  <div class="cover" :class="{ leaving: isLeaving }">
    <video
      ref="videoEl"
      class="cover-video"
      :class="{ zoom: isLeaving }"
      autoplay
      muted
      loop
      playsinline
      preload="auto"
    >
      <source src="/intro.mp4" type="video/mp4" />
    </video>

    <div class="cover-veil"></div>

    <div class="cover-content" :class="{ leaving: isLeaving }">
      <div class="cover-mark">
        <span class="mark-line"></span>
        <span class="mark-dot"></span>
        <span class="mark-line"></span>
      </div>
      <h1 class="cover-title">智云校遇</h1>
      <p class="cover-subtitle">遇见更好的自己</p>
      <button class="enter-btn" @click="handleEnter">
        <span class="enter-btn-text">进 入</span>
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'

const emit = defineEmits<{ entered: [] }>()
const isLeaving = ref(false)
const videoEl = ref<HTMLVideoElement | null>(null)

onMounted(() => {
  if (videoEl.value) {
    videoEl.value.play().catch(() => {})
  }
})

function handleEnter() {
  if (isLeaving.value) return
  isLeaving.value = true
  const reduceMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches
  const delay = reduceMotion ? 100 : 500
  setTimeout(() => { emit('entered') }, delay)
}
</script>

<style scoped>
.cover {
  position: fixed;
  inset: 0;
  z-index: 1000;
  overflow: hidden;
  background: #0a0a0a;
  transition: opacity 0.5s cubic-bezier(0.4, 0, 0.2, 1);
}
.cover.leaving {
  opacity: 0;
}

.cover-video {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.5s cubic-bezier(0.4, 0, 0.2, 1), filter 0.5s ease;
}
.cover-video.zoom {
  transform: scale(1.06);
  filter: brightness(0.7);
}

.cover-veil {
  position: absolute;
  inset: 0;
  background:
    linear-gradient(
      180deg,
      rgba(12, 12, 12, 0.45) 0%,
      rgba(12, 12, 12, 0.72) 22%,
      rgba(12, 12, 12, 0.88) 38%,
      rgba(12, 12, 12, 0.82) 52%,
      rgba(12, 12, 12, 0.5) 72%,
      rgba(12, 12, 12, 0.25) 100%
    );
}

.cover-content {
  position: absolute;
  top: 38%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 10;
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  transition: transform 0.5s cubic-bezier(0.4, 0, 0.2, 1), opacity 0.5s ease, filter 0.5s ease;
}
.cover-content.leaving {
  transform: translate(-50%, -54%);
  opacity: 0;
  filter: blur(10px);
}

.cover-mark {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 28px;
  opacity: 0.7;
}
.mark-line {
  width: 48px;
  height: 1px;
  background: linear-gradient(90deg, transparent, rgba(212, 184, 150, 0.9), transparent);
}
.mark-dot {
  width: 5px;
  height: 5px;
  border-radius: 50%;
  background: #D4B896;
  box-shadow: 0 0 8px rgba(212, 184, 150, 0.6);
}

.cover-title {
  font-size: clamp(44px, 8vw, 76px);
  font-weight: 800;
  color: #F5F2EC;
  letter-spacing: 12px;
  text-indent: 12px;
  text-shadow: 0 4px 30px rgba(0, 0, 0, 0.5);
  margin-bottom: 18px;
  font-family: var(--font-sans);
}

.cover-subtitle {
  font-size: clamp(15px, 2.2vw, 20px);
  color: rgba(245, 242, 236, 0.78);
  letter-spacing: 6px;
  text-indent: 6px;
  margin-bottom: 52px;
  font-weight: 300;
}

.enter-btn {
  position: relative;
  padding: 15px 64px;
  border-radius: 4px;
  background: rgba(12, 12, 12, 0.38);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  border: 1px solid rgba(245, 242, 236, 0.28);
  color: #F5F2EC;
  font-size: 16px;
  font-weight: 500;
  letter-spacing: 8px;
  text-indent: 8px;
  box-shadow: inset 0 1px 0 rgba(245, 242, 236, 0.12), 0 6px 28px rgba(0, 0, 0, 0.25);
  transition: all 0.35s cubic-bezier(0.4, 0, 0.2, 1);
  overflow: hidden;
}
.enter-btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(245, 242, 236, 0.14), transparent);
  transition: left 0.6s ease;
}
.enter-btn:hover {
  background: rgba(12, 12, 12, 0.5);
  border-color: rgba(212, 184, 150, 0.55);
  box-shadow: inset 0 1px 0 rgba(245, 242, 236, 0.18), 0 10px 40px rgba(0, 0, 0, 0.35), 0 0 24px rgba(212, 184, 150, 0.25);
  transform: translateY(-2px);
}
.enter-btn:hover::before {
  left: 100%;
}
.enter-btn:active {
  transform: translateY(0);
}

@media (prefers-reduced-motion: reduce) {
  .cover,
  .cover-video,
  .cover-content {
    transition: opacity 0.2s ease !important;
  }
  .cover-video.zoom {
    transform: none;
    filter: none;
  }
  .cover-content.leaving {
    transform: translate(-50%, -50%);
    filter: none;
  }
}

@media (max-width: 768px) {
  .cover-title {
    letter-spacing: 8px;
    text-indent: 8px;
  }
  .cover-subtitle {
    letter-spacing: 4px;
    text-indent: 4px;
    margin-bottom: 40px;
  }
  .enter-btn {
    padding: 13px 48px;
    font-size: 15px;
    letter-spacing: 6px;
    text-indent: 6px;
  }
  .mark-line {
    width: 32px;
  }
}
</style>