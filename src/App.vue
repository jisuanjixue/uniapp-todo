<script setup lang="ts">
import { onLaunch, onShow, onHide } from "@dcloudio/uni-app";

onLaunch(() => {
  console.log("App Launch");
  // Initialize todos storage if it doesn't exist
  const todos = uni.getStorageSync('todos');
  if (!todos) {
    uni.setStorageSync('todos', JSON.stringify([]));
  }
});

onShow(() => {
  console.log("App Show");
});

onHide(() => {
  console.log("App Hide");
});
</script>

<style>
/* Global styles */
page {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen,
    Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  background-color: #f8f8f8;
  font-size: 14px;
  color: #333;
}

.container {
  padding: 20rpx;
}

.btn {
  padding: 20rpx 30rpx;
  border-radius: 8rpx;
  font-size: 28rpx;
  margin: 10rpx;
  text-align: center;
}

.btn-primary {
  background-color: #007aff;
  color: #ffffff;
}

.btn-danger {
  background-color: #ff3b30;
  color: #ffffff;
}

.btn-success {
  background-color: #4cd964;
  color: #ffffff;
}

.input {
  border: 1px solid #ddd;
  padding: 20rpx;
  border-radius: 8rpx;
  margin-bottom: 20rpx;
  background-color: #fff;
}
</style>
