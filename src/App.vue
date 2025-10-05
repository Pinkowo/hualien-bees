<template>
  <div class="app-shell">
    <AppHeader @create="openCreate" />

    <main class="page-main">
      <RequestFilters
        v-model:selectedTag="selectedTag"
        v-model:showPendingOnly="showPendingOnly"
        :type-options="typeOptions"
      />

      <RequestList
        :requests="visibleRequests"
        :loading="loading"
        :loading-more="loadingMore"
        :has-more="hasMore"
        :total-items="totalItems"
        :type-meta="typeMeta"
        :completed-collapsed="completedCollapsed"
        @delivery="openDelivery"
        @toggle-collapse="toggleCompletedCollapse"
        @load-more="loadMoreRequests"
      />
    </main>

    <footer class="page-footer">
      若您有能力提供物資，請優先以「我要配送」填寫可支援的數量，感謝協助。
    </footer>

    <CreateRequestDialog
      v-model="createDialogVisible"
      :type-options="typeOptions"
      :type-meta="typeMeta"
      @submit="handleCreateSubmit"
    />

    <DeliveryDialog
      v-model="deliveryDialogVisible"
      :target="deliveryTarget"
      :type-meta="typeMeta"
      @submit="handleDeliverySubmit"
      @submit-all="handleDeliverAllSubmit"
    />
  </div>
</template>

<script setup>
import { onMounted, onUnmounted, ref, watch } from "vue";
import AppHeader from './components/AppHeader.vue';
import RequestFilters from './components/RequestFilters.vue';
import RequestList from './components/RequestList.vue';
import CreateRequestDialog from './components/CreateRequestDialog.vue';
import DeliveryDialog from './components/DeliveryDialog.vue';
import { useRequests } from './composables/useRequests.js';
import { useCreateRequest } from './composables/useCreateRequest.js';
import { useDelivery } from './composables/useDelivery.js';
import { useFilters } from './composables/useFilters.js';
import { useCompletedCollapse } from './composables/useCompletedCollapse.js';

// 主要狀態管理
const { requests, loading, loadingMore, hasMore, totalItems, fetchRequests } = useRequests();
const { submitting: createSubmitting, submitCreate } = useCreateRequest();
const { submitting: deliverySubmitting, submitDelivery, submitDeliverAll } = useDelivery();

// 篩選和顯示邏輯
const selectedTag = ref("");
const showPendingOnly = ref(false);
const { typeOptions, typeMeta, isCompleted, visibleRequests } = useFilters(
  requests, 
  selectedTag, 
  showPendingOnly
);

// 完成狀態摺疊管理
const { completedCollapsed, syncCompletedCollapseState, isCompletedCollapsed, toggleCompletedCollapse } = useCompletedCollapse(requests, isCompleted);

// Dialog 狀態
const createDialogVisible = ref(false);
const deliveryDialogVisible = ref(false);
const deliveryTarget = ref(null);

// 事件處理
const openCreate = () => {
  createDialogVisible.value = true;
};

const openDelivery = (req) => {
  deliveryTarget.value = req;
  deliveryDialogVisible.value = true;
};

const handleCreateSubmit = async (payload) => {
  try {
    await submitCreate(payload);
    await fetchRequests();
  } catch (error) {
    console.error('Create error:', error);
  }
};

const handleDeliverySubmit = async (target, summary) => {
  try {
    await submitDelivery(target, summary);
    await fetchRequests();
  } catch (error) {
    console.error('Delivery error:', error);
  }
};

const handleDeliverAllSubmit = async (target, summary) => {
  try {
    await submitDeliverAll(target, summary);
    await fetchRequests();
  } catch (error) {
    console.error('Deliver all error:', error);
  }
};

// 移除這些函數，因為現在使用 v-model 自動處理

const loadMoreRequests = () => {
  if (!hasMore.value || loading.value || loadingMore.value) return;
  fetchRequests({ append: true });
};

// 工具函數
const adjustGoogleSitesHeight = () => {
  const minHeight =
    window.innerWidth <= 480 ? 500 : window.innerWidth <= 768 ? 600 : 800;
  document.documentElement.style.minHeight = `${minHeight}px`;
  document.body.style.minHeight = `${minHeight}px`;
};

// 生命週期
onMounted(() => {
  fetchRequests();
  adjustGoogleSitesHeight();
  window.addEventListener("resize", adjustGoogleSitesHeight);
});

onUnmounted(() => {
  window.removeEventListener("resize", adjustGoogleSitesHeight);
});

// 監聽器
watch(visibleRequests, () => {
  syncCompletedCollapseState();
  requestAnimationFrame(adjustGoogleSitesHeight);
});

watch([selectedTag, showPendingOnly], () => {
  requestAnimationFrame(adjustGoogleSitesHeight);
});
</script>

<style scoped>
html {
  font-size: 20px;
}

.app-shell {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.page-main {
  flex: 1;
  padding: 0 24px 24px;
}

.page-footer {
  padding: 0 24px 32px;
  text-align: center;
  color: #6b7280;
  font-size: 0.95rem;
}

@media (max-width: 640px) {
  .page-main {
    padding: 0;
  }
}
</style>

