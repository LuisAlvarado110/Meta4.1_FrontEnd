<template>
    <div class="btn_google_wrapper">
      <div ref="googleLoginBtn" class="btn_google"></div>
    </div>
  </template>
  
  <style scoped>
  .btn_google_wrapper {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100%;
    width: 100%; 
  }
  
  </style>
  

<script setup>
import { ref, onMounted } from 'vue'

const googleLoginBtn = ref(null);
onMounted(() => {
    console.log(import.meta.env.VITE_APP_ID_CLIENT_GOOGLE);
    window.google.accounts.id.initialize({
        client_id: import.meta.env.VITE_APP_ID_CLIENT_GOOGLE,
        callback: onSuccess,
        context: 'signin',
        auto_select: false,
        referrerPolicy: {
            policy: 'strict-origin-when-cross-origin'
        }
    });
    window.google.accounts.id.renderButton(
        googleLoginBtn.value, {
        text: 'signin_with', // or 'signup_with' | 'continue_with' | 'signin'
        size: 'large', // or 'small' | 'medium'
        width: '400', // max width 400
        theme: 'outline', // or 'filled_black' |  'filled_blue'
        logo_alignment: 'center' // or 'center'
    });
})
function onSuccess(response) {
    console.log(response.credential);
}

</script>