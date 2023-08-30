<script setup lang="ts">
import { ref, computed, onMounted } from "vue";
import axios from "axios";

defineProps<{ msg: string }>();
// TODO 秘匿情報の扱い
const state = "202308301948";
const client_id = import.meta.env.VITE_CLIENT_ID;
const client_secret = import.meta.env.VITE_CLIENT_SECRET;
const count = ref(0);
const redirect_uri = "http://localhost:5173";
const scope = "openid profile";
const authUrl = computed(
  () =>
    `https://access.line.me/oauth2/v2.1/authorize?response_type=code&client_id=${client_id}&redirect_uri=${redirect_uri}&state=${state}&scope=${scope}&disable_auto_login=true`
);

const getQueryParameterByName = (name) => {
  const url = window.location.href;
  const nameProc = name.replace(/[\[\]]/g, "\\$&");
  const regex = new RegExp("[?&]" + nameProc + "(=([^&#]*)|&|#|$)"),
    results = regex.exec(url);
  if (!results) return null;
  if (!results[2]) return "";
  return decodeURIComponent(results[2].replace(/\+/g, " "));
};

const sendRequest = async () => {
  try {
    console.log("axios");
    window.location.href = authUrl.value;
  } catch (error) {
    console.error("An error occurred:", error);
  }
};

const getUserInfo = async (access_token) => {
  const headers = {
    Authorization: `Bearer ${access_token}`,
  };
  const resp = await axios.get("https://api.line.me/oauth2/v2.1/userinfo", {
    headers,
  });
  console.log("user data");
  // sub/name/picture
  console.log(resp.data);
  // TODO acccess_tokenの保存場所、refresh_token管理
};

onMounted(async () => {
  const code = getQueryParameterByName("code");
  const received_state = getQueryParameterByName("state");
  console.log(`code: ${code}`);
  if (received_state != state) {
    return alert(`received: ${received_state}, ans: ${state}`);
  }
  const tokenUrl = "https://api.line.me/oauth2/v2.1/token";
  const params = {
    grant_type: "authorization_code",
    code: code,
    redirect_uri: redirect_uri,
    client_id: client_id,
    client_secret,
  };
  const headers = {
    "Content-Type": "application/x-www-form-urlencoded",
  };
  const resp = await axios.post(tokenUrl, params, { headers });
  console.log(resp);
  getUserInfo(resp.data.access_token);
});
</script>

<template>
  <h1>{{ msg }}</h1>

  <div class="card">
    <button type="button" @click="count++">count is {{ count }}</button>
    <p>
      Edit
      <code>components/HelloWorld.vue</code> to test HMR
    </p>
  </div>

  <p>
    Check out
    <a href="https://vuejs.org/guide/quick-start.html#local" target="_blank"
      >create-vue</a
    >, the official Vue + Vite starter
  </p>
  <p>
    Install
    <a href="https://github.com/vuejs/language-tools" target="_blank">Volar</a>
    in your IDE for a better DX
  </p>
  <p class="read-the-docs">Click on the Vite and Vue logos to learn more</p>
  <button @click="sendRequest">ログイン</button>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
