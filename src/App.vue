<template>
  <div>
    <input v-model="text" placeholder="Enter text" />
    <input v-model="speaker" placeholder="Enter speaker URL" />
    <input type="file" @change="handleFileUpload" accept=".wav, .m4a" />
    <button @click="prediction" :disabled="isLoading">Generate Prediction</button>
    <p v-if="isLoading">Loading...</p>
    <p v-if="message">{{ message }}</p>
    <audio v-if="outputAudio" :src="outputAudio" controls></audio>
    <p v-if="output">{{ output }}</p>
  </div>
</template>

<script>
import { AI, AIOptions, User } from 'aonweb'

export default {
  data() {
    return {
      isLoading: false,
      message: '',
      outputAudio: null,
      output: null,
      text: '',
      speaker: '',
      file: null // 新增的 data 属性，用于存储用户上传的文件
    }
  },
  methods: {
    sleep(ms) {
      return new Promise(resolve => setTimeout(resolve, ms));
    },
    async handleFileUpload(event) {
      const file = event.target.files[0];
      if (!file) {
        return;
      }

      const formData = new FormData();
      formData.append('file', file);

      try {
        const data = await this.uploadFile(formData);
        if (data && data.code === 200 && data.data) {
          this.speaker = data.data; // 将返回的 URL 填充到 speaker 的输入框中
          this.message = 'File uploaded successfully!';
        } else {
          this.message = 'Failed to upload file. Please try again.';
        }
      } catch (error) {
        console.error('Error in file upload:', error);
        this.message = 'An error occurred during file upload. Please try again.';
      }
    },
    async uploadFile(formData) {
      const response = await fetch('https://tmp-file.aigic.ai/api/v1/upload?expires=1800&type=audio/wav', {
        method: 'POST',
        body: formData
      });

      const data = await response.json();
      return data;
    },
    async prediction() {
      this.isLoading = true;
      this.message = '';
      this.outputAudio = null;
      this.output = null;

      let user = new User()
      let is_login = await user.islogin()
      
      if (!is_login) {
        for (let i = 0; i < 10; i++) {
          let result = await user.getOwnedUsers()
          let userid = result && result._userIds && result._userIds.length && result._userIds[0]
          if (userid && userid.length) {
            break
          }
          await this.sleep(300)       
        }
        is_login = await user.islogin()
        if (!is_login) {
          this.message = "Login failed, please try again later";
          this.isLoading = false;
          return
        }
      }

      const ai_options = new AIOptions({
        appId: 'REPLACE_APP_ID' // replace with your actual app id
      })
      let price = 10
      const ai = new AI(ai_options)
      
      try {
        let response = await ai.prediction("/predictions/ai/xtts-v2",
        {
          input:{
            "text": this.text, 
            "speaker": this.speaker, 
            "language": "en",
            "cleanup_voice": false
          }
        }, price);

        if (response && response.code == 200 && response.data) {
          response = response.data
        }

        if (response.task.exec_code == 200 && response.task.is_success) {
          console.log("test", response.output);
          this.outputAudio = response.output;
          this.output = JSON.stringify(response);
          this.message = "Audio generated successfully!";
        } else {
          this.message = "Failed to generate audio. Please try again.";
        }
      } catch (error) {
        console.error("Error in prediction:", error);
        this.message = "An error occurred. Please try again.";
      } finally {
        this.isLoading = false;
      }
    }
  }
}
</script>

<style scoped>
button {
  margin-bottom: 10px;
}
audio {
  max-width: 100%;
  height: auto;
}
input {
  display: block;
  margin-bottom: 10px;
  padding: 8px;
  width: calc(100% - 16px);
}
</style>
