<template>
  <aside class="sidebar">
    <div class="logo-container">
      <router-link to="/" class="logo-link">
        <span class="logo-icon">V</span>
        <h2 class="logo-text">Vision</h2>
      </router-link>
    </div>

    <div class="history-section">
      <h3 class="history-title">最近任务</h3>
      
      <div v-if="isLoading" class="status-info">正在加载...</div>
      <div v-else-if="error" class="status-info error">{{ error }}</div>
      
      <ul v-else-if="tasks.length > 0" class="item-list">
        <li v-for="task in tasks" :key="task.id" class="list-item">
          <span class="item-icon" :class="getTaskIcon(task.task_type_display).class">{{ getTaskIcon(task.task_type_display).icon }}</span>
          <div class="item-info">
            <span class="item-title">{{ task.task_type_display }}</span>
            <span class="item-status" :class="task.status_display.toLowerCase()">{{ task.status_display }}</span>
          </div>
        </li>
      </ul>
      
      <p v-else class="status-info">暂无历史任务</p>
    </div>
  </aside>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import apiClient from '@/api/axios';
import { useAuthStore } from '@/stores/auth';

const tasks = ref([]);
const isLoading = ref(true);
const error = ref('');
const authStore = useAuthStore();

const getTaskIcon = (taskType) => {
  if (taskType.includes('方案')) return { icon: '💡', class: 'plan' };
  if (taskType.includes('图片')) return { icon: '🖼️', class: 'image' };
  if (taskType.includes('模型')) return { icon: '🧊', class: 'model' };
  if (taskType.includes('聊天')) return { icon: '💬', class: 'chat' };
  return { icon: '📁', class: 'default' };
};

onMounted(async () => {
  if (!authStore.isAuthenticated) {
    error.value = '用户未登录';
    isLoading.value = false;
    return;
  }
  
  try {
    const response = await apiClient.get('/api/ai/tasks/');
    if (response.data && Array.isArray(response.data.results)) {
      tasks.value = response.data.results.slice(0, 15); // 显示最近15条
    } else {
      throw new Error('返回数据格式不正确');
    }
  } catch (err) {
    error.value = '无法加载任务列表';
    console.error('获取任务列表失败:', err);
  } finally {
    isLoading.value = false;
  }
});
</script>

<style scoped>
/* 样式部分保持不变，仅因为按钮移除，布局会自动调整 */
.sidebar { width: 280px; background-color: #FFFFFF; padding: 1.5rem; display: flex; flex-direction: column; height: 100vh; border-right: 1px solid #EEEEEE; box-sizing: border-box; flex-shrink: 0; }
.logo-container { display: flex; align-items: center; padding: 0 0.5rem; margin-bottom: 2rem; } /* 原始为2rem，您可以根据需要微调 */
.logo-link { text-decoration: none; display: flex; align-items: center; }
.logo-icon { width: 40px; height: 40px; background: linear-gradient(45deg, #FFA726, #FFD54F); color: white; display: grid; place-items: center; font-size: 1.5rem; font-weight: bold; border-radius: 10px; margin-right: 0.8rem; }
.logo-text { font-size: 1.5rem; font-weight: 600; margin: 0; color: #333; }
.history-section { flex-grow: 1; overflow-y: auto; }
.history-title { font-size: 0.9rem; text-transform: uppercase; color: #757575; margin-bottom: 1rem; padding: 0 0.5rem; }
.item-list { list-style: none; padding: 0; margin: 0; }
.list-item { display: flex; align-items: center; padding: 0.8rem; border-radius: 8px; margin-bottom: 0.5rem; cursor: pointer; transition: background-color 0.2s; }
.list-item:hover { background-color: #FFF8E1; }
.item-icon { width: 36px; height: 36px; border-radius: 8px; display: grid; place-items: center; font-size: 1.2rem; margin-right: 1rem; }
.item-icon.plan { background-color: rgba(255, 213, 79, 0.3); }
.item-icon.image { background-color: rgba(255, 167, 38, 0.3); }
.item-icon.model { background-color: rgba(255, 184, 77, 0.3); }
.item-icon.chat { background-color: rgba(100, 181, 246, 0.3); }
.item-info { display: flex; flex-direction: column; overflow: hidden; }
.item-title { font-weight: 500; color: #333; font-size: 0.9rem; }
.item-status { font-size: 0.8rem; color: #757575; }
.status-info { padding: 1rem; text-align: center; color: #757575; }
.status-info.error { color: #D32F2F; }
</style>