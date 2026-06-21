<template>
  <div class="modal-overlay" @click.self="$emit('close')">
    <div class="modal card">
      <h3>📝 第 {{ review.day }} 天复盘</h3>
      <p class="desc">今日执行结果总览</p>

      <div class="summary-grid">
        <div class="summary-item">
          <span class="sum-label">资金变动</span>
          <span class="sum-value" :class="review.moneyChange >= 0 ? 'pos' : 'neg'">
            {{ review.moneyChange >= 0 ? '+' : '' }}¥{{ review.moneyChange.toLocaleString() }}
          </span>
        </div>
        <div class="summary-item">
          <span class="sum-label">粉丝变动</span>
          <span class="sum-value" :class="review.fansChange >= 0 ? 'pos' : 'neg'">
            {{ review.fansChange >= 0 ? '+' : '' }}{{ review.fansChange.toLocaleString() }}
          </span>
        </div>
      </div>

      <div v-if="Object.keys(review.activitiesBreakdown).length > 0" class="section">
        <h4>🎯 活动分布</h4>
        <div class="activity-breakdown">
          <div
            v-for="(count, key) in review.activitiesBreakdown"
            :key="key"
            class="act-row"
          >
            <span class="act-icon">{{ activities[key]?.icon }}</span>
            <span class="act-name">{{ activities[key]?.label }}</span>
            <span class="act-count">{{ count }} 人</span>
          </div>
        </div>
      </div>

      <div v-if="review.highlights.length > 0" class="section">
        <h4>✨ 今日亮点</h4>
        <ul class="highlight-list">
          <li v-for="(h, i) in review.highlights" :key="i">
            <strong>{{ h.trainee }}</strong> 的{{ h.stat }} <span class="gain">+{{ h.value }}</span>
          </li>
        </ul>
      </div>

      <div v-if="review.warnings.length > 0" class="section">
        <h4>⚠️ 需要关注</h4>
        <ul class="warning-list">
          <li v-for="(w, i) in review.warnings" :key="i">
            <strong>{{ w.trainee }}</strong>
            <span v-if="w.type === 'fatigue'" class="warn-tag fatigue">
              疲劳过高 ({{ w.value }})
            </span>
            <span v-else-if="w.type === 'stress'" class="warn-tag stress">
              压力过大 ({{ w.value }})
            </span>
          </li>
        </ul>
      </div>

      <div v-if="review.events && review.events.length > 0" class="section">
        <h4>🎲 今日事件</h4>
        <ul class="event-list">
          <li v-for="(e, i) in review.events" :key="i">{{ e.text }}</li>
        </ul>
      </div>

      <div class="section">
        <h4>👤 练习生详情</h4>
        <div class="trainee-detail-list">
          <div
            v-for="tc in review.traineeChanges"
            :key="tc.id"
            class="trainee-detail"
          >
            <div class="td-header">
              <span class="td-name">{{ tc.name }}</span>
              <span v-if="tc.activity" class="td-activity">
                {{ activities[tc.activity]?.icon }} {{ activities[tc.activity]?.label }}
              </span>
              <span v-if="tc.illnessDays > 0" class="td-ill">休养 {{ tc.illnessDays }}天</span>
            </div>
            <div class="td-stats">
              <span
                v-for="(val, key) in tc.statChanges"
                :key="key"
                class="stat-chip"
                :class="val >= 0 ? 'up' : 'down'"
              >
                {{ statLabels[key] }} {{ val >= 0 ? '+' : '' }}{{ val }}
              </span>
              <span
                v-if="tc.fatigueChange !== 0"
                class="stat-chip"
                :class="tc.fatigueChange <= 0 ? 'up' : 'down'"
              >
                疲劳 {{ tc.fatigueChange >= 0 ? '+' : '' }}{{ tc.fatigueChange }}
              </span>
              <span
                v-if="tc.stressChange !== 0"
                class="stat-chip"
                :class="tc.stressChange <= 0 ? 'up' : 'down'"
              >
                压力 {{ tc.stressChange >= 0 ? '+' : '' }}{{ tc.stressChange }}
              </span>
              <span
                v-if="tc.fansChange !== 0"
                class="stat-chip up"
              >
                粉丝 +{{ tc.fansChange }}
              </span>
            </div>
          </div>
        </div>
      </div>

      <div class="modal-actions">
        <button class="btn primary" @click="$emit('close')">继续 →</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { GAME_CONFIG } from '../config/gameConfig'

defineProps({
  review: Object,
})
defineEmits(['close'])

const activities = GAME_CONFIG.activities
const statLabels = GAME_CONFIG.statLabels
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
  padding: 1rem;
}

.modal {
  max-width: 560px;
  width: 100%;
  max-height: 85vh;
  overflow-y: auto;
  padding: 1.5rem;
}

.desc {
  color: var(--text-secondary);
  font-size: 0.9rem;
  margin-bottom: 1rem;
}

.summary-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0.75rem;
  margin-bottom: 1rem;
}

.summary-item {
  background: var(--bg-secondary);
  border-radius: 8px;
  padding: 0.75rem 1rem;
  display: flex;
  flex-direction: column;
  gap: 0.2rem;
}

.sum-label {
  font-size: 0.8rem;
  color: var(--text-muted);
}

.sum-value {
  font-size: 1.25rem;
  font-weight: 700;
}

.sum-value.pos { color: var(--success); }
.sum-value.neg { color: var(--danger); }

.section {
  margin-bottom: 1rem;
}

.section h4 {
  font-size: 0.95rem;
  margin-bottom: 0.5rem;
}

.activity-breakdown {
  display: flex;
  flex-direction: column;
  gap: 0.35rem;
  background: var(--bg-secondary);
  border-radius: 8px;
  padding: 0.5rem 0.75rem;
}

.act-row {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.85rem;
}

.act-icon { font-size: 1rem; }
.act-name { flex: 1; }
.act-count { color: var(--text-muted); font-weight: 600; }

.highlight-list,
.warning-list,
.event-list {
  list-style: none;
  background: var(--bg-secondary);
  border-radius: 8px;
  padding: 0.5rem 0.75rem;
  font-size: 0.85rem;
}

.highlight-list li,
.warning-list li,
.event-list li {
  padding: 0.25rem 0;
}

.highlight-list .gain {
  color: var(--success);
  font-weight: 600;
}

.warn-tag {
  display: inline-block;
  padding: 0.1rem 0.4rem;
  border-radius: 4px;
  font-size: 0.75rem;
  margin-left: 0.25rem;
}

.warn-tag.fatigue {
  background: var(--warning-soft);
  color: var(--warning);
}

.warn-tag.stress {
  background: var(--danger-soft);
  color: var(--danger);
}

.trainee-detail-list {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.trainee-detail {
  background: var(--bg-secondary);
  border-radius: 8px;
  padding: 0.5rem 0.75rem;
}

.td-header {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 0.35rem;
  flex-wrap: wrap;
}

.td-name {
  font-weight: 600;
  font-size: 0.9rem;
}

.td-activity {
  font-size: 0.75rem;
  color: var(--accent);
  background: var(--accent-soft);
  padding: 0.1rem 0.4rem;
  border-radius: 4px;
}

.td-ill {
  font-size: 0.75rem;
  color: var(--warning);
}

.td-stats {
  display: flex;
  flex-wrap: wrap;
  gap: 0.3rem;
}

.stat-chip {
  font-size: 0.75rem;
  padding: 0.1rem 0.45rem;
  border-radius: 4px;
  background: var(--bg-card);
}

.stat-chip.up { color: var(--success); }
.stat-chip.down { color: var(--danger); }

.modal-actions {
  display: flex;
  gap: 0.75rem;
  justify-content: flex-end;
  margin-top: 0.5rem;
}
</style>
