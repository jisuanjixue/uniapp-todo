<template>
  <view class="container">
    <view class="header">
      <text class="title">Todo List</text>
      <view class="add-btn" @click="showAddForm = true">+</view>
    </view>

    <!-- Filter tabs -->
    <view class="filter-tabs">
      <view
        v-for="filter in filters"
        :key="filter.value"
        class="filter-tab"
        :class="{ active: currentFilter === filter.value }"
        @click="currentFilter = filter.value"
      >
        {{ filter.label }}
      </view>
    </view>

    <!-- Add Todo Form -->
    <TodoForm
      v-if="showAddForm"
      @submit="addTodo"
      @cancel="showAddForm = false"
    />

    <!-- Todo List -->
    <view class="todo-list">
      <view v-if="filteredTodos.length === 0" class="empty-state">
        <text>{{ emptyStateMessage }}</text>
      </view>

      <TodoItem
        v-for="todo in filteredTodos"
        :key="todo.id"
        :todo="todo"
        @toggle="toggleTodo"
        @delete="confirmDeleteTodo"
        @edit="navigateToEdit"
        @view="viewTodoDetails"
      />
    </view>

    <!-- Stats Footer -->
    <view class="todo-stats" v-if="todos.length > 0">
      <text>{{ activeCount }} items left</text>
      <text v-if="completedCount > 0" class="clear-completed" @click="clearCompleted">
        Clear completed ({{ completedCount }})
      </text>
    </view>

    <!-- Todo Details Modal -->
    <view class="modal" v-if="showDetailsModal">
      <view class="modal-content">
        <view class="modal-header">
          <text class="modal-title">Task Details</text>
          <text class="modal-close" @click="showDetailsModal = false">×</text>
        </view>
        <view class="modal-body" v-if="selectedTodo">
          <view class="detail-item">
            <text class="detail-label">Task:</text>
            <text class="detail-value">{{ selectedTodo.text }}</text>
          </view>
          <view class="detail-item" v-if="selectedTodo.dueDate">
            <text class="detail-label">Due Date:</text>
            <text class="detail-value">{{ formatDate(selectedTodo.dueDate) }}</text>
          </view>
          <view class="detail-item" v-if="selectedTodo.priority">
            <text class="detail-label">Priority:</text>
            <text class="detail-value">{{ selectedTodo.priority }}</text>
          </view>
          <view class="detail-item" v-if="selectedTodo.notes">
            <text class="detail-label">Notes:</text>
            <text class="detail-value">{{ selectedTodo.notes }}</text>
          </view>
          <view class="detail-item">
            <text class="detail-label">Status:</text>
            <text class="detail-value">{{ selectedTodo.completed ? 'Completed' : 'Active' }}</text>
          </view>
          <view class="detail-item">
            <text class="detail-label">Created:</text>
            <text class="detail-value">{{ formatDateTime(selectedTodo.createdAt) }}</text>
          </view>
        </view>
        <view class="modal-footer">
          <view class="btn btn-primary" @click="navigateToEdit(selectedTodo?.id || '')">Edit</view>
          <view class="btn" @click="showDetailsModal = false">Close</view>
        </view>
      </view>
    </view>
  </view>
</template>

<script setup lang="ts">
import { ref, reactive, computed, onMounted } from 'vue';
import { onShow } from '@dcloudio/uni-app';
import TodoItem from '../../components/TodoItem.vue';
import TodoForm from '../../components/TodoForm.vue';

interface Todo {
  id: string;
  text: string;
  completed: boolean;
  dueDate: string;
  priority: string;
  notes: string;
  createdAt: number;
}

// Filters
const filters = [
  { label: 'All', value: 'all' },
  { label: 'Active', value: 'active' },
  { label: 'Completed', value: 'completed' }
];

const currentFilter = ref('all');
const todos = ref<Todo[]>([]);
const showAddForm = ref(false);
const showDetailsModal = ref(false);
const selectedTodo = ref<Todo | null>(null);

// Load todos from storage
const loadTodos = () => {
  const storedTodos = uni.getStorageSync('todos');
  if (storedTodos) {
    try {
      todos.value = JSON.parse(storedTodos);
    } catch (e) {
      console.error('Error parsing todos:', e);
      todos.value = [];
    }
  }
};

// Save todos to storage
const saveTodos = () => {
  uni.setStorageSync('todos', JSON.stringify(todos.value));
};

// Load todos on mount and when page shows (returning from edit page)
onMounted(loadTodos);
onShow(loadTodos);

// Computed properties
const filteredTodos = computed(() => {
  if (currentFilter.value === 'active') {
    return todos.value.filter(todo => !todo.completed);
  } else if (currentFilter.value === 'completed') {
    return todos.value.filter(todo => todo.completed);
  }
  return todos.value;
});

const activeCount = computed(() => {
  return todos.value.filter(todo => !todo.completed).length;
});

const completedCount = computed(() => {
  return todos.value.filter(todo => todo.completed).length;
});

const emptyStateMessage = computed(() => {
  if (todos.value.length === 0) {
    return 'No todos yet. Add one!';
  }

  if (currentFilter.value === 'active' && activeCount.value === 0) {
    return 'No active todos!';
  }

  if (currentFilter.value === 'completed' && completedCount.value === 0) {
    return 'No completed todos!';
  }

  return 'No todos found';
});

// Generate a unique ID
const generateId = (): string => {
  return Date.now().toString(36) + Math.random().toString(36).substr(2);
};

// Add a new todo
const addTodo = (todoData: Omit<Todo, 'id' | 'createdAt'>) => {
  const newTodo: Todo = {
    ...todoData,
    id: generateId(),
    createdAt: Date.now()
  };

  todos.value.unshift(newTodo);
  saveTodos();
  showAddForm.value = false;

  uni.showToast({
    title: 'Task added!',
    icon: 'success',
    duration: 2000
  });
};

// Toggle todo completion status
const toggleTodo = (id: string) => {
  const index = todos.value.findIndex(todo => todo.id === id);
  if (index !== -1) {
    todos.value[index].completed = !todos.value[index].completed;
    saveTodos();
  }
};

// Confirm delete todo
const confirmDeleteTodo = (id: string) => {
  uni.showModal({
    title: 'Delete Task',
    content: 'Are you sure you want to delete this task?',
    success: (res) => {
      if (res.confirm) {
        deleteTodo(id);
      }
    }
  });
};

// Delete todo
const deleteTodo = (id: string) => {
  todos.value = todos.value.filter(todo => todo.id !== id);
  saveTodos();

  uni.showToast({
    title: 'Task deleted',
    icon: 'success',
    duration: 2000
  });
};

// Clear completed todos
const clearCompleted = () => {
  uni.showModal({
    title: 'Clear Completed',
    content: `Are you sure you want to delete ${completedCount.value} completed tasks?`,
    success: (res) => {
      if (res.confirm) {
        todos.value = todos.value.filter(todo => !todo.completed);
        saveTodos();

        uni.showToast({
          title: 'Completed tasks cleared',
          icon: 'success',
          duration: 2000
        });
      }
    }
  });
};

// Navigate to edit page
const navigateToEdit = (id: string) => {
  console.log('Navigating to edit page with ID:', id);

  // 确保ID不为空
  if (!id) {
    console.error('Cannot navigate to edit page: ID is empty');
    uni.showToast({
      title: 'Error: Invalid task ID',
      icon: 'none'
    });
    return;
  }

  showDetailsModal.value = false;

  // 使用encodeURIComponent确保ID被正确编码
  const url = `/pages/edit/edit?id=${encodeURIComponent(id)}`;
  console.log('Navigation URL:', url);

  uni.navigateTo({
    url,
    success: () => {
      console.log('Navigation successful');
    },
    fail: (err) => {
      console.error('Navigation failed:', err);
    }
  });
};

// View todo details
const viewTodoDetails = (id: string) => {
  const todo = todos.value.find(todo => todo.id === id);
  if (todo) {
    selectedTodo.value = todo;
    showDetailsModal.value = true;
  }
};

// Format date for display
const formatDate = (dateString: string): string => {
  if (!dateString) return '';

  const date = new Date(dateString);
  return date.toLocaleDateString();
};

// Format date and time for display
const formatDateTime = (timestamp: number): string => {
  if (!timestamp) return '';

  const date = new Date(timestamp);
  return date.toLocaleDateString() + ' ' + date.toLocaleTimeString();
};
</script>

<style>
.container {
  padding: 30rpx;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30rpx;
}

.title {
  font-size: 48rpx;
  font-weight: bold;
  color: #333;
}

.add-btn {
  width: 80rpx;
  height: 80rpx;
  background-color: #007aff;
  color: #fff;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 48rpx;
  font-weight: bold;
}

.filter-tabs {
  display: flex;
  margin-bottom: 30rpx;
  background-color: #fff;
  border-radius: 8rpx;
  overflow: hidden;
  box-shadow: 0 2rpx 6rpx rgba(0, 0, 0, 0.1);
}

.filter-tab {
  flex: 1;
  text-align: center;
  padding: 20rpx 0;
  font-size: 28rpx;
  color: #666;
}

.filter-tab.active {
  background-color: #007aff;
  color: #fff;
}

.todo-list {
  margin-bottom: 30rpx;
}

.empty-state {
  text-align: center;
  padding: 60rpx 0;
  color: #999;
  font-size: 32rpx;
}

.todo-stats {
  display: flex;
  justify-content: space-between;
  font-size: 28rpx;
  color: #666;
  padding: 20rpx 0;
}

.clear-completed {
  color: #007aff;
}

/* Modal styles */
.modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 999;
}

.modal-content {
  width: 80%;
  background-color: #fff;
  border-radius: 8rpx;
  overflow: hidden;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 30rpx;
  border-bottom: 1px solid #eee;
}

.modal-title {
  font-size: 36rpx;
  font-weight: bold;
}

.modal-close {
  font-size: 48rpx;
  color: #999;
}

.modal-body {
  padding: 30rpx;
}

.detail-item {
  margin-bottom: 20rpx;
}

.detail-label {
  font-size: 28rpx;
  color: #666;
  margin-bottom: 8rpx;
  display: block;
}

.detail-value {
  font-size: 32rpx;
  color: #333;
}

.modal-footer {
  padding: 20rpx 30rpx;
  display: flex;
  justify-content: flex-end;
  border-top: 1px solid #eee;
}
</style>
