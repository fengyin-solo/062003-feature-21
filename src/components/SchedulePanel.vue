<template>
  <div class="schedule-panel card">
    <h3>📅 今日日程安排</h3>
    <p class="hint">为每位练习生选择活动，同活动一起训练可提升默契</p>

    <div class="templates-section">
      <span class="section-label">快速模板：</span>
      <div class="template-btns">
        <button
          v-for="(tpl, key) in templates"
          :key="key"
          class="template-btn"
          :title="tpl.description"
          @click="$emit('apply-template', key)"
        >
          <span class="tpl-icon">{{ tpl.icon }}</span>
          <span class="tpl-label">{{ tpl.label }}</span>
        </button>
      </div>
    </div>

    <div v-if="budgetStatus" class="budget-bar" :class="budgetStatus.level">
      <div class="budget-info">
        <span class="budget-label">💰 今日预算</span>
        <span class="budget-detail">
          训练 ¥{{ budgetStatus.scheduleCost.toLocaleString() }}
          + 运营 ¥{{ budgetStatus.operatingCost.toLocaleString() }}
          = <strong>¥{{ budgetStatus.totalCost.toLocaleString() }}</strong>
        </span>
      </div>
      <div class="budget-remain">
        <span v-if="budgetStatus.level === 'safe'" class="remain-safe">
          剩余资金 ¥{{ budgetStatus.moneyAfter.toLocaleString() }}
        </span>
        <span v-else-if="budgetStatus.level === 'warn'" class="remain-warn">
          ⚠️ {{ budgetStatus.message }}
        </span>
        <span v-else class="remain-danger">
          🚨 {{ budgetStatus.message }}
        </span>
      </div>
    </div>

    <div class="schedule-list">
      <div
        v-for="trainee in schedulable"
        :key="trainee.id"
        class="schedule-row"
      >
        <span class="name">{{ trainee.name }}</span>
        <span v-if="trainee.illnessDays > 0" class="ill-tag">休养中</span>
        <div v-else class="activity-btns">
          <button
            v-for="(act, key) in activities"
            :key="key"
            class="act-btn"
            :class="{ active: schedule[trainee.id] === key }"
            :title="`${act.label} ¥${act.moneyCost}`"
            @click="$emit('set', trainee.id, key)"
          >
            {{ act.icon }}
          </button>
        </div>
        <span v-if="schedule[trainee.id]" class="chosen">
          {{ activities[schedule[trainee.id]]?.label }}
        </span>
      </div>
    </div>

    <div class="legend">
      <span v-for="(act, key) in activities" :key="key" class="legend-item">
        {{ act.icon }} {{ act.label }} ¥{{ act.moneyCost }}
      </span>
    </div>

    <div class="actions">
      <button class="btn ghost" @click="$emit('clear')">清空</button>
      <button class="btn primary" :disabled="!canEnd" @click="$emit('end-day')">
        结束今日 →
      </button>
    </div>
    <p v-if="!canEnd" class="warn">请为所有可安排的练习生选择日程</p>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { GAME_CONFIG } from '../config/gameConfig'

const props = defineProps({
  trainees: Array,
  schedule: Object,
  canEnd: Boolean,
  budgetStatus: Object,
})

defineEmits(['set', 'clear', 'end-day', 'apply-template'])

const activities = GAME_CONFIG.activities
const templates = GAME_CONFIG.templates

const schedulable = computed(() =>
  props.trainees.filter((t) => t.status !== 'left')
)
</script>

<style scoped>
.schedule-panel h3 { margin-bottom: 0.25rem; }

.hint {
  font-size: 0.85rem;
  color: var(--text-muted);
  margin-bottom: 0.75rem;
}

.templates-section {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  flex-wrap: wrap;
  margin-bottom: 0.75rem;
  padding-bottom: 0.75rem;
  border-bottom: 1px solid var(--border);
}

.section-label {
  font-size: 0.85rem;
  color: var(--text-muted);
  flex-shrink: 0;
}

.template-btns {
  display: flex;
  gap: 0.4rem;
  flex-wrap: wrap;
}

.template-btn {
  display: flex;
  align-items: center;
  gap: 0.25rem;
  padding: 0.3rem 0.6rem;
  border: 1px solid var(--border);
  border-radius: 6px;
  background: var(--bg-secondary);
  cursor: pointer;
  font-size: 0.8rem;
  transition: all 0.15s;
}

.template-btn:hover {
  border-color: var(--accent);
  background: var(--accent-soft);
}

.tpl-icon { font-size: 0.95rem; }
.tpl-label { white-space: nowrap; }

.budget-bar {
  padding: 0.6rem 0.75rem;
  border-radius: 8px;
  margin-bottom: 0.75rem;
  border: 1px solid var(--border);
}

.budget-bar.safe {
  background: var(--success-soft);
  border-color: var(--success);
}

.budget-bar.warn {
  background: #fef9e7;
  border-color: var(--warning);
}

.budget-bar.danger {
  background: var(--danger-soft);
  border-color: var(--danger);
}

[data-theme='dark'] .budget-bar.warn {
  background: #2d2410;
}

.budget-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin-bottom: 0.25rem;
}

.budget-label {
  font-weight: 600;
  font-size: 0.85rem;
}

.budget-detail {
  font-size: 0.8rem;
  color: var(--text-secondary);
}

.budget-detail strong {
  color: var(--text-primary);
}

.budget-remain {
  font-size: 0.8rem;
  font-weight: 500;
}

.remain-safe { color: var(--success); }
.remain-warn { color: var(--warning); }
.remain-danger { color: var(--danger); }

.schedule-list {
  display: flex;
  flex-direction: column;
  gap: 0.6rem;
  margin-bottom: 1rem;
}

.schedule-row {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  flex-wrap: wrap;
}

.name {
  width: 72px;
  font-weight: 600;
  font-size: 0.9rem;
}

.activity-btns {
  display: flex;
  gap: 0.35rem;
}

.act-btn {
  width: 36px;
  height: 36px;
  border: 1px solid var(--border);
  border-radius: 8px;
  background: var(--bg-secondary);
  cursor: pointer;
  font-size: 1rem;
  transition: all 0.15s;
}

.act-btn:hover { border-color: var(--accent); }
.act-btn.active {
  background: var(--accent-soft);
  border-color: var(--accent);
  transform: scale(1.1);
}

.chosen {
  font-size: 0.8rem;
  color: var(--accent);
}

.ill-tag {
  font-size: 0.8rem;
  color: var(--warning);
}

.legend {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem 1rem;
  font-size: 0.75rem;
  color: var(--text-muted);
  margin-bottom: 1rem;
}

.actions {
  display: flex;
  gap: 0.75rem;
  justify-content: flex-end;
}

.warn {
  text-align: right;
  font-size: 0.8rem;
  color: var(--warning);
  margin-top: 0.5rem;
}
</style>
