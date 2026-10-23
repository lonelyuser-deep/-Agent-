<template>
  <div class="home">
    <InfoBanner :messages="infoMessages" />

    <div class="welcome-banner fade-in-up">
      <div class="banner-left">
        <span class="banner-tag">CAMPUS DELIVERY</span>
        <h1 class="banner-title">今天还有几件快递待领取？</h1>
        <p class="banner-desc">管理你的校园快递，取件码一目了然，不再错过每一件包裹</p>
        <div class="banner-actions">
          <button class="btn-primary" @click="goAdd">
            <svg viewBox="0 0 16 16" fill="none" width="16" height="16" style="display:inline;vertical-align:-2px;margin-right:6px"><path d="M8 3V13M3 8H13" stroke="currentColor" stroke-width="2" stroke-linecap="round"/></svg>
            添加快递
          </button>
          <button class="btn-outline" @click="goAdd">短信智能解析</button>
          <button class="btn-ghost" @click="goReminder">设置提醒</button>
        </div>
      </div>
      <div class="banner-right">
        <StatsChart
          :pending="stats.pending"
          :picked="stats.picked"
          :total="stats.total"
          :trend="trend"
        />
      </div>
    </div>

    <div class="quick-actions fade-in-up" style="animation-delay:0.05s">
      <div
        v-for="a in quickActions"
        :key="a.key"
        class="action-card"
        @click="a.handler()"
      >
        <div class="action-icon" v-html="a.icon"></div>
        <div class="action-text">
          <span class="action-label">{{ a.label }}</span>
          <span class="action-desc">{{ a.desc }}</span>
        </div>
        <svg class="action-arrow" viewBox="0 0 16 16" fill="none" width="16" height="16">
          <path d="M6 4L10 8L6 12" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
      </div>
    </div>

    <div v-if="pendingList.length > 0" class="section fade-in-up" style="animation-delay:0.1s">
      <div class="section-header">
        <h2 class="section-title">待取快递</h2>
        <span class="section-count">{{ pendingList.length }} 件</span>
      </div>
      <div class="card-list">
        <ExpressCard v-for="item in pendingList" :key="item.id" :item="item" @click="goDetail(item.id)" @pick="quickPick(item)" />
      </div>
    </div>

    <div v-if="pendingList.length === 0 && pickedList.length === 0" class="empty-state fade-in">
      <svg viewBox="0 0 120 120" fill="none" width="100" height="100" class="empty-icon">
        <rect x="20" y="40" width="80" height="60" rx="8" stroke="var(--border)" stroke-width="2"/>
        <path d="M20 40L60 20L100 40" stroke="var(--border)" stroke-width="2" stroke-linejoin="round"/>
        <rect x="48" y="60" width="24" height="20" rx="3" fill="var(--jade-light)" stroke="var(--border)" stroke-width="1.5"/>
        <path d="M40 40V30M80 40V30" stroke="var(--border)" stroke-width="2" stroke-linecap="round"/>
        <path d="M52 70L58 76L68 66" stroke="var(--gold)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
      </svg>
      <h3 class="empty-title">还没有快递记录</h3>
      <p class="empty-desc">添加你的第一件快递，开始轻松管理取件</p>
      <button class="btn-primary" @click="goAdd">添加第一件快递</button>
    </div>

    <div v-if="pickedList.length > 0" class="section">
      <div class="section-header toggle" @click="showPicked = !showPicked">
        <h2 class="section-title">已取快递</h2>
        <div class="toggle-right">
          <span class="section-count">{{ pickedList.length }} 件</span>
          <svg viewBox="0 0 16 16" fill="none" width="16" height="16" class="toggle-arrow" :class="{ open: showPicked }">
            <path d="M4 6L8 10L12 6" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </div>
      </div>
      <transition name="collapse">
        <div v-if="showPicked" class="card-list">
          <ExpressCard v-for="item in pickedList" :key="item.id" :item="item" @click="goDetail(item.id)" />
        </div>
      </transition>
    </div>

    <div class="insights fade-in-up" style="animation-delay:0.15s">
      <div class="insight-card">
        <div class="insight-head">
          <span class="insight-title">驿站分布</span>
          <span class="insight-sub">按快递数量排序</span>
        </div>
        <div class="station-list" v-if="stationStats.length > 0">
          <div v-for="s in stationStats" :key="s.name" class="station-row">
            <span class="station-name">{{ s.name }}</span>
            <div class="station-bar-wrap">
              <div class="station-bar" :style="{ width: (s.count / stationStats[0].count * 100) + '%' }"></div>
            </div>
            <span class="station-count">{{ s.count }}</span>
          </div>
        </div>
        <div v-else class="insight-empty">暂无数据</div>
      </div>

      <div class="insight-card">
        <div class="insight-head">
          <span class="insight-title">取件效率</span>
          <span class="insight-sub">本周概览</span>
        </div>
        <div class="efficiency-grid">
          <div class="eff-item">
            <span class="eff-num">{{ stats.picked }}</span>
            <span class="eff-label">累计已取</span>
          </div>
          <div class="eff-item">
            <span class="eff-num">{{ weekPicked }}</span>
            <span class="eff-label">本周已取</span>
          </div>
          <div class="eff-item">
            <span class="eff-num">{{ avgPerDay }}</span>
            <span class="eff-label">日均取件</span>
          </div>
          <div class="eff-item">
            <span class="eff-num">{{ pickupRate }}%</span>
            <span class="eff-label">取件率</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import ExpressCard from '@/components/ExpressCard.vue'
import StatsChart from '@/components/StatsChart.vue'
import InfoBanner from '@/components/InfoBanner.vue'
import { getList, getStats, updateRecord } from '@/utils/storage'
import type { ExpressData } from '@shared/types/express'

const router = useRouter()
const pendingList = ref<ExpressData[]>([])
const pickedList = ref<ExpressData[]>([])
const stats = ref({ pending: 0, picked: 0, total: 0 })
const showPicked = ref(false)

const todayPicked = computed(() => {
  const today = new Date()
  return pickedList.value.filter(r => {
    if (!r.pickedAt) return false
    const d = new Date(r.pickedAt)
    return d.toDateString() === today.toDateString()
  }).length
})

const trend = computed(() => {
  const today = new Date()
  const result: number[] = []
  for (let i = 6; i >= 0; i--) {
    const d = new Date(today)
    d.setDate(today.getDate() - i)
    const key = d.toDateString()
    result.push(pickedList.value.filter(r => r.pickedAt && new Date(r.pickedAt).toDateString() === key).length)
  }
  return result
})

const weekPicked = computed(() => trend.value.reduce((a, b) => a + b, 0))
const avgPerDay = computed(() => (weekPicked.value / 7).toFixed(1))
const pickupRate = computed(() => stats.value.total === 0 ? 0 : Math.round((stats.value.picked / stats.value.total) * 100))

const stationStats = computed(() => {
  const map = new Map<string, number>()
  for (const r of [...pendingList.value, ...pickedList.value]) {
    const s = r.station || '未知驿站'
    map.set(s, (map.get(s) || 0) + 1)
  }
  return [...map.entries()]
    .map(([name, count]) => ({ name, count }))
    .sort((a, b) => b.count - a.count)
    .slice(0, 5)
})

const infoMessages = computed(() => {
  const msgs: { id: string; level: 'info' | 'warn' | 'success'; title: string; desc: string }[] = []
  if (stats.value.pending > 0) {
    msgs.push({
      id: 'pending',
      level: 'warn',
      title: `${stats.value.pending} 件快递待取`,
      desc: '点击下方列表快速取件，别让包裹在驿站等太久'
    })
  }
  if (todayPicked.value > 0) {
    msgs.push({
      id: 'today-picked',
      level: 'success',
      title: `今日已取 ${todayPicked.value} 件`,
      desc: '取件顺利，继续保持好习惯'
    })
  }
  if (stats.value.total === 0) {
    msgs.push({
      id: 'welcome',
      level: 'info',
      title: '欢迎使用智云校遇',
      desc: '添加你的第一件快递，开启高效取件管理'
    })
  }
  return msgs
})

const quickActions = [
  {
    key: 'add',
    label: '添加快递',
    desc: '手动录入取件信息',
    icon: '<svg viewBox="0 0 20 20" fill="none" width="20" height="20"><rect x="3" y="5" width="14" height="12" rx="2" stroke="currentColor" stroke-width="1.5"/><path d="M10 9V13M8 11H12" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/></svg>',
    handler: () => router.push('/add'),
  },
  {
    key: 'parse',
    label: '短信解析',
    desc: '智能识别取件短信',
    icon: '<svg viewBox="0 0 20 20" fill="none" width="20" height="20"><path d="M3 5H17V13H11L7 16V13H3V5Z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/><path d="M6 8H14M6 10H11" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/></svg>',
    handler: () => router.push('/add'),
  },
  {
    key: 'reminder',
    label: '取件提醒',
    desc: '设置时间不再错过',
    icon: '<svg viewBox="0 0 20 20" fill="none" width="20" height="20"><path d="M10 3C6.5 3 4 5.5 4 9V13L2 16H18L16 13V9C16 5.5 13.5 3 10 3Z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/><path d="M8 17C8 18 9 19 10 19C11 19 12 18 12 17" stroke="currentColor" stroke-width="1.5"/></svg>',
    handler: () => router.push('/reminder'),
  },
  {
    key: 'settings',
    label: '偏好设置',
    desc: '个性化你的助手',
    icon: '<svg viewBox="0 0 20 20" fill="none" width="20" height="20"><circle cx="10" cy="10" r="3" stroke="currentColor" stroke-width="1.5"/><path d="M10 2V4M10 16V18M2 10H4M16 10H18M4.5 4.5L5.8 5.8M14.2 14.2L15.5 15.5M4.5 15.5L5.8 14.2M14.2 5.8L15.5 4.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/></svg>',
    handler: () => router.push('/settings'),
  },
]

function loadData() {
  const list = getList()
  pendingList.value = list.filter(r => r.status === 'pending')
  pickedList.value = list.filter(r => r.status === 'picked')
  stats.value = getStats()
}

function goAdd() { router.push('/add') }
function goReminder() { router.push('/reminder') }
function goDetail(id: string) { router.push(`/detail/${id}`) }
function quickPick(item: ExpressData) {
  updateRecord(item.id, { status: 'picked', pickedAt: Date.now() })
  loadData()
  window.dispatchEvent(new Event('storage-updated'))
}

onMounted(loadData)
</script>

<style scoped>
.home { max-width: 920px; margin: 0 auto; }

.welcome-banner {
  background: linear-gradient(135deg, var(--pine) 0%, var(--jade-deep) 60%, var(--jade) 100%);
  border-radius: var(--radius-xl);
  padding: 32px;
  display: flex;
  justify-content: space-between;
  align-items: stretch;
  gap: 28px;
  color: var(--cream);
  margin-bottom: 24px;
  position: relative;
  overflow: hidden;
  box-shadow: 0 8px 32px rgba(26, 26, 26, 0.18);
}
.welcome-banner::before {
  content: '';
  position: absolute;
  top: -30%;
  right: -10%;
  width: 50%;
  height: 160%;
  background: radial-gradient(ellipse, rgba(184, 149, 106, 0.18), transparent 70%);
  pointer-events: none;
}

.banner-left { flex: 1; position: relative; z-index: 1; display: flex; flex-direction: column; }
.banner-tag { font-family: var(--font-mono); font-size: 12px; letter-spacing: 3px; opacity: 0.55; }
.banner-title { font-size: 26px; font-weight: 700; margin: 12px 0 10px; letter-spacing: 1px; }
.banner-desc { font-size: 14px; opacity: 0.72; margin-bottom: 24px; line-height: 1.7; }
.banner-actions { display: flex; gap: 12px; flex-wrap: wrap; margin-top: auto; }
.banner-actions .btn-primary {
  background: var(--gold);
  color: var(--white);
  border-radius: var(--radius-md);
  padding: 11px 22px;
  font-weight: 600;
  box-shadow: 0 2px 12px rgba(184, 149, 106, 0.3);
}
.banner-actions .btn-primary:hover { background: var(--gold-soft); transform: translateY(-1px); }
.banner-actions .btn-outline {
  background: rgba(245, 242, 236, 0.1);
  color: var(--cream);
  border: 1px solid rgba(245, 242, 236, 0.22);
  border-radius: var(--radius-md);
  padding: 11px 22px;
  font-weight: 500;
  backdrop-filter: blur(8px);
}
.banner-actions .btn-outline:hover { background: rgba(245, 242, 236, 0.18); border-color: var(--gold-soft); }
.banner-actions .btn-ghost {
  background: transparent;
  color: rgba(245, 242, 236, 0.78);
  border-radius: var(--radius-md);
  padding: 11px 18px;
  font-weight: 500;
}
.banner-actions .btn-ghost:hover { background: rgba(245, 242, 236, 0.08); color: var(--cream); }

.banner-right { width: 380px; flex-shrink: 0; position: relative; z-index: 1; display: flex; align-items: center; }
.banner-right :deep(.stats-chart) { background: rgba(245, 242, 236, 0.06); border-color: rgba(245, 242, 236, 0.12); }
.banner-right :deep(.chart-title) { color: var(--cream); }
.banner-right :deep(.chart-sub) { color: rgba(245, 242, 236, 0.5); }
.banner-right :deep(.donut-track) { stroke: rgba(245, 242, 236, 0.12); }
.banner-right :deep(.donut-num) { fill: var(--cream); }
.banner-right :deep(.donut-label) { fill: rgba(245, 242, 236, 0.55); }
.banner-right :deep(.legend-label) { color: rgba(245, 242, 236, 0.7); }
.banner-right :deep(.legend-value) { color: var(--cream); }
.banner-right :deep(.legend-dot.picked) { background: var(--gold-soft); }
.banner-right :deep(.legend-dot.total) { background: rgba(245, 242, 236, 0.3); }
.banner-right :deep(.trend-title) { color: var(--cream); }
.banner-right :deep(.trend-avg) { color: rgba(245, 242, 236, 0.5); }
.banner-right :deep(.grid-line) { stroke: rgba(245, 242, 236, 0.1); }
.banner-right :deep(.bar) { fill: rgba(245, 242, 236, 0.4); }
.banner-right :deep(.bar.active) { fill: var(--gold-soft); }
.banner-right :deep(.trend-labels span) { color: rgba(245, 242, 236, 0.5); }
.banner-right :deep(.chart-body) { border-bottom-color: rgba(245, 242, 236, 0.1); }

.quick-actions {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 12px;
  margin-bottom: 28px;
}
.action-card {
  background: var(--white);
  border: 1px solid var(--border-soft);
  border-radius: var(--radius-md);
  padding: 16px 18px;
  display: flex;
  align-items: center;
  gap: 14px;
  cursor: pointer;
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: var(--shadow-sm);
}
.action-card:hover {
  transform: translateY(-2px);
  box-shadow: var(--shadow-md);
  border-color: var(--gold-soft);
}
.action-icon {
  width: 40px;
  height: 40px;
  border-radius: 10px;
  background: var(--jade-light);
  color: var(--ink);
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}
.action-card:hover .action-icon { background: var(--gold); color: var(--white); }
.action-text { flex: 1; display: flex; flex-direction: column; gap: 2px; min-width: 0; }
.action-label { font-size: 14px; font-weight: 600; color: var(--ink); }
.action-desc { font-size: 12px; color: var(--ink-hint); }
.action-arrow { color: var(--ink-hint); flex-shrink: 0; transition: transform 0.25s ease; }
.action-card:hover .action-arrow { transform: translateX(3px); color: var(--gold); }

.section { margin-bottom: 28px; }
.section-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 16px;
  padding: 0 4px;
}
.section-header.toggle { cursor: pointer; user-select: none; }
.section-title { font-size: 18px; font-weight: 600; color: var(--ink); }
.section-count { font-size: 13px; color: var(--ink-hint); font-family: var(--font-mono); }
.toggle-right { display: flex; align-items: center; gap: 8px; }
.toggle-arrow { color: var(--ink-hint); transition: transform 0.3s ease; }
.toggle-arrow.open { transform: rotate(180deg); }

.card-list { display: flex; flex-direction: column; gap: 12px; }

.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 60px 0;
  gap: 8px;
}
.empty-icon { margin-bottom: 8px; }
.empty-title { font-size: 18px; font-weight: 600; color: var(--ink); }
.empty-desc { font-size: 14px; color: var(--ink-hint); margin-bottom: 20px; }

.insights {
  display: grid;
  grid-template-columns: 1.3fr 1fr;
  gap: 16px;
  margin-bottom: 28px;
}
.insight-card {
  background: var(--white);
  border: 1px solid var(--border-soft);
  border-radius: var(--radius-lg);
  padding: 20px 22px;
  box-shadow: var(--shadow-sm);
}
.insight-head {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  margin-bottom: 16px;
}
.insight-title { font-size: 15px; font-weight: 600; color: var(--ink); }
.insight-sub { font-size: 11px; color: var(--ink-hint); font-family: var(--font-mono); letter-spacing: 1px; }

.station-list { display: flex; flex-direction: column; gap: 12px; }
.station-row {
  display: flex;
  align-items: center;
  gap: 12px;
}
.station-name {
  font-size: 13px;
  color: var(--ink);
  width: 90px;
  flex-shrink: 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.station-bar-wrap {
  flex: 1;
  height: 8px;
  background: var(--jade-light);
  border-radius: 4px;
  overflow: hidden;
}
.station-bar {
  height: 100%;
  background: linear-gradient(90deg, var(--gold-deep), var(--gold-soft));
  border-radius: 4px;
  transition: width 0.6s cubic-bezier(0.4, 0, 0.2, 1);
}
.station-count {
  font-size: 13px;
  font-weight: 600;
  color: var(--ink);
  font-family: var(--font-mono);
  width: 28px;
  text-align: right;
}
.insight-empty { font-size: 13px; color: var(--ink-hint); padding: 12px 0; }

.efficiency-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}
.eff-item {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 4px;
  padding: 12px 14px;
  background: var(--cream-warm);
  border-radius: var(--radius-md);
}
.eff-num {
  font-size: 24px;
  font-weight: 700;
  color: var(--ink);
  font-family: var(--font-mono);
  line-height: 1;
}
.eff-label { font-size: 12px; color: var(--ink-hint); }

.collapse-enter-active, .collapse-leave-active { transition: all 0.3s ease; overflow: hidden; }
.collapse-enter-from, .collapse-leave-to { opacity: 0; max-height: 0; }
.collapse-enter-to, .collapse-leave-from { opacity: 1; max-height: 2000px; }

@media (max-width: 768px) {
  .welcome-banner { flex-direction: column; padding: 24px 20px; gap: 20px; }
  .banner-title { font-size: 22px; }
  .banner-right { width: 100%; }
  .quick-actions { grid-template-columns: repeat(2, 1fr); }
  .insights { grid-template-columns: 1fr; }
}
</style>