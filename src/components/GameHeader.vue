<template>
  <header class="game-header">
    <div class="header-left">
      <button class="btn ghost" @click="$emit('back')">← 存档</button>
      <h2>第 {{ state.day }} 天</h2>
      <span class="days-left">剩余 {{ daysLeft }} 天</span>
    </div>
    <div class="header-stats">
      <div class="stat-item">
        <span class="label">资金</span>
        <span
          class="value"
          :class="{ danger: budgetLevel === 'danger', warn: budgetLevel === 'warn' }"
          :title="budgetTooltip"
        >
          ¥{{ state.money.toLocaleString() }}
        </span>
        <span v-if="estimatedBurn > 0 && state.money > 0" class="sub-label">
          约可维持 {{ burnDays }} 天
        </span>
      </div>
      <div class="stat-item">
        <span class="label">粉丝</span>
        <span class="value">{{ state.fans.toLocaleString() }}</span>
      </div>
      <div class="stat-item">
        <span class="label">盈利</span>
        <span class="value" :class="profit >= 0 ? 'success' : 'danger'">
          ¥{{ profit.toLocaleString() }}
        </span>
      </div>
      <div class="stat-item">
        <span class="label">出道</span>
        <span class="value">{{ state.groups.length }}/{{ targetGroups }}</span>
      </div>
    </div>
    <button class="theme-btn" @click="$emit('toggle-theme')">
      {{ theme === 'light' ? '🌙' : '☀️' }}
    </button>
  </header>
</template>

<script setup>
import { computed } from 'vue'
import { GAME_CONFIG } from '../config/gameConfig'

const props = defineProps({
  state: Object,
  daysLeft: Number,
  profit: Number,
  theme: String,
  estimatedBurn: Number,
})
defineEmits(['back', 'toggle-theme'])

const targetGroups = GAME_CONFIG.victory.targetGroups

const budgetLevel = computed(() => {
  if (!props.estimatedBurn || props.estimatedBurn <= 0) return 'safe'
  const ratio = props.estimatedBurn / (props.state?.money || 1)
  if (ratio >= GAME_CONFIG.budgetWarning.dangerRatio || props.state?.money < 10000) return 'danger'
  if (ratio >= GAME_CONFIG.budgetWarning.warnRatio || props.state?.money < 30000) return 'warn'
  return 'safe'
})

const budgetTooltip = computed(() => {
  if (!props.estimatedBurn) return ''
  return `日均支出约 ¥${props.estimatedBurn.toLocaleString()}`
})

const burnDays = computed(() => {
  if (!props.estimatedBurn || props.estimatedBurn <= 0 || props.state?.money <= 0) return 0
  return Math.floor(props.state.money / props.estimatedBurn)
})
</script>

<style scoped>
.game-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  padding: 1rem 1.25rem;
  background: var(--bg-card);
  border-bottom: 1px solid var(--border);
  flex-wrap: wrap;
}

.header-left {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.header-left h2 {
  font-size: 1.25rem;
}

.days-left {
  font-size: 0.85rem;
  color: var(--text-muted);
}

.header-stats {
  display: flex;
  gap: 1.5rem;
  flex-wrap: wrap;
}

.stat-item {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.stat-item .label {
  font-size: 0.75rem;
  color: var(--text-muted);
}

.stat-item .value {
  font-weight: 700;
  font-size: 1rem;
}

.stat-item .sub-label {
  font-size: 0.7rem;
  color: var(--text-muted);
  margin-top: 0.1rem;
}

.stat-item .value.success { color: var(--success); }
.stat-item .value.danger { color: var(--danger); }
.stat-item .value.warn { color: var(--warning); }

.theme-btn {
  background: var(--bg-secondary);
  border: 1px solid var(--border);
  border-radius: 50%;
  width: 40px;
  height: 40px;
  cursor: pointer;
  font-size: 1.1rem;
}
</style>
