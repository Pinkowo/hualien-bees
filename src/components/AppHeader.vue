<template>
  <header class="page-header">
    <div class="header-left">
      <h1>我要配送</h1>
    </div>
    <div class="header-right">
      <el-button
        class="create-btn"
        type="primary"
        :icon="Plus"
        @click="$emit('create')"
      >
        新增物資需求
      </el-button>
    </div>
  </header>
</template>

<script setup>
import { Plus } from "@element-plus/icons-vue";

defineEmits(['create']);
</script>

<style scoped>
.page-header {
  display: flex;
  justify-content: space-around;
  align-items: center;
  flex-wrap: wrap;
  gap: 12px;
  padding: 24px;
  background: #1e293b;
  color: #fff;
  border-radius: 0 0 24px 24px;
  box-shadow: 0 12px 32px rgba(15, 23, 42, 0.12);
}

.header-left {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.header-right {
  display: flex;
  align-items: center;
  gap: 12px;
}

.page-header h1 {
  margin: 0;
  font-size: 1.75rem;
}

@media (max-width: 640px) {
  .page-header {
    padding: 16px;
    flex-direction: column;
    align-items: stretch;
    gap: 12px;
  }

  .header-right {
    width: 100%;
    flex-direction: column;
    align-items: stretch;
    gap: 8px;
  }

  .header-right .el-button {
    width: 100%;
  }
}
</style>

