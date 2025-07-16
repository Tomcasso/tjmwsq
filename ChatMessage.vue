<template>
  <div class="chat-message" :class="[message.role]">
    <div class="avatar">
      <span v-if="message.role === 'user'">🧑</span>
      <span v-else>🤖</span>
    </div>
    <div class="message-content-wrapper">
      <div class="message-content" v-html="renderMarkdown(message.content)"></div>
    </div>
  </div>
</template>

<script setup>
import { marked } from 'marked';

const props = defineProps({
  message: {
    type: Object,
    required: true,
  }
});

// 将Markdown文本转换为HTML
const renderMarkdown = (content) => {
  if (!content) return '';
  // 使用marked库解析Markdown，并启用GFM和换行符
  return marked.parse(content, { gfm: true, breaks: true });
};
</script>

<style scoped>
/* 【核心修改】调整布局，让气泡和头像底部对齐 */
.chat-message {
  display: flex;
  gap: 1rem;
  margin-bottom: 1.5rem;
  max-width: 90%;
  align-items: flex-end; /* 强制所有子元素底部对齐 */
}

.avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: #eee;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.5rem;
  flex-shrink: 0;
}

.message-content-wrapper {
  display: flex;
  flex-direction: column; /* 确保内容是垂直堆叠的 */
}

.message-content {
  padding: 0.1rem 1.2rem; /* 减小垂直内边距 */
  border-radius: 18px;
  background: #fff;
  color: #333;
  line-height: 2.0;
  word-wrap: break-word; /* 确保长单词或链接能换行 */
}

/* 用户的消息样式 */
.chat-message.user {
  flex-direction: row-reverse;
  margin-left: auto;
}
.chat-message.user .avatar { background: #FFA726; }
.chat-message.user .message-content-wrapper { align-items: flex-end; }
.chat-message.user .message-content {
  background: #FFA726;
  color: white;
  border-bottom-right-radius: 4px;
}

/* AI助手的消息样式 */
.chat-message.assistant {
  flex-direction: row;
  margin-right: auto;
}
.chat-message.assistant .message-content-wrapper { align-items: flex-start; }
.chat-message.assistant .message-content {
  background: #FFFFFF;
  border: 1px solid #e5e5e5;
  border-bottom-left-radius: 4px;
}

/* 用于渲染Markdown的样式 */
.prose {
  white-space: pre-wrap;
}
/* 确保Markdown解析出的段落之间有合适的间距 */
.prose :deep(p) {
  margin: 0 0 0.5rem 0;
}
.prose :deep(p:last-child) {
  margin-bottom: 0;
}
.prose :deep(ul), .prose :deep(ol) {
  padding-left: 1.5rem;
  margin-bottom: 0.5rem;
}
</style>