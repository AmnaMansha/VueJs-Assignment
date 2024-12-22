<script setup>
import { ref } from 'vue';

const emit = defineEmits(['filter-change']);

const statuses = [
  { id: 'all', label: 'All' },
  { id: 'unimportant', label: 'Unimportant' },
  { id: 'serious', label: 'Serious' },
  { id: 'urgent', label: 'Urgent' }
];

const selectedStatus = ref('all');

const selectStatus = (status) => {
  selectedStatus.value = status;
  emit('filter-change', status);
};
</script>

<template>
  <div class="filter-container">
    <div class="filter-options">
      <div
        v-for="status in statuses"
        :key="status.id"
        :class="['filter-option', { active: selectedStatus === status.id }]"
        @click="selectStatus(status.id)"
      >
      <i class="icon" :class="status.id"></i>
        {{ status.label }}
      </div>
     
    </div>
  </div>
</template>

<style scoped>
.filter-container {
  margin-bottom: 20px;
  padding: 10px;
}

.filter-options {
  display: flex;
  gap: 10px;
  justify-content: center;
  background: #f5f5f5;
  padding: 8px;
  border-radius: 12px;
}

.filter-option {
  padding: 8px 16px;
  cursor: pointer;
  border-radius: 8px;
  transition: all 0.3s ease;
  user-select: none;
}

.filter-option:hover {
  background: #e0e0e0;
}

.filter-option.active {
  background: #007bff;
  color: white;
}
.icon {
  margin-right: 8px;
}

.icon.all::before { content: '📑'; }
.icon.unimportant::before { content: '📌'; }
.icon.serious::before { content: '⚠️'; }
.icon.urgent::before { content: '🚨'; }
</style>