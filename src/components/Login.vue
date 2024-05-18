<script setup lang="ts">
// import { ref, computed, onMounted } from "vue";
import { generateCodeChallenge, generateCodeVerifier, generateState } from "../utils/pkce"; // PKCEユーティリティ関数をインポート

// TODO 移動
// クライアントID（LINEデベロッパーコンソールから取得）
const clientId = import.meta.env.VITE_LINE_CLIENT_ID;
// リダイレクトURI（LINEデベロッパーコンソールで設定）
const redirectUri = import.meta.env.VITE_LINE_REDIRECT_URI;
const myApiUri = import.meta.env.VITE_MY_API_URI;

// OAuth認証リクエストを送信
async function requestAuthorization() {
  const codeVerifier = generateCodeVerifier();
  const codeChallenge = await generateCodeChallenge(codeVerifier);
  localStorage.setItem('codeVerifier', codeVerifier);

  const state = generateState();
  localStorage.setItem('state', state);
  const scope = "openid profile";

  const authorizationUrl = `https://access.line.me/oauth2/v2.1/authorize?response_type=code&client_id=${clientId}&redirect_uri=${redirectUri}&code_challenge=${codeChallenge}&code_challenge_method=S256&state=${state}&scope=${scope}`;
  window.location.href = authorizationUrl;
}

// アクセストークンを取得
async function getAccessToken(authorizationCode:string) {
  const codeVerifier = localStorage.getItem('codeVerifier');
  const tokenResponse = await fetch(`${myApiUri}`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      code: authorizationCode,
      codeVerifier: codeVerifier,
      redirectUri: redirectUri,
      request_type: "token"
    }),
  });

  const tokenData = await tokenResponse.json();
  console.log(tokenData)
  return tokenData.access_token
}

async function getUser(access_token:string){
  const response = await fetch(myApiUri, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${access_token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        "request_type": "user"
      })
    });
    return response.json()
}

// リダイレクト後のコールバック処理
async function handleCallback() {
  const urlParams = new URLSearchParams(window.location.search);
  const authorizationCode = urlParams.get('code');
  const state = urlParams.get('state');
  if (authorizationCode) {
    console.log('fuga')
    const storedState = localStorage.getItem('state');
    if (state === storedState) {
      console.log('hoge')
      const access_token = await getAccessToken(authorizationCode);
      const user = await getUser(access_token)
      console.log(user)
    
    }
  }
}

// ページ読み込み時にコールバック処理を実行
window.onload = handleCallback;
</script>

<template>
  <div class="card">
    <button  @click="requestAuthorization">ログイン</button>
  </div>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>