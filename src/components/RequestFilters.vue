<template>
  <div class="list-controls">
    <el-select
      :model-value="selectedTag"
      class="tag-filter"
      clearable
      placeholder="篩選類別"
      size="large"
      @update:model-value="$emit('update:selectedTag', $event)"
    >
      <el-option
        v-for="option in tagFilterOptions"
        :key="option.value"
        :label="option.label"
        :value="option.value"
      />
    </el-select>

    <el-checkbox 
      :model-value="showPendingOnly"
      class="pending-checkbox"
      @update:model-value="$emit('update:showPendingOnly', $event)"
    >
      只顯示待配送物資
    </el-checkbox>
  </div>
</template>

<script setup>
import { computed } from 'vue';

const props = defineProps({
  selectedTag: {
    type: String,
    default: ''
  },
  showPendingOnly: {
    type: Boolean,
    default: false
  },
  typeOptions: {
    type: Array,
    default: () => []
  }
});

const emit = defineEmits(['update:selectedTag', 'update:showPendingOnly']);

const tagFilterOptions = computed(() => [
  { value: "", label: "全部類別" },
  ...props.typeOptions,
]);
</script>

<style scoped>
.list-controls {
  display: flex;
  align-items: center;
  justify-content: flex-start;
  gap: 48px;
  max-width: 720px;
  margin: 0 auto 16px;
}

.list-controls .tag-filter {
  min-width: 200px;
}

.pending-checkbox {
  margin: 0;
  white-space: nowrap;
}

@media (max-width: 640px) {
  .list-controls {
    flex-direction: column;
    align-items: stretch;
    gap: 8px;
    padding: 0 32px;
    box-sizing: border-box;
  }

  .list-controls .tag-filter {
    width: 100%;
  }

  .pending-checkbox {
    align-self: flex-start;
  }
}
</style>

