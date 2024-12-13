<template>
  <v-layout class="rounded rounded-md">
    <v-app-bar>
      <v-btn icon @click="goToHome">
        <v-icon>mdi-arrow-left</v-icon>
      </v-btn>
      <v-spacer></v-spacer>
      <div class="text-center">Meta 4.1 - Integración Frontend-Backend</div>
      <v-spacer></v-spacer>
    </v-app-bar>

    <v-navigation-drawer>
      <v-list>
        <v-list-item prepend-icon="mdi-account-school" to="/TablaEstudiante" title="Estudiantes"></v-list-item>
        <v-list-item prepend-icon="mdi-human-male-board" to="/TablaProfesores" title="Profesores"></v-list-item>
        <v-list-item prepend-icon="mdi-book-variant" to="/TablaCursos" title="Cursos"></v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-main class="d-flex align-center justify-center" style="min-height: 300px;">
      <router-view />
    </v-main>
  </v-layout>
</template>

<script setup>
import { useRouter } from 'vue-router';

const router = useRouter();

function goToHome() {
  router.push('/');
}
</script>
