<template>
  <div class="page">
    <h1 class="titulo">Elige la Dificultad</h1>

    <div class="categoria-info" v-if="categoriaActual">
      <p>Categoría: <strong>{{ formatearCategoria(categoriaActual) }}</strong></p>
    </div>

    <div class="opciones">
      <CustomButton 
        label="Fácil - 6 minutos"
        @click="seleccionarDificultad('facil')"
      />
      <CustomButton 
        label="Media - 4 minutos" 
        @click="seleccionarDificultad('media')"
      />
      <CustomButton 
        label="Difícil - 2 minutos"
        @click="seleccionarDificultad('dificil')"
      />
    </div>

    <div class="volver">
      <CustomButton 
        label="Volver a Categorías"
        @click="router.push('/categoria')"
        secondary
      />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { useRoute, useRouter } from "vue-router";
import CustomButton from "../components/CustomButton.vue";

const route = useRoute();
const router = useRouter();

const categoriaActual = ref("");

onMounted(() => {
  categoriaActual.value = route.query.cat || "animales";
  console.log("Categoría recibida:", categoriaActual.value);
});

const formatearCategoria = (cat) => {
  const categorias = {
    animales: "🐾 Animales",
    frutas: "🍎 Frutas", 
    paises: "🌎 Países",
    deportes: "⚽ Deportes",
    peliculas: "🎬 Películas",  
    ciencia: "🔬 Ciencia"      
  };
  return categorias[cat] || cat;
};

function seleccionarDificultad(dificultad) {
  router.push(`/juego?cat=${categoriaActual.value}&dif=${dificultad}`);
}
</script>

<style scoped>
.page {
  text-align: center;
  padding-top: 60px;
}

.titulo {
  font-size: 36px;
  margin-bottom: 30px;
}

.categoria-info {
  margin-bottom: 30px;
  padding: 15px;
  background: #e8f4ff;
  border-radius: 10px;
  display: inline-block;
}

.categoria-info p {
  margin: 0;
  font-size: 18px;
  color: #2c3e50;
}

.opciones {
  display: flex;
  flex-direction: column;
  gap: 18px;
  width: 280px;
  margin: 30px auto;
}

.volver {
  margin-top: 30px;
}
</style>
