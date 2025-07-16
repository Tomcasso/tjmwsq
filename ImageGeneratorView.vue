<template>
  <div class="image-generator-page">
    <div class="content-wrapper">
      <header class="page-header">
        <router-link to="/dashboard" class="back-link">← 返回主页</router-link>
        <h1>AI效果图创作</h1>
        <p>输入您的灵感，Vision AI 将为您绘制媲美真实照片的家装效果图。</p>
      </header>

      <div class="input-section">
        <textarea 
          v-model="formData.prompt"
          class="prompt-input"
          placeholder="请在这里输入详细的画面描述，例如：“一间现代风格的客厅，有白色的布艺沙发，落地窗，阳光明媚，木地板”"
          :disabled="imageStore.taskStatus === 'loading'"
        ></textarea>
        <div class="options-and-submit">
          <div class="options-group">
            <div class="form-group">
              <label for="num_images">生成张数</label>
              <select id="num_images" v-model="formData.num_images" :disabled="imageStore.taskStatus === 'loading'">
                <option value="1">1张</option>
                <option value="2">2张</option>
                <option value="3">3张</option>
                <option value="4">4张</option>
              </select>
            </div>
            <div class="form-group">
              <label for="aspect_ratio">图片比例</label>
              <select id="aspect_ratio" v-model="formData.aspect_ratio" :disabled="imageStore.taskStatus === 'loading'">
                <option value="16:9">宽屏 (16:9)</option>
                <option value="1:1">方形 (1:1)</option>
                <option value="9:16">竖屏 (9:16)</option>
                <option value="4:3">标准 (4:3)</option>
                <option value="3:2">风景 (3:2)</option>
              </select>
            </div>
          </div>
          <button @click="submit" class="submit-btn" :disabled="imageStore.taskStatus === 'loading' || !formData.prompt.trim()">
            <span v-if="imageStore.taskStatus === 'loading'" class="loading-content">
              <div class="spinner"></div>
              生成中...
            </span>
            <span v-else>立即生成</span>
          </button>
        </div>
      </div>

      <div class="results-section">
        <div v-if="imageStore.taskStatus === 'loading'" class="status-view">
           <div class="loader"></div>
           <p>AI画师正在创作中，请稍候...</p>
        </div>
        
        <div v-else-if="imageStore.taskStatus === 'error'" class="status-view error-view">
          <p class="error-icon">😕</p>
          <p>糟糕，出错了！</p>
          <p class="error-message">{{ imageStore.error }}</p>
        </div>

        <div v-else-if="imageStore.taskStatus === 'success' && imageStore.generatedImages.length > 0" class="image-gallery">
          <div v-for="(url, index) in imageStore.generatedImages" :key="index" class="image-wrapper">
            <div v-if="activeSegmentationUrl !== url" class="image-container">
              <img :src="url" alt="AI生成的效果图">
              <div class="image-actions">
                <a :href="url" download class="action-btn" title="下载图片">
                  <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path><polyline points="7 10 12 15 17 10"></polyline><line x1="12" y1="15" x2="12" y2="3"></line></svg>
                  <span>下载</span>
                </a>
                <button @click="activateSegmentationMode(url)" class="action-btn" title="分割图像">
                  <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M14.5 3.5a2.12 2.12 0 0 1 3 3L7 19l-4 1 1-4Z"></path><path d="m15 5 3 3"></path></svg>
                  <span>分割</span>
                </button>
              </div>
            </div>
            
            <InteractiveImage 
              v-else
              :image-url="url"
              @start-segment="payload => handleStartSegmentation(url, payload)"
            />
          </div>
        </div>

        <div v-else-if="imageStore.taskStatus === 'idle' && imageStore.generatedImages.length === 0" class="status-view welcome-mat">
          <img :src="welcomeImageUrl" alt="AI Art Inspiration" class="welcome-image">
        </div>
      </div>
      
      <div class="segmentation-results-section" v-if="imageStore.segmentTaskId">
          <hr>
          <h3 class="results-title">分割任务结果</h3>
          <div v-if="imageStore.segmentTaskStatus === 'loading'" class="status-view">
            <div class="loader"></div>
            <p>正在分割图像，请稍候...</p>
          </div>
          <div v-else-if="imageStore.segmentTaskStatus === 'success' && imageStore.segmentationResultUrl" class="segment-result">
            <div class="image-container">
              <img :src="imageStore.segmentationResultUrl" alt="分割后的图像">
              <div class="image-actions">
                 <a :href="imageStore.segmentationResultUrl" download class="action-btn" title="下载分割图">
                    <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path><polyline points="7 10 12 15 17 10"></polyline><line x1="12" y1="15" x2="12" y2="3"></line></svg>
                    <span>下载</span>
                  </a>
                 <button 
                    @click="handleGenerate3d"
                    class="action-btn generate-3d" 
                    title="基于此图生成3D模型"
                    :disabled="imageStore.model3dTaskStatus === 'loading'"
                  >
                    <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="m21.12 6-1.5-1.5a2.12 2.12 0 0 0-3 0L3 18.003V21h2.997Z"></path><path d="m15 7 3 3"></path><path d="M22 22 8 8"></path><path d="M10 22H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h12a2 2 0 0 1 2 2v6"></path></svg>
                    <span>生成3D</span>
                  </button>
              </div>
            </div>
          </div>
          <div v-else-if="imageStore.segmentTaskStatus === 'error'" class="status-view error-view">
            <p>😕 分割出错了！</p>
            <p class="error-message">{{ imageStore.segmentationError }}</p>
          </div>
      </div>

      <div class="final-task-section" v-if="imageStore.model3dTaskId">
          <hr>
          <h3 class="results-title">3D模型生成任务</h3>
           <div v-if="imageStore.model3dTaskStatus === 'loading'" class="status-view">
            <div class="loader"></div>
            <p>正在启动3D模型生成，这可能需要几分钟时间...</p>
          </div>
           <div v-else-if="imageStore.model3dTaskStatus === 'success'" class="status-view success-view">
            <p class="success-icon">✅</p>
            <h4>3D模型生成任务已成功创建！</h4>
            <p>您可以到主仪表盘的“最近任务”列表中查看它的实时进度。</p>
          </div>
           <div v-else-if="imageStore.model3dTaskStatus === 'error'" class="status-view error-view">
            <p class="error-icon">😕</p>
            <p>启动3D模型任务失败！</p>
            <p class="error-message">{{ imageStore.model3dError }}</p>
          </div>
      </div>

    </div>
  </div>
</template>

<script setup>
import { ref, onUnmounted } from 'vue';
import { useImageStore } from '@/stores/image';
import InteractiveImage from '@/components/InteractiveImage.vue';
import welcomeImageUrl from '@/assets/images/welcome-image.jpg';

const imageStore = useImageStore();
const formData = ref({ prompt: '', aspect_ratio: '16:9', num_images: 2 });

// 【核心修改】新增状态，记录哪张图正处于“分割模式”
const activeSegmentationUrl = ref(null);

const submit = () => {
  if (!formData.value.prompt.trim()) { alert('请输入画面描述！'); return; }
  activeSegmentationUrl.value = null; // 开始一次新生成时，退出任何已激活的分割模式
  imageStore.generateImages(formData.value);
};

// 【核心修改】点击“分割”按钮时，激活对应图片的交互模式
const activateSegmentationMode = (imageUrl) => {
  activeSegmentationUrl.value = imageUrl;
};

// 处理来自InteractiveImage组件的事件
const handleStartSegmentation = (imageUrl, payload) => {
    if (payload.points.length === 0 && payload.boxes.length === 0) {
        alert("请先在图片上标记需要分割的物体！");
        return;
    }
    imageStore.startSegmentation(imageUrl, payload);
    activeSegmentationUrl.value = null; // 提交后退出交互模式
};

const handleGenerate3d = () => {
    const sourceTaskId = imageStore.segmentTaskId;
    if (!sourceTaskId) {
        alert("错误：找不到源分割任务ID，无法继续！");
        return;
    }
    imageStore.generate3dFromCutout(sourceTaskId);
};

onUnmounted(() => { imageStore.reset(); });
</script>

<style scoped>
/* 样式与之前基本一致，只为新区域增加样式 */
.image-generator-page { background-color: #FFF8E1; min-height: 100vh; padding: 2rem 4rem; box-sizing: border-box; font-family: inherit; }
.content-wrapper { max-width: 1200px; margin: 0 auto; }
.page-header { text-align: center; margin-bottom: 2rem; position: relative; }
.back-link { position: absolute; left: 0; top: 50%; transform: translateY(-50%); text-decoration: none; color: #555; font-weight: 500; }
.page-header h1 { font-size: 2rem; }
.page-header p { color: #757575; }
.input-section { background: #fff; padding: 1.5rem 2rem; border-radius: 12px; box-shadow: 0 4px 20px rgba(0,0,0,0.05); margin-bottom: 2rem; }
.prompt-input { width: 100%; min-height: 100px; padding: 1rem; border: 1px solid #E0E0E0; border-radius: 8px; resize: vertical; font-size: 1rem; margin-bottom: 1rem; box-sizing: border-box; font-family: inherit; }
.options-and-submit { display: flex; justify-content: space-between; align-items: center; gap: 1rem; }
.options-group { display: flex; gap: 1.5rem; }
.form-group { display: flex; align-items: center; }
.form-group label { margin-right: 0.5rem; color: #555; font-weight: 500;}
.form-group select { padding: 0.8rem; border-radius: 6px; border: 1px solid #ccc; font-size: 0.9rem; }
.submit-btn { display: inline-flex; align-items: center; justify-content: center; gap: 0.5rem; padding: 0.8rem 2rem; min-width: 130px; height: 42px; box-sizing: border-box; border: none; border-radius: 8px; background-color: #FFA726; color: white; font-size: 1rem; font-weight: 600; cursor: pointer; transition: background-color 0.2s; }
.submit-btn:disabled { background-color: #ccc; cursor: not-allowed; }
.loading-content { display: inline-flex; align-items: center; }
.spinner { width: 16px; height: 16px; border: 2px solid rgba(255,255,255,0.3); border-top-color: #fff; border-radius: 50%; animation: spin 1s linear infinite; }
.results-section { min-height: 400px; }
.status-view { display: flex; flex-direction: column; justify-content: center; align-items: center; min-height: 400px; text-align: center; padding: 2rem; color: #757575; background-color: #fff; border-radius: 12px; }
.error-view .error-icon { font-size: 3rem; margin-bottom: 1rem; }
.error-view .error-message { color: #D32F2F; background-color: #ffebee; padding: 1rem; border-radius: 8px; display: inline-block; }
.loader { border: 4px solid #f3f3f3; border-top: 4px solid #FFA726; border-radius: 50%; width: 50px; height: 50px; animation: spin 1s linear infinite; margin-bottom: 1.5rem; }
@keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
.welcome-mat { padding: 0; overflow: hidden; height: 500px; }
.welcome-mat img { width: 100%; height: 100%; object-fit: cover; border-radius: 12px; }
.image-gallery { display: grid; grid-template-columns: repeat(auto-fill, minmax(400px, 1fr)); gap: 1.5rem; }
.image-wrapper { border-radius: 8px; background: #fff; padding: 0.5rem; }
.image-container, .segment-result .image-container { position: relative; overflow: hidden; border-radius: 8px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); display: inline-block; }
.image-container img { width: 100%; display: block; }
.image-actions { position: absolute; bottom: 1rem; right: 1rem; padding: 0.5rem; background: rgba(255, 255, 255, 0.8); backdrop-filter: blur(5px); border-radius: 8px; display: flex; gap: 0.5rem; opacity: 0; transform: translateY(10px); transition: all 0.3s ease; pointer-events: none; }
.image-container:hover .image-actions { opacity: 1; transform: translateY(0); pointer-events: auto; }
.action-btn { display: inline-flex; align-items: center; gap: 0.4rem; background: #fff; color: #333; border: 1px solid #ddd; padding: 0.5rem 1rem; border-radius: 6px; cursor: pointer; text-decoration: none; font-size: 0.85rem; font-weight: 500; transition: all 0.2s; }
.action-btn:hover { background-color: #f0f0f0; border-color: #ccc; }
.action-btn svg { stroke: #555; }
.action-btn.generate-3d { background-color: #4CAF50; color: white; border-color: #4CAF50; }
.action-btn.generate-3d:hover { background-color: #43A047; }
.action-btn.generate-3d svg { stroke: white; }
.segmentation-results-section { margin-top: 3rem; }
.segmentation-results-section hr { border: none; border-top: 1px solid #e0e0e0; margin-bottom: 2rem; }
.segmentation-results-section .results-title { text-align: center; font-size: 1.5rem; margin-bottom: 2rem; }
.segment-result { text-align: center; }
.final-task-section { margin-top: 3rem; }
.final-task-section hr { border: none; border-top: 1px solid #e0e0e0; margin-bottom: 2rem; }
.final-task-section .results-title { text-align: center; font-size: 1.5rem; margin-bottom: 2rem; }
.success-view .success-icon { font-size: 3rem; margin-bottom: 1rem; }
.success-view h4 { font-size: 1.2rem; color: #333; }
</style>