<script setup>
import Loading from './components/Loading.vue'
import { AI, AIOptions, User } from 'aonweb'
import { ref } from 'vue'

const imageUrl = ref('')
const showLoading = ref(false)
const responseData = ref(null) // 用于存储整个 response 的 JSON 数据

function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

const prediction = async () => {
  try {
    showLoading.value = true
    let user = new User()
    let is_login = await user.islogin()
    if (!is_login) {
      for (let i = 0; i < 10; i++) {
        let result = await user.getOwnedUsers()
        let userid = result && result._userIds && result._userIds.length && result._userIds[0]
        if (userid && userid.length) {
          break
        }
        await sleep(300)
      }
      is_login = await user.islogin()
      if (!is_login) {
        console.log("login failed, please try again later");
        showLoading.value = false
        return
      }
    }
    const ai_options = new AIOptions({
      appId: 'k3ebyfaSz8b87xJb_VyEGXx_AJ0MM8ngqU7Ym3AKeW8A' //replace app id
    })

    let price = 1
    const ai = new AI(ai_options)
    let voiceResponse = await ai.prediction("/predictions/ai/xtts-v2", {
        input: {
            "text": "Hi there, I'm your new voice clone. Try your best to upload quality audio",
            "speaker": "https://aonet.ai/pbxt/Jt79w0xsT64R1JsiJ0LQRL8UcWspg5J4RFrU6YwEKpOT1ukS/male.wav",
            "language": "en",
            "cleanup_voice": false
        }
    }, price);

    showLoading.value = false
    if (response && response.code == 200 && response.data) {
      response = response.data
    }
    if (response.task.exec_code == 200 && response.task.is_success) {
      console.log("test", response.output);
      responseData.value = response // 存储整个 response 的 JSON 数据
      let url = response.output
      if (Array.isArray(response.output)) {
        url = response.output && response.output.length && response.output[0]
      }
      if (typeof url === 'object') {
        return
      }
      imageUrl.value = url
    }
  } catch (error) {
    console.log("prediction error = ", error)
    showLoading.value = false
  }
}

</script>

<template>
  <Loading v-if="showLoading" :showLoading="showLoading" />
  <div>
    <!-- 页面内容 -->
    <div class="">
      <button class="" @click="prediction">
        <text>生成</text>
      </button>
      <div class="uni-form-item uni-column">
      </div>
      <div v-if="responseData" class="json-output">
        <pre>{{ JSON.stringify(responseData, null, 2) }}</pre>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  padding: 0 6.4vw 18.4vw;
  margin: 0;
}
.json-output {
  margin-top: 20px;
  white-space: pre-wrap;
  word-break: break-all;
  background-color: #f0f0f0;
  padding: 10px;
  border-radius: 5px;
}
</style>
