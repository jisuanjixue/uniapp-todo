<template>
  <view class="todo-item" :class="{ 'todo-completed': todo.completed }">
    <view class="todo-checkbox" @click="toggleTodo">
      <text class="checkbox-icon" :class="{ 'checked': todo.completed }">
        {{ todo.completed ? '✓' : '' }}
      </text>
    </view>
    <view class="todo-content" @click="viewDetails">
      <text class="todo-text">{{ todo.text }}</text>
      <text v-if="todo.dueDate" class="todo-due-date">Due: {{ formatDate(todo.dueDate) }}</text>
    </view>
    <view class="todo-actions">
      <text class="todo-edit" @click="editTodo">✎</text>
      <text class="todo-delete" @click="deleteTodo">×</text>
    </view>
  </view>
</template>

<script setup lang="ts">
// No imports needed for defineProps and defineEmits in Vue 3 script setup

interface TodoItem {
  id: string;
  text: string;
  completed: boolean;
  dueDate?: string;
  priority?: string;
  notes?: string;
  createdAt: number;
}

const props = defineProps<{
  todo: TodoItem;
}>();

const emit = defineEmits<{
  (e: 'toggle', id: string): void;
  (e: 'delete', id: string): void;
  (e: 'edit', id: string): void;
  (e: 'view', id: string): void;
}>();

const toggleTodo = () => {
  emit('toggle', props.todo.id);
};

const deleteTodo = () => {
  emit('delete', props.todo.id);
};

const editTodo = () => {
  emit('edit', props.todo.id);
};

const viewDetails = () => {
  emit('view', props.todo.id);
};

const formatDate = (dateString: string): string => {
  if (!dateString) return '';

  const date = new Date(dateString);
  return date.toLocaleDateString();
};
</script>

<style>
.todo-item {
  display: flex;
  padding: 24rpx;
  background-color: #ffffff;
  border-radius: 8rpx;
  margin-bottom: 20rpx;
  box-shadow: 0 2rpx 6rpx rgba(0, 0, 0, 0.1);
  align-items: center;
}

.todo-completed {
  opacity: 0.7;
}

.todo-completed .todo-text {
  text-decoration: line-through;
  color: #999;
}

.todo-checkbox {
  width: 40rpx;
  height: 40rpx;
  margin-right: 20rpx;
}

.checkbox-icon {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 40rpx;
  height: 40rpx;
  border: 2rpx solid #007aff;
  border-radius: 50%;
  color: #ffffff;
  font-size: 24rpx;
}

.checkbox-icon.checked {
  background-color: #007aff;
}

.todo-content {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.todo-text {
  font-size: 32rpx;
  margin-bottom: 8rpx;
}

.todo-due-date {
  font-size: 24rpx;
  color: #666;
}

.todo-actions {
  display: flex;
  align-items: center;
}

.todo-edit, .todo-delete {
  padding: 10rpx;
  font-size: 40rpx;
  color: #666;
  margin-left: 20rpx;
}

.todo-delete {
  color: #ff3b30;
}

.todo-edit {
  color: #007aff;
}
</style>
