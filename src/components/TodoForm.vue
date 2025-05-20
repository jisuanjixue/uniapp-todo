<template>
  <view class="todo-form">
    <view class="form-header">
      <text class="form-title">{{ isEdit ? 'Edit Todo' : 'Add New Todo' }}</text>
    </view>

    <view class="form-group">
      <input
        class="input todo-input"
        type="text"
        v-model="todoData.text"
        placeholder="What needs to be done?"
        maxlength="100"
        @keyup.enter="submitForm"
      />
    </view>

    <view class="form-group">
      <text class="form-label">Due Date</text>
      <picker
        mode="date"
        :value="todoData.dueDate"
        @change="onDateChange"
        class="date-picker"
      >
        <view class="picker-value">
          {{ todoData.dueDate ? formatDate(todoData.dueDate) : 'Select a date' }}
        </view>
      </picker>
    </view>

    <view class="form-group">
      <text class="form-label">Priority</text>
      <picker
        mode="selector"
        :range="priorityOptions"
        :value="priorityIndex"
        @change="onPriorityChange"
        class="priority-picker"
      >
        <view class="picker-value">
          {{ todoData.priority || 'Select priority' }}
        </view>
      </picker>
    </view>

    <view class="form-group">
      <text class="form-label">Notes</text>
      <textarea
        class="input todo-notes"
        v-model="todoData.notes"
        placeholder="Add notes (optional)"
        maxlength="500"
      />
    </view>

    <view class="form-actions">
      <view class="btn btn-primary" @click="submitForm">
        {{ isEdit ? 'Update' : 'Add' }}
      </view>
      <view class="btn" @click="cancelForm">Cancel</view>
    </view>
  </view>
</template>

<script setup lang="ts">
import { ref, reactive, watch } from 'vue';

interface TodoFormData {
  id?: string;
  text: string;
  completed: boolean;
  dueDate: string;
  priority: string;
  notes: string;
}

const props = defineProps<{
  todo?: TodoFormData;
  isEdit?: boolean;
}>();

const emit = defineEmits<{
  (e: 'submit', todo: TodoFormData): void;
  (e: 'cancel'): void;
}>();

const priorityOptions = ['Low', 'Medium', 'High'];
const priorityIndex = ref(1); // Default to Medium

// Initialize form data
const todoData = reactive<TodoFormData>({
  id: props.todo?.id || '',
  text: props.todo?.text || '',
  completed: props.todo?.completed || false,
  dueDate: props.todo?.dueDate || '',
  priority: props.todo?.priority || 'Medium',
  notes: props.todo?.notes || ''
});

// Update priority index when priority changes
watch(() => todoData.priority, (newPriority) => {
  priorityIndex.value = priorityOptions.indexOf(newPriority);
  if (priorityIndex.value === -1) priorityIndex.value = 1; // Default to Medium
}, { immediate: true });

// Handle date picker change
const onDateChange = (e: any) => {
  todoData.dueDate = e.detail.value;
};

// Handle priority picker change
const onPriorityChange = (e: any) => {
  const index = e.detail.value;
  todoData.priority = priorityOptions[index];
};

// Format date for display
const formatDate = (dateString: string): string => {
  if (!dateString) return '';

  const date = new Date(dateString);
  return date.toLocaleDateString();
};

// Submit form
const submitForm = () => {
  if (!todoData.text.trim()) {
    uni.showToast({
      title: 'Please enter a task',
      icon: 'none'
    });
    return;
  }

  emit('submit', { ...todoData });
};

// Cancel form
const cancelForm = () => {
  emit('cancel');
};
</script>

<style>
.todo-form {
  background-color: #ffffff;
  border-radius: 8rpx;
  padding: 30rpx;
  margin-bottom: 20rpx;
  box-shadow: 0 2rpx 6rpx rgba(0, 0, 0, 0.1);
}

.form-header {
  margin-bottom: 30rpx;
}

.form-title {
  font-size: 36rpx;
  font-weight: bold;
  color: #333;
}

.form-group {
  margin-bottom: 20rpx;
}

.form-label {
  display: block;
  margin-bottom: 10rpx;
  font-size: 28rpx;
  color: #666;
}

.todo-input {
  font-size: 32rpx;
}

.todo-notes {
  width: 100%;
  height: 200rpx;
  font-size: 28rpx;
}

.date-picker, .priority-picker {
  background-color: #f8f8f8;
  padding: 20rpx;
  border-radius: 8rpx;
  border: 1px solid #ddd;
}

.picker-value {
  font-size: 28rpx;
  color: #333;
}

.form-actions {
  display: flex;
  justify-content: flex-end;
  margin-top: 30rpx;
}
</style>
