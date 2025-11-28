<template>
  <div class="page">
    <h1 class="titulo">Elige una categoría</h1>

    <div class="opciones">
      <CustomButton 
        label="Animales"
        @click="seleccionar('animales')"
      />
      <CustomButton 
        label="Frutas"
        @click="seleccionar('frutas')"
      />
      <CustomButton 
        label="Países"
        @click="seleccionar('paises')"
      />
      <CustomButton 
        label="Deportes"
        @click="seleccionar('deportes')"
      />
      <CustomButton 
        label="Películas"
        @click="seleccionar('peliculas')" 
      />
      <CustomButton 
        label="Ciencia"
        @click="seleccionar('ciencia')"    
      />
    </div>
  </div>
</template>

<script setup>
import { useRouter } from "vue-router";
import CustomButton from "../components/CustomButton.vue";

const router = useRouter();

function seleccionar(cat) {
  router.push(`/dificultad?cat=${cat}`);
}
</script>

<style scoped>
.page {
  text-align: center;
  padding-top: 80px;
}

.titulo {
  font-size: 40px;
  margin-bottom: 40px;
}

.opciones {
  display: flex;
  flex-direction: column;
  gap: 18px;
  width: 250px;
  margin: auto;
}
</style>
