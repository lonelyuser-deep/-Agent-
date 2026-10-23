<template>
  <div class="stats-chart">
    <div class="chart-header">
      <span class="chart-title">快递数据总览</span>
      <span class="chart-sub">实时统计</span>
    </div>

    <div class="chart-body">
      <div class="donut-wrap">
        <svg class="donut" viewBox="0 0 120 120">
          <circle class="donut-track" cx="60" cy="60" r="50" />
          <circle
            class="donut-progress"
            cx="60" cy="60" r="50"
            :stroke-dasharray="circumference"
            :stroke-dashoffset="dashOffset"
          />
          <text x="60" y="56" text-anchor="middle" class="donut-num">{{ pickedRate }}%</text>
          <text x="60" y="74" text-anchor="middle" class="donut-label">已取率</text>
        </svg>
      </div>

      <div class="legend">
        <div class="legend-item">
          <span class="legend-dot pending"></span>
          <span class="legend-label">待取件</span>
          <span class="legend-value">{{ pending }}</span>
        </div>
        <div class="legend-item">
          <span class="legend-dot picked"></span>
          <span class="legend-label">已取件</span>
          <span class="legend-value">{{ picked }}</span>
        </div>
        <div class="legend-item">
          <span class="legend-dot total"></span>
          <span class="legend-label">累计</span>
          <span class="legend-value">{{ total }}</span>
        </div>
      </div>
    </div>

    <div class="trend">
      <div class="trend-header">
        <span class="trend-title">近 7 日取件趋势</span>
        <span class="trend-avg">日均 {{ avgPerDay }}</span>
      </div>
      <svg class="bars" :viewBox="`0 0 ${barAreaW} ${barAreaH}`" preserveAspectRatio="none">
        <line
          v-for="g in gridLines"
          :key="'g' + g"
          :x1="0" :x2="barAreaW"
          :y1="g" :y2="g"
          class="grid-line"
        />
        <rect
          v-for="(b, i) in bars"
          :key="'b' + i"
          :x="b.x"
          :y="b.y"
          :width="b.w"
          :height="b.h"
          rx="3"
          class="bar"
          :class="{ active: b.value === maxVal && maxVal > 0 }"
        />
      </svg>
      <div class="trend-labels">
        <span v-for="(d, i) in dayLabels" :key="i">{{ d }}</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'

const props = withDefaults(defineProps<{
  pending: number
  picked: number
  total: number
  trend?: number[]
}>(), {
  trend: () => [0, 0, 0, 0, 0, 0, 0]
})

const circumference = 2 * Math.PI * 50
const pickedRate = computed(() => {
  if (props.total === 0) return 0
  return Math.round((props.picked / props.total) * 100)
})
const dashOffset = computed(() => {
  const rate = props.total === 0 ? 0 : props.picked / props.total
  return circumference * (1 - rate)
})

const dayLabels = computed(() => {
  const labels: string[] = []
  const today = new Date()
  for (let i = 6; i >= 0; i--) {
    const d = new Date(today)
    d.setDate(today.getDate() - i)
    labels.push(`${d.getMonth() + 1}/${d.getDate()}`)
  }
  return labels
})

const barAreaW = 280
const barAreaH = 90
const barCount = 7
const barGap = 14
const maxVal = computed(() => Math.max(1, ...props.trend))
const avgPerDay = computed(() => {
  const sum = props.trend.reduce((a, b) => a + b, 0)
  return (sum / 7).toFixed(1)
})

const bars = computed(() => {
  const totalGap = barGap * (barCount - 1)
  const w = (barAreaW - totalGap) / barCount
  return props.trend.map((v, i) => {
    const h = maxVal.value === 0 ? 0 : (v / maxVal.value) * (barAreaH - 6)
    return {
      x: i * (w + barGap),
      y: barAreaH - h,
      w,
      h,
      value: v,
    }
  })
})

const gridLines = computed(() => {
  return [0, barAreaH / 3, (barAreaH / 3) * 2, barAreaH]
})
</script>

<style scoped>
.stats-chart {
  background: var(--white);
  border-radius: var(--radius-lg);
  border: 1px solid var(--border-soft);
  box-shadow: var(--shadow-sm);
  padding: 20px 22px;
  width: 100%;
}

.chart-header {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  margin-bottom: 18px;
}
.chart-title {
  font-size: 15px;
  font-weight: 600;
  color: var(--ink);
  letter-spacing: 0.5px;
}
.chart-sub {
  font-size: 11px;
  color: var(--ink-hint);
  font-family: var(--font-mono);
  letter-spacing: 1px;
}

.chart-body {
  display: flex;
  align-items: center;
  gap: 24px;
  padding-bottom: 18px;
  border-bottom: 1px dashed var(--border);
  margin-bottom: 18px;
}

.donut-wrap { flex-shrink: 0; }
.donut { width: 120px; height: 120px; }
.donut-track {
  fill: none;
  stroke: var(--border-soft);
  stroke-width: 10;
}
.donut-progress {
  fill: none;
  stroke: var(--gold);
  stroke-width: 10;
  stroke-linecap: round;
  transform: rotate(-90deg);
  transform-origin: 60px 60px;
  transition: stroke-dashoffset 0.8s cubic-bezier(0.4, 0, 0.2, 1);
}
.donut-num {
  font-size: 22px;
  font-weight: 700;
  fill: var(--ink);
  font-family: var(--font-mono);
}
.donut-label {
  font-size: 9px;
  fill: var(--ink-hint);
  letter-spacing: 1px;
}

.legend {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 12px;
}
.legend-item {
  display: flex;
  align-items: center;
  gap: 10px;
}
.legend-dot {
  width: 10px;
  height: 10px;
  border-radius: 3px;
  flex-shrink: 0;
}
.legend-dot.pending { background: var(--ink); }
.legend-dot.picked { background: var(--gold); }
.legend-dot.total { background: var(--border); }
.legend-label {
  font-size: 13px;
  color: var(--ink-soft);
  flex: 1;
}
.legend-value {
  font-size: 18px;
  font-weight: 700;
  color: var(--ink);
  font-family: var(--font-mono);
}

.trend-header {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  margin-bottom: 10px;
}
.trend-title {
  font-size: 13px;
  font-weight: 600;
  color: var(--ink);
}
.trend-avg {
  font-size: 11px;
  color: var(--ink-hint);
  font-family: var(--font-mono);
}

.bars {
  width: 100%;
  height: 90px;
  display: block;
}
.grid-line {
  stroke: var(--border-soft);
  stroke-width: 1;
  stroke-dasharray: 3 3;
}
.bar {
  fill: var(--ink-soft);
  opacity: 0.55;
  transition: all 0.3s ease;
}
.bar.active {
  fill: var(--gold);
  opacity: 1;
}

.trend-labels {
  display: flex;
  justify-content: space-between;
  margin-top: 8px;
  padding: 0 2px;
}
.trend-labels span {
  font-size: 10px;
  color: var(--ink-hint);
  font-family: var(--font-mono);
  width: 40px;
  text-align: center;
}

@media (max-width: 768px) {
  .chart-body {
    flex-direction: column;
    gap: 18px;
  }
  .legend {
    flex-direction: row;
    width: 100%;
    justify-content: space-around;
  }
  .legend-item {
    flex-direction: column;
    gap: 4px;
  }
}
</style>