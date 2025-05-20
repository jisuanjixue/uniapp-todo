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
import { onLoad } from '@dcloudio/uni-app';
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

// 使用onLoad生命周期钩子获取页面参数
onLoad((options) => {
  console.log('Page options:', options);

  if (options && options.id) {
    todoId.value = options.id;
    loadTodo();
  } else {
    todoFound.value = false;
  }
});

// 保留onMounted以防需要其他初始化
onMounted(() => {
  console.log('Edit page mounted');
});

// Load todo from storage
const loadTodo = () => {
  console.log('Loading todo with ID:', todoId.value);

  const storedTodos = uni.getStorageSync('todos');
  console.log('Stored todos:', storedTodos);

  if (storedTodos) {
    try {
      const todos: Todo[] = JSON.parse(storedTodos);
      console.log('Parsed todos:', todos);

      // 打印所有todo的ID以便比较
      console.log('Todo IDs in storage:', todos.map(t => t.id));

      const todo = todos.find(t => t.id === todoId.value);
      console.log('Found todo:', todo);

      if (todo) {
        Object.assign(currentTodo, todo);
        console.log('Current todo after assignment:', currentTodo);
      } else {
        console.log('Todo not found with ID:', todoId.value);
        todoFound.value = false;
      }
    } catch (e) {
      console.error('Error parsing todos:', e);
      todoFound.value = false;
    }
  } else {
    console.log('No todos found in storage');
    todoFound.value = false;
  }
};

// Update todo
const updateTodo = (todoData: Todo) => {
  console.log('Updating todo with data:', todoData);
  console.log('Current todoId:', todoId.value);

  const storedTodos = uni.getStorageSync('todos');
  if (storedTodos) {
    try {
      const todos: Todo[] = JSON.parse(storedTodos);
      console.log('All todos before update:', todos);

      // 打印所有todo的ID以便比较
      console.log('Todo IDs in storage:', todos.map(t => t.id));

      const index = todos.findIndex(t => t.id === todoId.value);
      console.log('Found todo at index:', index);

      if (index !== -1) {
        // Preserve the original ID and creation date
        todoData.id = todoId.value;
        todoData.createdAt = currentTodo.createdAt;

        console.log('Updated todo data:', todoData);

        // Update the todo
        todos[index] = todoData;

        // Save back to storage
        uni.setStorageSync('todos', JSON.stringify(todos));
        console.log('Todos saved to storage');

        uni.showToast({
          title: 'Task updated!',
          icon: 'success',
          duration: 2000
        });

        // Go back to the list page
        setTimeout(() => {
          goBack();
        }, 1500);
      } else {
        console.error('Todo not found with ID:', todoId.value);
        showError();
      }
    } catch (e) {
      console.error('Error updating todo:', e);
      showError();
    }
  } else {
    console.error('No todos found in storage');
    showError();
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
