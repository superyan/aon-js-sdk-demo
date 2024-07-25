<template>
    <div>
      <input v-model="sourceImage" placeholder="Enter source image URL" />
      <input type="file" @change="handleImageUpload" accept="image/*" />
      <input v-model="drivenAudio" placeholder="Enter driven audio URL" />
      <input type="file" @change="handleAudioUpload" accept=".wav, .m4a" />
      <button @click="generateDigitalHuman" :disabled="isLoading">Generate Digital Human</button>
      <p v-if="isLoading">Loading...</p>
      <p v-if="message">{{ message }}</p>
      <video v-if="outputVideo" :src="outputVideo" controls></video>
      <p v-if="output">{{ output }}</p>
      <pre v-if="error">{{ error }}</pre>
    </div>
  </template>
  
  <script>
  import { AI, AIOptions, User } from 'aonweb'
  
  export default {
    data() {
      return {
        isLoading: false,
        message: '',
        outputVideo: null,
        output: null,
        sourceImage: '',
        drivenAudio: '',
        error: '' // To store error details
      }
    },
    methods: {
      sleep(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
      },
      async handleImageUpload(event) {
        const file = event.target.files[0];
        if (!file) {
          return;
        }
  
        const formData = new FormData();
        formData.append('file', file);
  
        try {
          const data = await this.uploadFile(formData, 'image');
          if (data && data.code === 200 && data.data) {
            this.sourceImage = data.data;
            this.message = 'Image uploaded successfully!';
          } else {
            this.message = 'Failed to upload image. Please try again.';
          }
        } catch (error) {
          console.error('Error in image upload:', error);
          this.message = 'An error occurred during image upload. Please try again.';
          this.error = JSON.stringify(error, null, 2); // Store error details
        }
      },
      async handleAudioUpload(event) {
        const file = event.target.files[0];
        if (!file) {
          return;
        }
  
        const formData = new FormData();
        formData.append('file', file);
  
        try {
          const data = await this.uploadFile(formData, 'audio');
          if (data && data.code === 200 && data.data) {
            this.drivenAudio = data.data;
            this.message = 'Audio uploaded successfully!';
          } else {
            this.message = 'Failed to upload audio. Please try again.';
          }
        } catch (error) {
          console.error('Error in audio upload:', error);
          this.message = 'An error occurred during audio upload. Please try again.';
          this.error = JSON.stringify(error, null, 2); // Store error details
        }
      },
      async uploadFile(formData, fileType) {
        const response = await fetch(`https://tmp-file.aigic.ai/api/v1/upload?expires=1800&type=${fileType}`, {
          method: 'POST',
          body: formData
        });
  
        const data = await response.json();
        return data;
      },
      async generateDigitalHuman() {
        this.isLoading = true;
        this.message = '';
        this.outputVideo = null;
        this.output = null;
        this.error = ''; // Clear previous error details
  
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
          appId: 'k3ebyfaSz8b87xJb_VyEGXx_AJ0MM8ngqU7Ym3AKeW8A' // replace with your actual app id
        })
        const ai = new AI(ai_options)
  
        try {
          let response = await ai.prediction("/predictions/ai/sadtalker",
          {
            input: {
              "still": true,
              "enhancer": "gfpgan",
              "preprocess": "full",
              "driven_audio": this.drivenAudio,
              "source_image": this.sourceImage
            }
          });
  
          if (response && response.code == 200 && response.data) {
            response = response.data
          }
  
          if (response.task.exec_code == 200 && response.task.is_success) {
            this.outputVideo = response.output;
            this.output = JSON.stringify(response);
            this.message = "Digital human generated successfully!";
          } else {
            this.message = "Failed to generate digital human. Please try again.";
          }
        } catch (error) {
          console.error("Error in prediction:", error);
          this.message = "An error occurred. Please try again.";
          this.error = JSON.stringify(error, null, 2); // Store error details
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
  video {
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
  