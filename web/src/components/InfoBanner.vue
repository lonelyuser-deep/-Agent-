<template>
  <transition-group name="banner" tag="div" class="info-banner">
    <div
      v-for="msg in visibleMessages"
      :key="msg.id"
      class="banner-item"
      :class="msg.level"
    >
      <span class="banner-icon" v-html="iconFor(msg.level)"></span>
      <div class="banner-body">
        <span class="banner-title">{{ msg.title }}</span>
        <span class="banner-desc">{{ msg.desc }}</span>
      </div>
      <button class="banner-close" @click="dismiss(msg.id)" aria-label="关闭提示">
        <svg viewBox="0 0 16 16" fill="none" width="14" height="14">
          <path d="M4 4L12 12M12 4L4 12" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" />
        </svg>
      </button>
    </div>
  </transition-group>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'

type Level = 'info' | 'warn' | 'success'

interface Message {
  id: string
  level: Level
  title: string
  desc: string
}

const props = defineProps<{ messages: Message[] }>()

const STORAGE_KEY = 'info_banner_dismissed'
const dismissed = ref<Set<string>>(new Set())

onMounted(() => {
  try {
    const raw = localStorage.getItem(STORAGE_KEY)
    if (raw) dismissed.value = new Set(JSON.parse(raw))
  } catch { dismissed.value = new Set() }
})

const visibleMessages = computed(() =>
  props.messages.filter(m => !dismissed.value.has(m.id))
)

function dismiss(id: string) {
  dismissed.value.add(id)
  localStorage.setItem(STORAGE_KEY, JSON.stringify([...dismissed.value]))
  dismissed.value = new Set(dismissed.value)
}

function iconFor(level: Level): string {
  if (level === 'warn') return '<svg viewBox="0 0 16 16" fill="none" width="16" height="16"><path d="M8 2L15 14H1L8 2Z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/><path d="M8 7V10M8 12.5V12.6" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/></svg>'
  if (level === 'success') return '<svg viewBox="0 0 16 16" fill="none" width="16" height="16"><circle cx="8" cy="8" r="6.5" stroke="currentColor" stroke-width="1.5"/><path d="M5 8L7 10L11 6" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></svg>'
  return '<svg viewBox="0 0 16 16" fill="none" width="16" height="16"><circle cx="8" cy="8" r="6.5" stroke="currentColor" stroke-width="1.5"/><path d="M8 7.5V11M8 5V5.1" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/></svg>'
}
</script>

<style scoped>
.info-banner {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 20px;
}

.banner-item {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  padding: 14px 16px;
  border-radius: var(--radius-md);
  border: 1px solid var(--border-soft);
  background: var(--white);
  box-shadow: var(--shadow-sm);
  position: relative;
  overflow: hidden;
}
.banner-item::before {
  content: '';
  position: absolute;
  left: 0;
  top: 0;
  bottom: 0;
  width: 3px;
}
.banner-item.info::before { background: var(--ink-soft); }
.banner-item.warn::before { background: var(--gold); }
.banner-item.success::before { background: #5A8A6B; }

.banner-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  border-radius: 8px;
  flex-shrink: 0;
  margin-top: 1px;
}
.banner-item.info .banner-icon { background: var(--jade-light); color: var(--ink-soft); }
.banner-item.warn .banner-icon { background: #F5EBDD; color: var(--gold-deep); }
.banner-item.success .banner-icon { background: #E8F0EA; color: #4A7A5B; }

.banner-body {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 2px;
  min-width: 0;
}
.banner-title {
  font-size: 14px;
  font-weight: 600;
  color: var(--ink);
}
.banner-desc {
  font-size: 12.5px;
  color: var(--ink-soft);
  line-height: 1.5;
}

.banner-close {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 24px;
  height: 24px;
  border-radius: 6px;
  color: var(--ink-hint);
  background: transparent;
  flex-shrink: 0;
}
.banner-close:hover {
  background: var(--jade-light);
  color: var(--ink);
}

.banner-enter-active,
.banner-leave-active {
  transition: all 0.35s cubic-bezier(0.4, 0, 0.2, 1);
}
.banner-enter-from {
  opacity: 0;
  transform: translateY(-10px);
}
.banner-leave-to {
  opacity: 0;
  transform: translateX(20px);
}
</style>