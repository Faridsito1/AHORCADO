<template>
  <div class="page-scores">

    <h2 class="titulo">
      {{ estado === "ganó" ? "¡Felicidades, Ganaste!" : "Game Over" }}
    </h2>

    <div class="resultado">
      <p><strong>Jugador:</strong> {{ nombre }}</p>
      <p><strong>Palabra:</strong> {{ palabra }}</p>
      <p><strong>Resultado:</strong> {{ estado === "ganó" ? "Victoria 🎉" : "Derrota 💀" }}</p>
      <p><strong>Categoría:</strong> {{ formatearCategoria(categoria) }}</p>
      <p><strong>Dificultad:</strong> {{ formatearDificultad(dificultad) }}</p>
      <p><strong>Tiempo:</strong> {{ formatearTiempo(tiempo) }}</p>
    </div>

    <q-btn
      color="primary"
      label="Volver al Inicio"
      class="q-mt-md"
      @click="router.push('/')"
    />

    <h3 class="subtitulo q-mt-xl">
      🏆 Mejores Tiempos - Solo Ganadores
    </h3>

    <div class="filtros">
      <select v-model="filtroCategoria" class="filtro-select">
      <option value="">Todas las categorías</option>
      <option value="animales">Animales</option>
      <option value="frutas">Frutas</option>
      <option value="paises">Países</option>
      <option value="deportes">Deportes</option>
      <option value="peliculas">Películas</option>
      <option value="ciencia">Ciencia</option>
    </select>

      <select v-model="filtroDificultad" class="filtro-select">
        <option value="">Todas las dificultades</option>
        <option value="facil">Fácil</option>
        <option value="media">Media</option>
        <option value="dificil">Difícil</option>
      </select>

      <button @click="limpiarFiltros" class="btn-limpiar">
        Limpiar Filtros
      </button>
    </div>

    <div class="info-filtros" v-if="filtroCategoria || filtroDificultad">
      <p>
        Mostrando: 
        <span v-if="filtroCategoria">{{ formatearCategoria(filtroCategoria) }}</span>
        <span v-if="filtroCategoria && filtroDificultad"> - </span>
        <span v-if="filtroDificultad">{{ formatearDificultad(filtroDificultad) }}</span>
        ({{ scoresFiltrados.length }} resultados)
      </p>
    </div>

    <div class="tabla">
      <table>
        <thead>
          <tr>
            <th>Pos</th>
            <th>Jugador</th>
            <th>Palabra</th>
            <th>Categoría</th>
            <th>Dificultad</th>
            <th>Tiempo</th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="(s, i) in scoresFiltrados" :key="s.fecha + s.nombre" 
              :class="{ 'fila-actual': esPartidaActual(s) }">
            <td>{{ i + 1 }}</td>
            <td>{{ s.nombre }}</td>
            <td>{{ s.palabra }}</td>
            <td class="categoria-cell">{{ formatearCategoria(s.categoria) }}</td>
            <td class="dificultad-cell">{{ formatearDificultad(s.dificultad) }}</td>
            <td class="tiempo-cell">{{ formatearTiempo(s.tiempo) }}</td>
          </tr>
          <tr v-if="scoresFiltrados.length === 0">
            <td colspan="6" class="no-data">
              {{ scoresGanadores.length === 0 ? 'No hay ganadores registrados' : 'No hay ganadores para mostrar con los filtros actuales' }}
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="acciones">
      <q-btn
        color="secondary"
        label="Jugar otra vez"
        class="q-mt-lg"
        @click="volverAJugar"
      />
      
      <q-btn
        color="negative"
        label="Limpiar Historial"
        class="q-mt-lg q-ml-md"
        @click="limpiarScores"
        v-if="scores.length > 0"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import { useRoute, useRouter } from "vue-router";

const route = useRoute();
const router = useRouter();

const estado = route.query.estado || "perdio";
const palabra = route.query.palabra || "";
const categoria = route.query.cat || "animales";
const dificultad = route.query.dif || "facil";
const tiempo = ref(0);
const nombre = ref("");

const filtroCategoria = ref("");
const filtroDificultad = ref("");


const scores = ref([]);

onMounted(() => {
  cargarScores();
  
  if (estado && palabra) {
    guardarPartidaActual();
  }
});

const cargarScores = () => {
  const scoresGuardados = JSON.parse(localStorage.getItem('ahorcadoScores') || '[]');
  scores.value = scoresGuardados;
  
  if (scoresGuardados.length > 0) {
    const partidaActual = scoresGuardados.find(score => 
      score.palabra === palabra && 
      score.categoria === categoria &&
      score.dificultad === dificultad
    );
    
    if (partidaActual) {
      nombre.value = partidaActual.nombre;
      tiempo.value = partidaActual.tiempo;
    } else {
      const ultimoScore = scoresGuardados[scoresGuardados.length - 1];
      nombre.value = ultimoScore.nombre || "Jugador";
      tiempo.value = ultimoScore.tiempo || 0;
    }
  } else {
    nombre.value = "Jugador";
    tiempo.value = 0;
  }
};

const guardarPartidaActual = () => {
  const scoresGuardados = JSON.parse(localStorage.getItem('ahorcadoScores') || '[]');
  
  const partidaExiste = scoresGuardados.some(score => 
    score.palabra === palabra && 
    score.categoria === categoria &&
    score.dificultad === dificultad
  );
  
  if (!partidaExiste && nombre.value) {
    const nuevoScore = {
      nombre: nombre.value,
      resultado: estado,
      palabra: palabra,
      categoria: categoria,
      dificultad: dificultad,
      tiempo: tiempo.value,
      fecha: new Date().toISOString()
    };
    
    scoresGuardados.push(nuevoScore);
    localStorage.setItem('ahorcadoScores', JSON.stringify(scoresGuardados));
    scores.value = scoresGuardados;
  }
};

const scoresGanadores = computed(() => {
  return scores.value.filter(score => score.resultado === "ganó");
});

const scoresFiltrados = computed(() => {
  let filtered = [...scoresGanadores.value];
  
  if (filtroCategoria.value) {
    filtered = filtered.filter(s => s.categoria === filtroCategoria.value);
  }
  
  if (filtroDificultad.value) {
    filtered = filtered.filter(s => s.dificultad === filtroDificultad.value);
  }
  
  return filtered.sort((a, b) => {
    if (a.tiempo !== b.tiempo) {
      return a.tiempo - b.tiempo; 
    }
    return new Date(b.fecha) - new Date(a.fecha); 
  });
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

const formatearDificultad = (dif) => {
  const dificultades = {
    facil: "🟢 Fácil",
    media: "🟡 Media",
    dificil: "🔴 Difícil"
  };
  return dificultades[dif] || dif;
};

const formatearTiempo = (segundos) => {
  if (segundos === undefined || segundos === null) return "0:00";
  const minutos = Math.floor(segundos / 60);
  const segs = segundos % 60;
  return `${minutos}:${segs.toString().padStart(2, '0')}`;
};

const volverAJugar = () => {
  router.push({
    path: '/dificultad',
    query: { cat: categoria }
  });
};

const limpiarScores = () => {
  if (confirm("¿Estás seguro de que quieres borrar todos los puntajes?")) {
    localStorage.removeItem('ahorcadoScores');
    scores.value = [];
    nombre.value = "Jugador";
    tiempo.value = 0;
  }
};

const limpiarFiltros = () => {
  filtroCategoria.value = "";
  filtroDificultad.value = "";
};

const esPartidaActual = (score) => {
  return score.nombre === nombre.value && 
         score.palabra === palabra && 
         score.categoria === categoria &&
         score.dificultad === dificultad &&
         score.resultado === estado;
};
</script>

<style scoped>
.page-scores {
  text-align: center;
  padding: 30px;
  max-width: 1000px;
  margin: auto;
}

.titulo {
  font-size: 32px;
  margin-bottom: 20px;
}

.resultado p {
  font-size: 18px;
  margin: 8px 0;
}

.subtitulo {
  margin-top: 40px;
  font-size: 24px;
  font-weight: bold;
  color: #27ae60; 
}

.filtros {
  margin: 20px 0;
  display: flex;
  gap: 15px;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
}

.filtro-select {
  padding: 8px 12px;
  border: 2px solid #ddd;
  border-radius: 6px;
  background: white;
  font-size: 14px;
  min-width: 150px;
}

.btn-limpiar {
  padding: 8px 16px;
  border: 2px solid #95a5a6;
  border-radius: 6px;
  background: white;
  color: #7f8c8d;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.3s;
}

.btn-limpiar:hover {
  background: #f8f9fa;
  border-color: #7f8c8d;
}

.info-filtros {
  margin: 10px 0;
  padding: 10px;
  background: #e8f4ff;
  border-radius: 6px;
  color: #2c3e50;
}

.tabla {
  margin-top: 20px;
  overflow-x: auto;
  border-radius: 10px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

table {
  width: 100%;
  border-collapse: collapse;
  background: white;
  color: #333;
  border-radius: 10px;
  overflow: hidden;
}

table th, table td {
  padding: 12px;
  border-bottom: 1px solid #eee;
  text-align: center;
}

table th {
  background: #27ae60; 
  color: white;
  font-weight: 600;
}

table tr:hover {
  background: #f5f5f5;
}

.fila-actual {
  background: #e8f4ff !important;
  font-weight: bold;
  border-left: 4px solid #4a90e2;
}

.categoria-cell, .dificultad-cell {
  text-transform: capitalize;
}

.tiempo-cell {
  font-family: 'Courier New', monospace;
  font-weight: bold;
  color: #2c3e50;
}

.no-data {
  text-align: center;
  color: #7f8c8d;
  font-style: italic;
  padding: 30px !important;
}

.acciones {
  display: flex;
  justify-content: center;
  gap: 15px;
  flex-wrap: wrap;
}

.q-mt-md {
  margin-top: 16px;
}

.q-mt-lg {
  margin-top: 24px;
}

.q-mt-xl {
  margin-top: 32px;
}

.q-ml-md {
  margin-left: 16px;
}

@media (max-width: 768px) {
  .filtros {
    flex-direction: column;
  }
  
  .filtro-select, .btn-limpiar {
    width: 100%;
    max-width: 250px;
  }
  
  .acciones {
    flex-direction: column;
    align-items: center;
  }
  
  .q-ml-md {
    margin-left: 0;
  }
}
</style>