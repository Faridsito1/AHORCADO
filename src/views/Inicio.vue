<template>
  <div class="inicio-container">
    <div v-for="(l, i) in letras" :key="i" class="letra" :style="l.style">
      {{ l.char }}
    </div>

    <div class="content">
      <div class="dibujo">
        <svg width="160" height="200">
          <line x1="20" y1="180" x2="140" y2="180" stroke="#1c1c1c" stroke-width="3"/>
          <line x1="80" y1="20" x2="80" y2="180" stroke="#1c1c1c" stroke-width="3"/>
          <line x1="80" y1="20" x2="140" y2="20" stroke="#1c1c1c" stroke-width="3"/>
          <line x1="140" y1="20" x2="140" y2="50" stroke="#1c1c1c" stroke-width="3"/>

          <circle cx="140" cy="65" r="15" stroke="#1c1c1c" fill="none" stroke-width="3"/>
          <line x1="140" y1="80" x2="140" y2="120" stroke="#1c1c1c" stroke-width="3"/>
          <line x1="140" y1="90" x2="160" y2="110" stroke="#1c1c1c" stroke-width="3"/>
          <line x1="140" y1="90" x2="120" y2="110" stroke="#1c1c1c" stroke-width="3"/>
          <line x1="140" y1="120" x2="160" y2="150" stroke="#1c1c1c" stroke-width="3"/>
          <line x1="140" y1="120" x2="120" y2="150" stroke="#1c1c1c" stroke-width="3"/>
        </svg>
      </div>

      <h1 class="titulo">EL AHORCADO</h1>
      <p class="subtitulo">
        Adivina la palabra antes de que sea demasiado tarde…
      </p>

      <CustomButton 
        label="Comenzar"
        @click="irJuego"
      />
    </div>
  </div>
</template>

<script setup>
import { useRouter } from "vue-router";
import CustomButton from "../components/CustomButton.vue";

const router = useRouter();

function irJuego() {
  router.push("/categoria");
}

const letras = Array.from("ABCDEFGHIJKLMNÑOPQRSTUVWXYZ").map(char => ({
  char,
  style: {
    top: Math.random() * 90 + "vh",
    left: Math.random() * 90 + "vw",
    transform: `rotate(${Math.random() * 40 - 20}deg)`
  }
}));
</script>

<style scoped>
.inicio-container {
  position: relative;
  width: 100%;
  height: 100vh;
  background: repeating-linear-gradient(
    white,
    white 28px,
    #d6d6d6 29px
  );
  overflow: hidden;
  font-family: "Patrick Hand", cursive;
}

.letra {
  position: absolute;
  font-size: 32px;
  opacity: 0.25;
  user-select: none;
  pointer-events: none;
  font-weight: bold;
  color: #3c3c3c;
}

.content {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
}

.dibujo {
  margin-bottom: 20px;
}

.titulo {
  font-size: 98px;
  letter-spacing: 6px;
  margin-bottom: 10px;
}

.subtitulo {
  font-size: 18px;
  margin-bottom: 25px;
  opacity: 0.8;
}
</style>
