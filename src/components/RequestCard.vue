<template>
  <el-card
    class="request-card"
    :class="cardClasses"
    :shadow="isCompleted ? 'never' : 'hover'"
  >
    <div
      class="card-main"
      :class="{ 'is-clickable': isCompleted }"
      @click="handleCardClick"
    >
      <div class="card-header">
        <div class="card-header-top">
          <div class="card-title">
            <h2>{{ request.org }}</h2>
            <div class="tags">
              <el-tag
                v-for="tag in cardTags"
                :key="tag.value"
                size="small"
                :style="{ backgroundColor: tag.color }"
                effect="dark"
              >
                {{ tag.label }}
              </el-tag>
              <el-tag
                size="small"
                :type="requestStatus.type"
                effect="dark"
              >
                {{ requestStatus.label }}
              </el-tag>
            </div>
          </div>
          <div class="published-at">
            <el-icon><Clock /></el-icon>
            <span class="meta-text">發布 {{ formatTimeAgo(request.created_at) }}</span>
          </div>
        </div>
      </div>

      <div v-show="!props.isCompletedCollapsed" class="card-body">
        <div class="card-contact">
          <div class="section-title">聯絡資訊</div>
          <div class="contact-info">
            <el-link
              class="contact-row contact-link contact-link-map"
              :href="mapLink(request.address)"
              target="_blank"
              :underline="true"
              @click.stop
            >
              <el-icon><Location /></el-icon>
              <span class="meta-text">{{ request.address }}</span>
              <el-icon class="contact-link-icon"><TopRight /></el-icon>
            </el-link>
            <el-link
              v-if="displayPhone"
              class="contact-row contact-link contact-link-phone"
              :href="phoneHref(displayPhone)"
              :underline="false"
              @click.stop
            >
              <el-icon><Phone /></el-icon>
              <span class="meta-text">{{ displayPhone }}</span>
            </el-link>
          </div>
        </div>

        <div class="card-content">
          <div class="section-title">需求物資</div>
          <div
            v-for="(item, index) in request.items"
            :key="`${request.id}-${index}`"
            class="item-row"
            :class="{ 'is-fulfilled': isItemFulfilled(item) }"
          >
            <div class="item-info">
              <div class="item-title">
                <span class="item-name">{{ item.name }}</span>
                <el-tag
                  size="small"
                  :style="{ backgroundColor: typeMeta(item.type).color }"
                  effect="dark"
                >
                  {{ typeMeta(item.type).label }}
                </el-tag>
              </div>
              <div class="item-description">
                <span>需求 {{ item.need }}{{ item.unit }}</span>
                <span>
                  已收到 {{ item.got }}/{{ item.need }}{{ item.unit }}，還需要：
                  <strong class="need-number">{{ remainingNeed(item) }}{{ item.unit }}</strong>
                </span>
              </div>
            </div>
            <el-progress
              :percentage="progressPercentage(item)"
              :status="itemProgressStatus(item)"
            />
          </div>
        </div>
      </div>
    </div>

    <template #footer>
      <div
        class="card-footer"
        :class="{ 'is-clickable': isCompleted }"
        @click="isCompleted ? handleCardClick : null"
      >
        <div v-if="!isCompleted" class="card-actions">
          <el-button
            type="primary"
            :icon="Van"
            @click.stop="$emit('delivery', request)"
          >
            我要配送
          </el-button>
        </div>
        <div v-if="isCompleted" class="completed-toggle">
          <el-button
            :class="['completed-toggle-btn', { 'is-expanded': !props.isCompletedCollapsed }]"
            circle
            plain
            size="small"
            @click.stop="handleCardClick"
          >
            <el-icon><ArrowDown /></el-icon>
          </el-button>
        </div>
      </div>
    </template>

    <div v-if="isCompleted" class="card-stamp">已完成</div>
  </el-card>
</template>

<script setup>
import { computed } from 'vue';
import { ArrowDown, Clock, Location, Phone, TopRight, Van } from "@element-plus/icons-vue";

const props = defineProps({
  request: {
    type: Object,
    required: true
  },
  typeMeta: {
    type: Function,
    required: true
  },
  isCompletedCollapsed: {
    type: Boolean,
    default: false
  }
});

const emit = defineEmits(['delivery', 'toggle-collapse']);

// 計算屬性
const isCompleted = computed(() => 
  props.request.items.every((item) => (item.got ?? 0) >= (item.need ?? 0))
);

const cardClasses = computed(() => ({
  "is-completed": isCompleted.value,
  "has-medical": props.request.items.some((item) => item.type === "醫療用品"),
  "is-collapsed": props.isCompletedCollapsed,
}));

const cardTags = computed(() => {
  const metas = [...new Set(props.request.items.map((item) => props.typeMeta(item.type)))];
  return metas
    .sort((a, b) => a.order - b.order)
    .map((meta) => ({
      label: meta.label,
      value: meta.label,
      color: meta.color,
    }));
});

const requestStatus = computed(() => {
  if (isCompleted.value) return { label: "已完成", type: "success" };
  if (props.request.items.some((item) => item.type === "醫療用品" && remainingNeed(item) > 0)) {
    return { label: "緊急", type: "danger" };
  }
  if (props.request.items.some((item) => remainingNeed(item) > 0)) {
    return { label: "尚缺", type: "warning" };
  }
  return { label: "緊急", type: "danger" };
});

const displayPhone = computed(() => {
  if (isCompleted.value) return "";
  return props.request.phone ?? "";
});

// 方法
const remainingNeed = (item) => Math.max(0, (item.need ?? 0) - (item.got ?? 0));

const isItemFulfilled = (item) => remainingNeed(item) === 0;

const progressPercentage = (item) => {
  if (!item.need || item.need <= 0) return 0;
  return Math.round(Math.max(0, Math.min(100, ((item.got ?? 0) / item.need) * 100)));
};

const itemProgressStatus = (item) => {
  if (remainingNeed(item) === 0) return "success";
  return "exception";
};

const formatTimeAgo = (timestamp) => {
  if (!timestamp) return "時間未知";
  const now = Math.floor(Date.now() / 1000);
  const diff = now - timestamp;
  if (diff < 0) return "剛剛";
  if (diff < 60) return "剛剛";
  if (diff < 3600) return `${Math.floor(diff / 60)}分鐘前`;
  const hours = Math.floor(diff / 3600);
  if (hours < 24) return `${hours}小時前`;
  const days = Math.floor(hours / 24);
  const remainingHours = hours % 24;
  if (days < 7) {
    return remainingHours === 0 ? `${days}天前` : `${days}天${remainingHours}小時前`;
  }
  const weeks = Math.floor(days / 7);
  const remainingDays = days % 7;
  if (weeks < 4) {
    return remainingDays === 0 ? `${weeks}週前` : `${weeks}週${remainingDays}天前`;
  }
  const months = Math.floor(days / 30);
  const remainingWeeks = Math.floor((days % 30) / 7);
  if (months < 12) {
    return remainingWeeks === 0 ? `${months}個月前` : `${months}個月${remainingWeeks}週前`;
  }
  const years = Math.floor(days / 365);
  const remainingMonths = Math.floor((days % 365) / 30);
  return remainingMonths === 0 ? `${years}年前` : `${years}年${remainingMonths}個月前`;
};

const mapLink = (address) =>
  `https://www.google.com/maps/search/?api=1&query=${encodeURIComponent(address)}`;

const phoneHref = (phone) => {
  if (!phone) return "";
  const sanitized = `${phone}`.replace(/[^0-9+#*]/g, "");
  return `tel:${sanitized || phone}`;
};

const handleCardClick = () => {
  if (!isCompleted.value) return;
  emit('toggle-collapse', props.request);
};
</script>

<style scoped>
.request-card {
  position: relative;
  overflow: visible;
  border-radius: 18px;
  min-height: 100%;
}

.request-card.has-medical {
  border: 2px solid rgba(239, 68, 68, 0.35);
}

.request-card.is-completed.has-medical {
  border: 2px solid rgba(239, 68, 68, 0.35);
}

.request-card.is-completed {
  background: #e2e8f0;
  border: 1px solid #cbd5f5;
  cursor: default;
}

.card-header {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.card-header-top {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  flex-wrap: wrap;
}

.card-title {
  display: flex;
  align-items: center;
  gap: 8px;
  flex-wrap: wrap;
}

.card-title h2 {
  margin: 0;
  font-size: 1.25rem;
  font-weight: 700;
  color: #111827;
}

.published-at {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 0.85rem;
  color: #6b7280;
}

.tags {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
}

:deep(.el-tag) {
  border: none;
}

.meta-text {
  margin-left: 6px;
}

.card-contact {
  display: flex;
  flex-direction: column;
  gap: 6px;
  margin-top: 4px;
}

.card-body {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.card-main {
  display: flex;
  flex-direction: column;
  background: transparent;
}

.card-main.is-clickable {
  cursor: pointer;
}

.card-footer {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  border-radius: 30px;
}

.request-card.is-completed :deep(.el-card__footer) {
  padding: 0;
}

.card-footer.is-clickable {
  cursor: pointer;
}

.completed-toggle {
  display: flex;
  justify-content: center;
  margin-top: 4px;
}

.completed-toggle-btn {
  padding: 4px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  background-color: transparent;
  border-color: transparent;
  box-shadow: none;
}

.completed-toggle-btn .el-icon {
  font-size: 18px;
  color: #334155;
  transition: transform 0.2s ease, color 0.2s ease;
}

.completed-toggle-btn.is-expanded .el-icon {
  transform: rotate(180deg);
}

@media (hover: hover) {
  .card-main.is-clickable:hover {
    background: #f1f5f9;
  }

  .completed-toggle-btn:hover {
    border-color: transparent;
  }

  .completed-toggle-btn:hover .el-icon,
  .request-card.is-completed:hover .completed-toggle-btn .el-icon {
    color: var(--el-color-primary);
  }
}

.section-title {
  font-size: 0.85rem;
  font-weight: 600;
  color: #1f2937;
  position: relative;
}

.section-title::before {
  content: "";
  display: block;
  width: 100%;
  border-top: 1px dashed #e2e8f0;
  margin: 12px 0;
}

.contact-info {
  display: flex;
  flex-direction: column;
  gap: 6px;
  align-items: flex-start;
  width: 100%;
}

.contact-row {
  display: inline-flex;
  align-items: flex-start;
  gap: 6px;
  font-size: 0.9rem;
  color: #334155;
}

.contact-link {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  justify-content: flex-start;
  text-decoration: none;
}

.contact-link-map {
  text-decoration: underline;
  text-underline-offset: 3px;
  color: var(--el-color-primary);
}

.contact-link-map:hover {
  color: var(--el-color-primary-dark-2);
}

.contact-link-icon {
  font-size: 1rem;
  margin-top: 0 !important;
}

.contact-link-phone {
  color: #0f766e;
}

.contact-link-phone:hover {
  color: #0d9488;
}

.contact-row :deep(.el-icon) {
  line-height: 1;
  margin-top: 2px;
}

.card-content {
  margin-top: 0;
  display: flex;
  flex-direction: column;
  gap: 12px;
  background: transparent;
}

.item-row {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.item-row + .item-row {
  border-top: 1px dashed #e5e7eb;
  padding-top: 12px;
}

.item-row.is-fulfilled {
  background: #f3f4f6;
  border-radius: 12px;
  padding: 12px;
}

.item-row.is-fulfilled + .item-row {
  border-top: none;
}

.item-row.is-fulfilled .item-name {
  color: #4b5563;
}

.item-row.is-fulfilled .item-description,
.item-row.is-fulfilled .need-number {
  color: #6b7280;
}

.item-info {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.item-title {
  display: flex;
  align-items: center;
  gap: 8px;
}

.item-name {
  font-weight: 600;
  font-size: 1rem;
}

.item-description {
  display: flex;
  flex-direction: column;
  font-size: 0.9rem;
  color: #4b5563;
  gap: 4px;
  width: 100%;
}

.need-number {
  color: #ef4444;
}

.card-actions {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  gap: 12px;
}

.card-stamp {
  position: absolute;
  top: 16px;
  right: 12px;
  transform: rotate(10deg);
  padding: 6px 32px;
  font-weight: 700;
  border: 3px solid #22c55e;
  color: #15803d;
  border-radius: 12px;
  background: rgba(34, 197, 94, 0.12);
  font-size: 1rem;
  z-index: 5;
  pointer-events: none;
}

@media (min-width: 768px) {
  .item-description {
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    gap: 8px;
  }

  .item-description span:last-child {
    text-align: right;
    margin-right: 56px;
  }
}

.request-card.is-completed .card-main,
.request-card.is-completed .card-header,
.request-card.is-completed .card-header-top,
.request-card.is-completed .card-body,
.request-card.is-completed .card-content,
.request-card.is-completed .card-footer,
.request-card.is-completed .card-contact,
.request-card.is-completed .item-row,
.request-card.is-completed .card-actions {
  background: #e2e8f0;
}
</style>

