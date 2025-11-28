<template>
  <q-layout view="lHh Lpr fFf" class="app-layout">
    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script setup>
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Patrick+Hand&family=Shadows+Into+Light&display=swap');

html, body, #app {
  margin: 0;
  padding: 0;
  height: 100%;
}

.app-layout {
  background: repeating-linear-gradient(
    white,
    white 28px,
    #d6d6d6 29px
  );
  font-family: "Patrick Hand", cursive;
}

body {
  overflow-x: hidden;
}
</style>
