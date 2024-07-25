<template>
  <div>
    <button @click="prediction" :disabled="isLoading">Generate Prediction</button>
    <p v-if="isLoading">Loading...</p>
    <p v-if="message">{{ message }}</p>
    <audio v-if="outputAudio" :src="outputAudio" controls></audio>
    <div v-if="responseData">
      <h3>AI Model Response Data:</h3>
      <pre>{{ JSON.stringify(responseData, null, 2) }}</pre>
    </div>
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
      responseData: null
    }
  },
  methods: {
    sleep(ms) {
      return new Promise(resolve => setTimeout(resolve, ms));
    },
    async prediction() {
      this.isLoading = true;
      this.message = '';
      this.outputAudio = null;
      this.responseData = null;

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
            "text": "Hi there, I'm your new voice clone. Try your best to upload quality audio",
            "speaker": "https://aonet.ai/pbxt/Jt79w0xsT64R1JsiJ0LQRL8UcWspg5J4RFrU6YwEKpOT1ukS/male.wav",
            "language": "en",
            "cleanup_voice": false
          }
        }, price);

        if (response && response.code === 200 && response.data) {
          response = response.data
        }

        if (response.task.exec_code === 200 && response.task.is_success) {
          console.log("Audio URL:", response.output[0]);
          this.outputAudio = response.output[0];  // Ensure this URL points to the audio file
          this.responseData = response;
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
pre {
  background-color: #f8f8f8;
  padding: 10px;
  border-radius: 5px;
  overflow-x: auto;
}
</style>
