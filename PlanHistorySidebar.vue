<template>
  <aside class="sidebar">
    <div class="logo-container">
      <router-link to="/" class="logo-link">
        <span class="logo-icon">V</span>
        <h2 class="logo-text">Vision</h2>
      </router-link>
    </div>

    <router-link to="/chat" class="new-item-btn">+ 新建对话</router-link>

    <div class="history-section">
      <h3 class="history-title">历史方案</h3>
      
      <div v-if="isLoading" class="status-info">正在加载...</div>
      <div v-else-if="error" class="status-info error">{{ error }}</div>
      
      <ul v-else-if="plans.length > 0" class="item-list">
        <li v-for="plan in plans" :key="plan.id" class="list-item" @click="navigateToPlan(plan.id)">
          <span class="item-icon plan">💡</span>
          <div class="item-info">
            <span class="item-title">{{ plan.title }}</span>
            <span class="item-status">{{ formatTime(plan.created_at) }}</span>
          </div>
        </li>
      </ul>
      
      <p v-else class="status-info">暂无历史方案</p>
    </div>
  </aside>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import apiClient from '@/api/axios';
import { useRouter } from 'vue-router';

const plans = ref([]);
const isLoading = ref(true);
const error = ref('');
const router = useRouter();

// 辅助函数：格式化时间
const formatTime = (timeStr) => {
  if (!timeStr) return '';
  return new Date(timeStr).toLocaleDateString();
};

// 点击列表项后的跳转逻辑
const navigateToPlan = (planId) => {
  // TODO: 将来这里应该跳转到方案详情页
  // router.push(`/plans/${planId}`);
  alert(`您点击了方案ID: ${planId}，详情页功能待实现。`);
};

onMounted(async () => {
  isLoading.value = true;
  try {
    // 【核心不同】调用我们为方案列表专门创建的API接口
    const response = await apiClient.get('/ai/plans/');
    if (response.data && Array.isArray(response.data.results)) {
      plans.value = response.data.results;
    } else {
      throw new Error('返回数据格式不正确');
    }
  } catch (err) {
    error.value = '无法加载历史方案';
    console.error('获取方案列表失败:', err);
  } finally {
    isLoading.value = false;
  }
});
</script>

<style scoped>
/* 样式可以完全复用 DashboardSidebar.vue 的样式，保持风格统一 */
.sidebar { width: 280px; background-color: #FFFFFF; padding: 1.5rem; display: flex; flex-direction: column; height: 100vh; border-right: 1px solid #EEEEEE; box-sizing: border-box; flex-shrink: 0; }
.logo-container { display: flex; align-items: center; padding: 0 0.5rem; margin-bottom: 2rem; }
.logo-link { text-decoration: none; display: flex; align-items: center; }
.logo-icon { width: 40px; height: 40px; background: linear-gradient(45deg, #FFA726, #FFD54F); color: white; display: grid; place-items: center; font-size: 1.5rem; font-weight: bold; border-radius: 10px; margin-right: 0.8rem; }
.logo-text { font-size: 1.5rem; font-weight: 600; margin: 0; color: #333; }
.new-item-btn { width: 100%; padding: 0.9rem; text-align: center; text-decoration: none; background-color: #FFA726; color: white; border: none; border-radius: 8px; font-size: 1rem; cursor: pointer; margin-bottom: 2rem; transition: all 0.2s; }
.new-item-btn:hover { background-color: #FB8C00; }
.history-section { flex-grow: 1; overflow-y: auto; }
.history-title { font-size: 0.9rem; text-transform: uppercase; color: #757575; margin-bottom: 1rem; padding: 0 0.5rem; }
.item-list { list-style: none; padding: 0; margin: 0; }
.list-item { display: flex; align-items: center; padding: 0.8rem; border-radius: 8px; margin-bottom: 0.5rem; cursor: pointer; transition: background-color 0.2s; }
.list-item:hover { background-color: #FFF8E1; }
.item-icon { width: 36px; height: 36px; border-radius: 8px; display: grid; place-items: center; font-size: 1.2rem; margin-right: 1rem; }
.item-icon.plan { background-color: rgba(255, 213, 79, 0.3); }
.item-info { display: flex; flex-direction: column; overflow: hidden; }
.item-title { font-weight: 500; color: #333; font-size: 0.9rem; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.item-status { font-size: 0.8rem; color: #757575; }
.status-info { padding: 1rem; text-align: center; color: #757575; }
.status-info.error { color: #D32F2F; }
</style>