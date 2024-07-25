<script setup>
import Loading from './components/Loading.vue'
import { AIOptions, User } from 'aonweb'
import { ref } from 'vue'
import { aonet } from 'aonweb'

const imageUrl = ref('')
const showLoading = ref(false);

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

    const ai = new AI(ai_options)
    let response = await aonet.prediction("/predictions/ai/sadtalker",
    {
        input:{
            "still": true,
            "enhancer": "gfpgan",
            "preprocess": "full",
            "driven_audio": "https://aonet.ai/pbxt/Jf1gczNATWiC94VPrsTTLuXI0ZmtuZ6k0aWBcQpr7VuRc5f3/japanese.wav",
            "source_image": "https://replicate.delivery/pbxt/Jf1gcsODejVsGRd42eeUj0RXX11zjxzHuLuqXmVFwMAi2tZq/art_1.png"
        }
    });

    showLoading.value = false
    if (response && response.code == 200 && response.data) {
        response = response.data
    }
    if (response.task.exec_code == 200 && response.task.is_success) {
        console.log("test", response.output);
        let url = response.output
        if (Array.isArray(response.output)) {
          url = response.output && response.output.length && response.output[0]
        }
        if (typeof url == 'object' || typeof url == 'Object') {
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
        <img class="res_img" :src="imageUrl" mode=""></img> 
			</div>
		</div>
	</div>
</template>

<style scoped>

.container {
	padding: 0 6.4vw 18.4vw;
	margin: 0;
}

</style>
