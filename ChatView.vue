<template>
  <div class="chat-page-layout">
    <ChatHistorySidebar class="sidebar" />
    
    <div class="chat-view">
      <div class="messages-container" ref="messagesContainerRef">
        <div v-if="isLoading" class="status-display">
          <div class="loader"></div>
          <p>正在加载对话...</p>
        </div>
        
        <div v-else-if="!activeSession" class="status-display welcome-display">
          <div class="welcome-icon-container">
            <img src="https://img.icons8.com/fluency-systems-regular/96/armchair.png" alt="Vision AI Furniture Icon">
          </div>
          <h1>Vision 智能家装助手</h1>
          <p>从左侧选择历史对话，或点击“+”开启新的创作之旅</p>
        </div>

        <div v-else>
          <ChatMessage 
            v-for="(msg, index) in activeSession.history" 
            :key="index" 
            :message="msg"
          />
          <div v-if="isSendingMessage" class="typing-indicator-wrapper">
             <div class="avatar">🤖</div>
             <div class="message-content typing-indicator">
                <span></span><span></span><span></span>
             </div>
          </div>
        </div>
      </div>

      <div class="input-area">
        <div class="input-wrapper">
          <textarea 
            v-model="newMessage" 
            @keydown.enter.exact.prevent="sendMessage"
            placeholder="输入您的问题，或描述您的家装想法..."
            :disabled="isSendingMessage"
            @input="autoResizeTextarea"
            ref="textareaRef"
            rows="1"
          ></textarea>
          <button @click="sendMessage" :disabled="isSendingMessage || !newMessage.trim()">发送</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, nextTick } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { useChatStore } from '@/stores/chat';
import { storeToRefs } from 'pinia';
import ChatHistorySidebar from '@/components/ChatHistorySidebar.vue';
import ChatMessage from '@/components/ChatMessage.vue';
// welcomeImageUrl 已不再需要，可以删除
// import welcomeImageUrl from '@/assets/images/welcome-image.jpg';

const route = useRoute();
const router = useRouter();
const chatStore = useChatStore();
const { activeSession, isLoading, isSendingMessage } = storeToRefs(chatStore);

const newMessage = ref('');
const messagesContainerRef = ref(null);
const textareaRef = ref(null);

watch(() => route.params.sessionId, (newId) => {
  if (newId) {
    chatStore.loadSession(newId);
  } else {
    chatStore.startNewChat();
  }
}, { immediate: true });


watch(() => activeSession.value?.history, () => {
  scrollToBottom();
}, { deep: true });

const sendMessage = () => {
  if (!newMessage.value.trim()) return;
  chatStore.sendMessage(newMessage.value);
  newMessage.value = '';
  nextTick(() => {
      if (textareaRef.value) {
          textareaRef.value.style.height = 'auto';
      }
  });
};

const scrollToBottom = () => {
  nextTick(() => {
    if (messagesContainerRef.value) {
      messagesContainerRef.value.scrollTop = messagesContainerRef.value.scrollHeight;
    }
  });
};

const autoResizeTextarea = () => {
    const el = textareaRef.value;
    if (el) {
        el.style.height = 'auto';
        el.style.height = (el.scrollHeight) + 'px';
    }
};

watch(newMessage, autoResizeTextarea);
</script>

<style scoped>
/* 【核心修改】为聊天页面根元素添加背景色 */
.chat-page-layout { 
  display: flex; 
  height: 100vh; 
  width: 100vw; 
  background-color: #FFF8E1; /* <-- 和主仪表盘一致的背景色 */
}
.sidebar { 
  width: 280px; 
  flex-shrink: 0; 
  background-color: #FFFFFF; /* 侧边栏保持白色 */
  border-right: 1px solid #e0e0e0;
}
.chat-view { 
  display: flex; 
  flex-direction: column; 
  flex-grow: 1; 
  height: 100%; 
  /* 【核心修改】移除chat-view的背景色，让父元素的背景色透出来 */
}
.messages-container { 
  flex-grow: 1; 
  padding: 2rem; 
  overflow-y: auto; 
  /* 【核心修改】移除消息容器的背景色 */
}
.status-display { text-align: center; margin: auto; color: #999; display: flex; flex-direction: column; align-items: center; }
.loader { border: 4px solid #f3f3f3; border-top: 4px solid #FFA726; border-radius: 50%; width: 40px; height: 40px; animation: spin 1s linear infinite; margin: 0 auto 1rem; }
@keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }

.welcome-display h1 { font-size: 1.8rem; margin-top: 1rem; color: #555; }
.welcome-display p { font-size: 1rem; }
.welcome-icon-container {
  width: 96px;
  height: 96px;
  margin-bottom: 1.5rem;
  background-color: #fff;
  border-radius: 24px;
  display: grid;
  place-items: center;
  box-shadow: 0 8px 24px rgba(0,0,0,0.05);
}
.welcome-icon-container img {
  width: 60px;
  height: 60px;
  opacity: 0.7;
}

.input-area { 
  flex-shrink: 0; 
  padding: 1rem 2rem; 
  border-top: 1px solid #e0e0e0; 
  background: #ffffff; 
  box-shadow: 0 -5px 15px rgba(0,0,0,0.02); 
}
.input-wrapper { display: flex; align-items: flex-end; padding: 0.5rem; border: 1px solid #e0e0e0; border-radius: 12px; background: #f9f9f9; transition: border-color 0.2s, box-shadow 0.2s; }
.input-wrapper:focus-within { border-color: #FFA726; box-shadow: 0 0 0 3px rgba(255, 167, 38, 0.2); }
textarea { flex-grow: 1; padding: 0.5rem; border: none; resize: none; font-size: 1rem; font-family: inherit; background: transparent; line-height: 1.6; max-height: 150px; overflow-y: auto; }
textarea:focus { outline: none; }
textarea::placeholder { color: #bbb; }
button { padding: 0.7rem 1.5rem; margin-left: 0.5rem; border: none; background: #FFA726; color: white; border-radius: 8px; cursor: pointer; font-weight: 500; font-size: 0.9rem; transition: background-color 0.2s; }
button:disabled { background: #ccc; }
.typing-indicator-wrapper { display: flex; gap: 1rem; align-items: flex-start; margin-bottom: 1.5rem; }
.typing-indicator-wrapper .avatar { width: 40px; height: 40px; border-radius: 50%; background: #eee; display: flex; align-items: center; justify-content: center; font-size: 1.5rem; flex-shrink: 0; }
.typing-indicator { padding: 1rem 1.2rem; background: #f1f1f1; border-radius: 18px; border-bottom-left-radius: 4px; }
.typing-indicator span { height: 8px; width: 8px; background-color: #ccc; border-radius: 50%; display: inline-block; margin: 0 2px; animation: wave 1.3s infinite; }
.typing-indicator span:nth-child(2) { animation-delay: 0.2s; }
.typing-indicator span:nth-child(3) { animation-delay: 0.4s; }
@keyframes wave { 0%, 60%, 100% { transform: initial; } 30% { transform: translateY(-8px); } }
</style>