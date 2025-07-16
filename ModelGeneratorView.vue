<template>
  <div class="model-generator-page">
    <div class="content-wrapper">
      <header class="page-header">
        <router-link to="/dashboard" class="back-link">← 返回主页</router-link>
        <h1>AI生成3D模型</h1>
        <p>通过文字或图片，快速生成可用于VR/MR场景的3D模型资产。</p>
      </header>

      <div class="generator-container">
        <div class="tabs">
          <button class="tab-btn" :class="{active: activeTab === 'text'}" @click="switchTab('text')">文生3D</button>
          <button class="tab-btn" :class="{active: activeTab === 'image'}" @click="switchTab('image')">图生3D</button>
        </div>

        <div v-if="activeTab === 'text'" class="form-content">
          <form @submit.prevent="handleTextSubmit">
            <div class="form-group">
                <label for="prompt">模型描述 (Prompt)</label>
                <textarea v-model="textFormData.prompt" id="prompt" class="prompt-input" placeholder="输入详细的模型描述，如：一个红色的、带有金色花纹的中国古代花瓶" required></textarea>
            </div>
            <div class="form-grid">
                <div class="form-group">
                    <label for="art_style">艺术风格</label>
                    <input type="text" id="art_style" v-model="textFormData.art_style" placeholder="例如: 写实、卡通、雕塑...">
                </div>
                <div class="form-group">
                    <label for="name">模型名称 (可选)</label>
                    <input type="text" id="name" v-model="textFormData.name" placeholder="为您的模型起个名字">
                </div>
            </div>
            <div class="form-group">
                <label for="negative_prompt">不希望出现的元素 (可选)</label>
                <input type="text" id="negative_prompt" v-model="textFormData.negative_prompt" placeholder="例如: 模糊、低质量、瑕疵">
            </div>
            <button type="submit" class="submit-btn" :disabled="modelStore.taskStatus === 'loading'">立即生成</button>
          </form>
        </div>

        <div v-if="activeTab === 'image'" class="form-content">
          <form @submit.prevent="handleImageSubmit">
            <div 
              class="drop-zone"
              @dragover.prevent @dragenter.prevent @dragleave.prevent @drop.prevent="handleDrop"
            >
              <input type="file" @change="handleFileChange" accept="image/*" ref="fileInputRef" style="display: none;">
              <div v-if="!imagePreviewUrl" class="upload-placeholder">
                <p>将图片拖拽到此处，或 <button type="button" @click="triggerFileInput" class="browse-btn">点击选择文件</button></p>
              </div>
              <div v-else class="image-preview">
                <img :src="imagePreviewUrl" alt="图片预览">
                <button type="button" @click="clearImage" class="clear-image-btn">×</button>
              </div>
            </div>
            <button type="submit" class="submit-btn" :disabled="!imageFile || modelStore.taskStatus === 'loading'">立即生成</button>
          </form>
        </div>
      </div>

      <div class="results-section" v-if="modelStore.taskStatus !== 'idle'">
        <div v-if="modelStore.taskStatus === 'loading'" class="status-view">
          <div class="loader"></div>
          <p>AI正在构建三维空间，请稍候...</p>
        </div>
        <div v-else-if="modelStore.taskStatus === 'error'" class="status-view error-view">
          <p class="error-icon">😕</p>
          <p>糟糕，出错了！</p>
          <p class="error-message">{{ modelStore.error }}</p>
        </div>
        <div v-else-if="modelStore.taskStatus === 'success'" class="status-view success-view">
          <p class="success-icon">✅</p>
          <h4>3D模型任务已成功创建！</h4>
          <p>您可以到主仪表盘的“最近任务”列表中查看它的实时进度。</p>
        </div>
      </div>

    </div>
  </div>
</template>

<script setup>
import { ref, onUnmounted } from 'vue';
import { useModelStore } from '@/stores/model';

const modelStore = useModelStore();
const activeTab = ref('text');

const textFormData = ref({
  prompt: '',
  name: '',
  art_style: 'realistic',
  negative_prompt: 'low quality, blurry, ugly',
});

const imageFile = ref(null);
const imagePreviewUrl = ref(null);
const fileInputRef = ref(null);

const switchTab = (tab) => {
    activeTab.value = tab;
    modelStore.reset();
}

const handleFileChange = (event) => {
  const file = event.target.files[0];
  if (file) {
    imageFile.value = file;
    imagePreviewUrl.value = URL.createObjectURL(file);
  }
};
const handleDrop = (event) => {
  const file = event.dataTransfer.files[0];
  if (file && file.type.startsWith('image/')) {
    imageFile.value = file;
    imagePreviewUrl.value = URL.createObjectURL(file);
  }
};
const triggerFileInput = () => { fileInputRef.value.click(); };
const clearImage = () => {
  imageFile.value = null;
  imagePreviewUrl.value = null;
  if(fileInputRef.value) {
    fileInputRef.value.value = '';
  }
};
const handleTextSubmit = () => {
  if (!textFormData.value.prompt.trim()) { alert('请输入模型描述！'); return; }
  modelStore.generateFromText(textFormData.value);
};
const handleImageSubmit = () => {
  if (!imageFile.value) { alert('请先选择一张图片！'); return; }
  const formData = new FormData();
  formData.append('source_image', imageFile.value);
  modelStore.generateFromImage(formData);
};
onUnmounted(() => { modelStore.reset(); });
</script>

<style scoped>
.model-generator-page { background-color: #FFF8E1; min-height: 100vh; padding: 2rem 4rem; box-sizing: border-box; font-family: inherit; }
.content-wrapper { max-width: 800px; margin: 0 auto; }
.page-header { text-align: center; margin-bottom: 2rem; position: relative; }
.back-link { position: absolute; left: 0; top: 50%; transform: translateY(-50%); text-decoration: none; color: #555; font-weight: 500; }
.page-header h1 { font-size: 2rem; }
.page-header p { color: #757575; }
.generator-container { background: #fff; border-radius: 12px; box-shadow: 0 4px 20px rgba(0,0,0,0.05); }
.tabs { display: flex; border-bottom: 1px solid #eee; }
.tab-btn { flex: 1; padding: 1rem; border: none; background: transparent; font-size: 1.1rem; cursor: pointer; color: #757575; transition: all 0.2s; }
.tab-btn.active { color: #FFA726; font-weight: 600; border-bottom: 2px solid #FFA726; }
.form-content { padding: 2rem; }
.form-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 1.5rem; margin-bottom: 1.5rem; }
.form-group { display: flex; flex-direction: column; text-align: left; }
.form-group label { margin-bottom: 0.5rem; font-weight: 500; color: #424242; font-size: 1rem; }
.form-group input, .form-group select, .prompt-input { 
    width: 100%; 
    padding: 0.8rem 1rem; 
    border: 1px solid #E0E0E0; 
    border-radius: 8px; 
    box-sizing: border-box; 
    font-size: 1rem; 
    font-family: inherit;
}
.prompt-input { min-height: 120px; resize: vertical; margin-bottom: 1.5rem; }

/* 【核心修改】移除了 font-style: italic; */
.form-group input::placeholder,
.prompt-input::placeholder {
    color: #bdbdbd;
    font-weight: 400;
}

.submit-btn { display: block; width: 220px; margin: 2rem auto 0; padding: 1rem; border: none; border-radius: 8px; background-color: #FFA726; color: white; font-size: 1.1rem; font-weight: 600; cursor: pointer; transition: all 0.2s; }
.submit-btn:hover:not(:disabled) { background-color: #FB8C00; }
.submit-btn:disabled { background-color: #E0E0E0; cursor: not-allowed; }
.drop-zone { border: 2px dashed #ccc; border-radius: 8px; padding: 2rem; text-align: center; color: #757575; min-height: 200px; display: flex; justify-content: center; align-items: center; transition: all 0.2s; }
.drop-zone:hover { border-color: #FFA726; background-color: #fffaf0; }
.browse-btn { color: #FFA726; background: none; border: none; text-decoration: underline; cursor: pointer; font-size: 1rem; }
.image-preview { position: relative; }
.image-preview img { max-height: 200px; border-radius: 8px; }
.clear-image-btn { position: absolute; top: -10px; right: -10px; background: #333; color: white; border: none; border-radius: 50%; width: 24px; height: 24px; cursor: pointer; font-size: 1rem; line-height: 24px; }
.results-section { margin-top: 2rem; }
.status-view { min-height: 150px; display: flex; flex-direction: column; justify-content: center; align-items: center; text-align: center; background: #fff; border-radius: 12px; padding: 2rem; }
.loader { border: 4px solid #f3f3f3; border-top: 4px solid #FFA726; border-radius: 50%; width: 40px; height: 40px; animation: spin 1s linear infinite; margin-bottom: 1rem; }
@keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
.success-view .success-icon { font-size: 2.5rem; }
.success-view h4 { font-size: 1.2rem; color: #333; margin: 1rem 0 0.5rem 0; }
.error-view .error-icon { font-size: 2.5rem; }
.error-view .error-message { color: #D32F2F; background-color: #ffebee; padding: 1rem; border-radius: 8px; display: inline-block; margin-top: 1rem; }
</style>