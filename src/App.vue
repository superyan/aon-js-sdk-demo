<script setup>
import Loading from './components/Loading.vue'
import { AI, AIOptions, User } from 'aonweb'
import { ref } from 'vue'


const audioUrl = ref('')
const showLoading = ref(false);
const logOutput = ref('');  // 用于保存 console.log 输出的内容


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
            console.log("login failed,please try again later");
            showLoading.value = false
            return
        }
    }
    const ai_options = new AIOptions({
        appId: 'k3ebyfaSz8b87xJb_VyEGXx_AJ0MM8ngqU7Ym3AKeW8A' //replace app id
    })
  
    const aonet = new AI(ai_options)
    let price = 10
    console.log("Before prediction call");
    let response = await aonet.prediction("/predictions/ai/xtts-v2",
    {
        input:{
          "text": "Hi there, I'm your new voice clone. Try your best to upload quality audio",
          "speaker": "https://aonet.ai/pbxt/Jt79w0xsT64R1JsiJ0LQRL8UcWspg5J4RFrU6YwEKpOT1ukS/male.wav",
          "language": "en",
          "cleanup_voice": false
        }
    }, price);
    showLoading.value = false
    logOutput.value = JSON.stringify(response, null, 2);  // 将响应内容保存到 logOutput
    console.log("test", response);
    if (response && response.code == 200 && response.data) {
        response = response.data
    }
    if (response.task.exec_code == 200 && response.task.is_success) {
        audioUrl.value = response.output
    }
  } catch (error) {
    console.log("prediction error = ", error)
    logOutput.value = `prediction error = ${error}`;  // 将错误内容保存到 logOutput
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
        <audio v-if="audioUrl" controls :src="audioUrl"></audio> <!-- 播放生成的音频 -->
			</div>
      <div class="log-output">
        <pre>{{ logOutput }}</pre>  <!-- 打印 console.log 输出的内容 -->
      </div>
		</div>
	</div>
</template>

<style scoped>

.container {
	padding: 0 6.4vw 18.4vw;
	margin: 0;
}

.log-output {
  white-space: pre-wrap;
  word-wrap: break-word;
  background: #e0e0e0;
  padding: 10px;
  border-radius: 5px;
  margin-top: 20px;
}

</style>
