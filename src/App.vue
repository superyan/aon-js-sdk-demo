<template>
  <div>
    <button @click="prediction" :disabled="isLoading">Generate Prediction</button>
    <p v-if="isLoading">Loading...</p>
    <p v-if="message">{{ message }}</p>
    <img v-if="outputImage" :src="outputImage" alt="Generated Image" />
  </div>
</template>

<script>
import { AI, AIOptions, User } from 'aonweb'

export default {
  data() {
    return {
      isLoading: false,
      message: '',
      outputImage: null
    }
  },
  methods: {
    sleep(ms) {
      return new Promise(resolve => setTimeout(resolve, ms));
    },
    async prediction() {
      this.isLoading = true;
      this.message = '';
      this.outputImage = null;

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
        let response = await ai.prediction("/predictions/ai/stable-diffusion-3",
        {
          input:{
            "prompt": "with smoke, half ice and half fire and ultra realistic in detail.wolf, typography, dark fantasy, wildlife photography, vibrant, cinematic and on a black background",
            "cfg": 3.5,
            "steps": 28,
            "aspect_ratio": "9:16",
            "output_format": "png",
            "output_quality": 90,
            "negative_prompt": "",
            "prompt_strength": 0.85
          }
        }, price);

        if (response && response.code == 200 && response.data) {
          response = response.data
        }

        if (response.task.exec_code == 200 && response.task.is_success) {
          console.log("test", response.output);
          this.outputImage = response.output;
          this.message = "Image generated successfully!";
        } else {
          this.message = "Failed to generate image. Please try again.";
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
img {
  max-width: 100%;
  height: auto;
}
</style>
Made with