<template>
    <div>
      <textarea v-model="text" placeholder="输入要合成的文本"></textarea>
      <input type="file" @change="handleFileUpload" accept="audio/*" />
      <button @click="generateVoiceClone" :disabled="isLoading">生成语音克隆</button>
      <p v-if="isLoading">处理中...</p>
      <p v-if="message">{{ message }}</p>
      <audio v-if="audioUrl" controls :src="audioUrl"></audio>
    </div>
  </template>
  
  <script>
  import { AI, AIOptions } from 'aonweb'
  
  export default {
    data() {
      return {
        text: '',
        audioFile: null,
        isLoading: false,
        message: '',
        audioUrl: null
      }
    },
    methods: {
      handleFileUpload(event) {
        const file = event.target.files[0];
        if (file.size > 30 * 1024 * 1024) {
          this.message = '文件大小不能超过 30MB';
          return;
        }
        this.audioFile = file;
      },
      async uploadFile(file) {
        const formData = new FormData();
        formData.append('file', file);
  
        try {
          const response = await fetch('https://tmp-file.aigic.ai/api/v1/upload?expires=1800&type=audio/wav', {
            method: 'POST',
            body: formData
          });
  
          const data = await response.json();
          if (data.code === 200 && data.data && data.data.length) {
            return data.data[0];
          } else {
            throw new Error('文件上传失败');
          }
        } catch (error) {
          console.error('文件上传错误:', error);
          throw error;
        }
      },
      async generateVoiceClone() {
        if (!this.text || !this.audioFile) {
          this.message = '请输入文本并上传音频文件';
          return;
        }
  
        this.isLoading = true;
        this.message = '';
        this.audioUrl = null;
  
        try {
          const uploadedFileUrl = await this.uploadFile(this.audioFile);
  
          const ai_options = new AIOptions({
            appId: 'YOUR_APP_ID' // 替换为你的实际 app id
          });
          const ai = new AI(ai_options);
  
          let response = await ai.prediction("/predictions/ai/xtts-v2", {
            input: {
              "text": this.text,
              "speaker": uploadedFileUrl,
              "language": "en",
              "cleanup_voice": false
            }
          });
  
          if (response && response.code === 200 && response.data) {
            response = response.data;
          }
  
          if (response.task.exec_code === 200 && response.task.is_success) {
            this.audioUrl = response.output;
            this.message = "语音克隆生成成功!";
          } else {
            this.message = "语音克隆生成失败,请重试。";
          }
        } catch (error) {
          console.error("语音克隆出错:", error);
          this.message = "发生错误,请重试。";
        } finally {
          this.isLoading = false;
        }
      }
    }
  }
  </script>
  
  <style scoped>
  textarea, input, button {
    display: block;
    margin-bottom: 10px;
  }
  textarea {
    width: 100%;
    height: 100px;
  }
  </style>