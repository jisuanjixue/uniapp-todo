<template>
  <view class="container">
    <view v-if="!todoFound" class="not-found">
      <text>Todo not found!</text>
      <view class="btn btn-primary" @click="goBack">Go Back</view>
    </view>
    
    <view v-else>
      <TodoForm 
        :todo="currentTodo" 
        :isEdit="true"
        @submit="updateTodo" 
        @cancel="goBack"
      />
    </view>
  </view>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue';
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

const todoId = ref('');
const todoFound = ref(true);
const currentTodo = reactive<Todo>({
  id: '',
  text: '',
  completed: false,
  dueDate: '',
  priority: 'Medium',
  notes: '',
  createdAt: 0
});

// Get todo ID from URL parameters
onMounted(() => {
  const query = uni.getEnterOptionsSync().query;
  if (query && query.id) {
    todoId.value = query.id as string;
    loadTodo();
  } else {
    todoFound.value = false;
  }
});

// Load todo from storage
const loadTodo = () => {
  const storedTodos = uni.getStorageSync('todos');
  if (storedTodos) {
    try {
      const todos: Todo[] = JSON.parse(storedTodos);
      const todo = todos.find(t => t.id === todoId.value);
      
      if (todo) {
        Object.assign(currentTodo, todo);
      } else {
        todoFound.value = false;
      }
    } catch (e) {
      console.error('Error parsing todos:', e);
      todoFound.value = false;
    }
  } else {
    todoFound.value = false;
  }
};

// Update todo
const updateTodo = (todoData: Todo) => {
  const storedTodos = uni.getStorageSync('todos');
  if (storedTodos) {
    try {
      const todos: Todo[] = JSON.parse(storedTodos);
      const index = todos.findIndex(t => t.id === todoId.value);
      
      if (index !== -1) {
        // Preserve the original ID and creation date
        todoData.id = todoId.value;
        todoData.createdAt = currentTodo.createdAt;
        
        // Update the todo
        todos[index] = todoData;
        
        // Save back to storage
        uni.setStorageSync('todos', JSON.stringify(todos));
        
        uni.showToast({
          title: 'Task updated!',
          icon: 'success',
          duration: 2000
        });
        
        // Go back to the list page
        setTimeout(() => {
          goBack();
        }, 1500);
      }
    } catch (e) {
      console.error('Error updating todo:', e);
      showError();
    }
  }
};

// Go back to the list page
const goBack = () => {
  uni.navigateBack();
};

// Show error toast
const showError = () => {
  uni.showToast({
    title: 'An error occurred',
    icon: 'error',
    duration: 2000
  });
};
</script>

<style>
.container {
  padding: 30rpx;
}

.not-found {
  text-align: center;
  padding: 100rpx 0;
}

.not-found text {
  font-size: 36rpx;
  color: #999;
  margin-bottom: 40rpx;
  display: block;
}
</style>
