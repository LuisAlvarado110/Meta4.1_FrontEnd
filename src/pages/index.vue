<template>
  <div class="d-flex justify-center align-center" style="min-height: 100vh;">
    <btn-google-login @success="handleGoogleAuthSuccess" />
  </div>
</template>

<script setup>
import { useRouter } from 'vue-router';
import BtnGoogleLogin from '@/components/BtnGoogleLogin.vue'; 
const router = useRouter();

function handleGoogleAuthSuccess() {
  // Redirigir a la página de tablas tras autenticación exitosa
  router.push('/TablaEstudiante');
}
</script>

<style scoped>

</style>
