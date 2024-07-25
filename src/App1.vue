<template>
  <div>
    <textarea v-model="text" placeholder="输入要合成的文本"></textarea>
    <input type="file" @change="handleFileUpload" accept="audio/*" />
    <button @click="generateVoiceClone" :disabled="isLoading">生成语音克隆</button>
    <p v-if="isLoading">处理中...</p>
    <div v-if="uploadedFileUrl">
      <h3>上传的文件URL:</h3>
      <p>{{ uploadedFileUrl }}</p>
    </div>
    <div v-if="message">
      <h3>消息:</h3>
      <p>{{ message }}</p>
    </div>
    <div v-if="errorDetails">
      <h3>错误详情:</h3>
      <pre>{{ errorDetails }}</pre>
    </div>
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
      errorDetails: '',
      audioUrl: null,
      uploadedFileUrl: ''
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
      this.message = `已选择文件: ${file.name}`;
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
        console.log('上传文件API返回数据:', data); // 添加日志

        // 解析 data 对象的结构
        if (data.code === 200 && data.data) {
          this.uploadedFileUrl = data.data;
          console.log('上传文件URL:', this.uploadedFileUrl); // 添加日志
          return this.uploadedFileUrl;
        } else {
          throw new Error('文件上传失败: ' + JSON.stringify(data));
        }
      } catch (error) {
        console.error('文件上传错误:', error);
        this.errorDetails = `文件上传错误: ${error.message}\n${error.stack}`;
        throw error;
      }
    },
    async generateVoiceClone() {
      if (!this.text || !this.audioFile) {
        this.message = '请输入文本并上传音频文件';
        return;
      }

      this.isLoading = true;
      this.message = '开始处理...';
      this.errorDetails = '';
      this.audioUrl = null;

      try {
        this.message = '正在上传文件...';
        const uploadedFileUrl = await this.uploadFile(this.audioFile);

        this.message = '文件上传成功,正在生成语音克隆...';

        const ai_options = new AIOptions({
          appId: 'YOUR_APP_ID' // 替换为你的实际 app id
        });
        const ai = new AI(ai_options);

        // 添加日志
        console.log('AI options:', ai_options);
        console.log('AI instance:', ai);

        let response;
        try {
          response = await ai.prediction("/predictions/ai/xtts-v2", {
            input: {
              "text": this.text,
              "speaker": uploadedFileUrl,
              "language": "en",
              "cleanup_voice": false
            }
          });
          
          // 添加日志
          console.log('API response:', response);
        } catch (apiError) {
          console.error('API调用错误:', apiError);
          this.errorDetails = `API调用错误: ${apiError.message}\n${apiError.stack}\n\n完整错误对象: ${JSON.stringify(apiError, null, 2)}`;
          throw apiError;
        }

        if (response && response.code === 200 && response.data) {
          response = response.data;
        }

        if (response.task && response.task.exec_code === 200 && response.task.is_success) {
          this.audioUrl = response.output;
          this.message = "语音克隆生成成功!";
        } else {
          throw new Error("语音克隆生成失败: " + JSON.stringify(response));
        }
      } catch (error) {
        console.error("语音克隆出错:", error);
        this.message = "发生错误,请查看错误详情。";
        this.errorDetails = `错误: ${error.message}\n${error.stack}\n\n完整错误对象: ${JSON.stringify(error, null, 2)}`;
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
pre {
  white-space: pre-wrap;
  word-wrap: break-word;
  background-color: #f0f0f0;
  padding: 10px;
  border-radius: 5px;
}
</style>
