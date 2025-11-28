<template>
  <div class="page-juego">
    <div v-if="mostrarModal" class="modal-overlay">
      <div class="modal">
        <h2>Ingresa tu nombre</h2>
        <input 
          v-model="nombreUsuario" 
          type="text" 
          placeholder="Tu nombre" 
          maxlength="20"
          @keyup.enter="confirmarNombre"
        />
        <p v-if="errorNombre" class="error">{{ errorNombre }}</p>
        <div class="modal-buttons">
          <button @click="confirmarNombre" class="btn-confirmar">Jugar</button>
        </div>
      </div>
    </div>

    <div v-else>
      <div class="cronometro">
        <div class="tiempo" :class="{ 'tiempo-bajo': tiempoRestante <= 30 }">
          {{ formatearTiempo(tiempoRestante) }}
        </div>
      </div>

      <div class="dibujo">
        <svg width="160" height="200">
          <line x1="20" y1="180" x2="140" y2="180" stroke="#1c1c1c" stroke-width="3"/>
          <line x1="80" y1="20" x2="80" y2="180" stroke="#1c1c1c" stroke-width="3"/>
          <line x1="80" y1="20" x2="140" y2="20" stroke="#1c1c1c" stroke-width="3"/>
          <line x1="140" y1="20" x2="140" y2="50" stroke="#1c1c1c" stroke-width="3"/>

          <circle v-if="errores >= 1" cx="140" cy="65" r="15" stroke="#1c1c1c" fill="none" stroke-width="3"/>
          <line v-if="errores >= 2" x1="140" y1="80" x2="140" y2="120" stroke="#1c1c1c" stroke-width="3"/>
          <line v-if="errores >= 3" x1="140" y1="90" x2="160" y2="110" stroke="#1c1c1c" stroke-width="3"/>
          <line v-if="errores >= 4" x1="140" y1="90" x2="120" y2="110" stroke="#1c1c1c" stroke-width="3"/>
          <line v-if="errores >= 5" x1="140" y1="120" x2="160" y2="150" stroke="#1c1c1c" stroke-width="3"/>
          <line v-if="errores >= 6" x1="140" y1="120" x2="120" y2="150" stroke="#1c1c1c" stroke-width="3"/>
        </svg>
      </div>

      <div class="palabra">
        <span v-for="(l, i) in palabra" :key="i">
          {{ letrasUsadas.includes(l) ? l : "_" }}
        </span>
      </div>

      <div class="teclado">
        <button
          v-for="l in abecedario"
          :key="l"
          class="btn-letra"
          :disabled="letrasUsadas.includes(l)"
          @click="usarLetra(l)"
        >
          {{ l }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onUnmounted } from "vue";
import { useRoute, useRouter } from "vue-router";

const router = useRouter();
const route = useRoute();

const mostrarModal = ref(true);
const nombreUsuario = ref("");
const errorNombre = ref("");

const tiempoRestante = ref(0);
const tiempoInicio = ref(0);
const intervalo = ref(null);

const obtenerNombresUsados = () => {
  const scores = JSON.parse(localStorage.getItem('ahorcadoScores') || '[]');
  return scores.map(score => score.nombre.toLowerCase());
};

const confirmarNombre = () => {
  const nombre = nombreUsuario.value.trim();
  
  if (!nombre) {
    errorNombre.value = "Por favor ingresa un nombre";
    return;
  }
  
  const nombresUsados = obtenerNombresUsados();
  
  if (nombresUsados.includes(nombre.toLowerCase())) {
    errorNombre.value = "Este nombre ya está en uso. Por favor elige otro.";
    return;
  }
  
  if (nombre.length < 2) {
    errorNombre.value = "El nombre debe tener al menos 2 caracteres";
    return;
  }
  
  mostrarModal.value = false;
  iniciarCronometro();
};

const obtenerTiempoPorDificultad = () => {
  const dificultad = route.query.dif || "facil";
  const tiempos = {
    facil: 6 * 60,    
    media: 4 * 60,    
    dificil: 2 * 60  
  };
  return tiempos[dificultad] || tiempos.facil;
};

const iniciarCronometro = () => {
  tiempoRestante.value = obtenerTiempoPorDificultad();
  tiempoInicio.value = Date.now();
  
  intervalo.value = setInterval(() => {
    tiempoRestante.value--;
    
    if (tiempoRestante.value <= 0) {
      clearInterval(intervalo.value);
      guardarScore('perdio');
      router.push(`/scores?estado=perdio&palabra=${palabra}&cat=${cat}&dif=${route.query.dif || 'facil'}`);
    }
  }, 1000);
};

const formatearTiempo = (segundos) => {
  const minutos = Math.floor(segundos / 60);
  const segs = segundos % 60;
  return `${minutos.toString().padStart(2, '0')}:${segs.toString().padStart(2, '0')}`;
};

const calcularTiempoTranscurrido = () => {
  const tiempoFinal = Date.now();
  const segundosTranscurridos = Math.floor((tiempoFinal - tiempoInicio.value) / 1000);
  const tiempoMaximo = obtenerTiempoPorDificultad();
  return Math.min(segundosTranscurridos, tiempoMaximo);
};

const palabras = {
  animales: ["GATO", "PERRO", "TIBURON", "ELEFANTE", "LEON"],
  frutas: ["MANGO", "SANDIA", "PERA", "NARANJA", "FRESA"],
  paises: ["COLOMBIA", "ARGENTINA", "MEXICO", "BRASIL", "CHILE"],
  deportes: ["FUTBOL", "TENIS", "NATACION", "CICLISMO", "ATLETISMO"],
  peliculas: ["AVATAR", "TITANIC", "STARWARS", "HARRYPOTTER", "MATRIX"],
  ciencia: ["ATOMO", "GENETICA", "FISICA", "QUIMICA", "BIOLOGIA"]
};

const cat = route.query.cat || "animales";

const palabra = palabras[cat][Math.floor(Math.random() * palabras[cat].length)];

const letrasUsadas = ref([]);
const errores = ref(0);

const abecedario = "ABCDEFGHIJKLMNÑOPQRSTUVWXYZ".split("");

function usarLetra(l) {
  letrasUsadas.value.push(l);

  if (!palabra.includes(l)) {
    errores.value++;

    if (errores.value >= 6) {
      const tiempoUsado = calcularTiempoTranscurrido();
      guardarScore('perdio', tiempoUsado);
      clearInterval(intervalo.value);
      router.push(`/scores?estado=perdio&palabra=${palabra}&cat=${cat}&dif=${route.query.dif || 'facil'}`);
    }
  } else {
    const gano = palabra.split("").every(x => letrasUsadas.value.includes(x));
    if (gano) {
      const tiempoUsado = calcularTiempoTranscurrido();
      guardarScore('ganó', tiempoUsado);
      clearInterval(intervalo.value);
      router.push(`/scores?estado=ganó&palabra=${palabra}&cat=${cat}&dif=${route.query.dif || 'facil'}`);
    }
  }
}

function guardarScore(estado, tiempoUsado) {
  const score = {
    nombre: nombreUsuario.value,
    resultado: estado,
    palabra: palabra,
    categoria: cat,
    dificultad: route.query.dif || "facil",
    tiempo: tiempoUsado,
    fecha: new Date().toISOString()
  };
  
  const scores = JSON.parse(localStorage.getItem('ahorcadoScores') || '[]');
  
  scores.push(score);
  
  localStorage.setItem('ahorcadoScores', JSON.stringify(scores));
}

onUnmounted(() => {
  if (intervalo.value) {
    clearInterval(intervalo.value);
  }
});
</script>

<style scoped>
.page-juego {
  text-align: center;
  padding-top: 40px;
}

.cronometro {
  margin-bottom: 20px;
}

.tiempo {
  font-size: 24px;
  font-weight: bold;
  color: #2c3e50;
  background: #ecf0f1;
  padding: 10px 20px;
  border-radius: 25px;
  display: inline-block;
  min-width: 100px;
}

.tiempo-bajo {
  color: #e74c3c;
  background: #ffebee;
  animation: pulse 1s infinite;
}

@keyframes pulse {
  0% { transform: scale(1); }
  50% { transform: scale(1.05); }
  100% { transform: scale(1); }
}

.dibujo {
  margin-bottom: 30px;
}

.palabra {
  font-size: 40px;
  letter-spacing: 8px;
  margin-bottom: 30px;
}

.teclado {
  max-width: 350px;
  margin: auto;
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 8px;
}

.btn-letra {
  padding: 10px;
  border: none;
  background: #1e1e1e;
  color: white;
  border-radius: 6px;
  cursor: pointer;
  transition: 0.2s;
}

.btn-letra:disabled {
  opacity: 0.3;
  cursor: default;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal {
  background: white;
  padding: 30px;
  border-radius: 10px;
  width: 90%;
  max-width: 400px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
}

.modal h2 {
  margin-top: 0;
  color: #333;
}

.modal input {
  width: 100%;
  padding: 12px;
  margin: 15px 0;
  border: 2px solid #ddd;
  border-radius: 6px;
  font-size: 16px;
  box-sizing: border-box;
}

.modal input:focus {
  border-color: #4a90e2;
  outline: none;
}

.error {
  color: #e74c3c;
  margin: 10px 0;
  font-size: 14px;
}

.modal-buttons {
  display: flex;
  justify-content: center;
  margin-top: 20px;
}

.btn-confirmar {
  background: #4a90e2;
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 16px;
  transition: background 0.2s;
}

.btn-confirmar:hover {
  background: #357ae8;
}
</style>