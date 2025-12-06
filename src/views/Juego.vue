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
      <div class="info-dificultad">
        <div class="dificultad-badge" :class="dificultad">
          Dificultad: {{ dificultad }}
        </div>
        <div class="intentos-restantes">
          Intentos: {{ intentosRestantes }}/{{ maxIntentos }}
        </div>
      </div>

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

          <circle v-if="intentosFallidos >= 1" cx="140" cy="65" r="15" stroke="#1c1c1c" fill="none" stroke-width="3"/>
          <line v-if="intentosFallidos >= 2" x1="140" y1="80" x2="140" y2="120" stroke="#1c1c1c" stroke-width="3"/>
          <line v-if="intentosFallidos >= 3" x1="140" y1="90" x2="160" y2="110" stroke="#1c1c1c" stroke-width="3"/>
          <line v-if="intentosFallidos >= 4" x1="140" y1="90" x2="120" y2="110" stroke="#1c1c1c" stroke-width="3"/>
          <line v-if="intentosFallidos >= 5" x1="140" y1="120" x2="160" y2="150" stroke="#1c1c1c" stroke-width="3"/>
          <line v-if="intentosFallidos >= 6" x1="140" y1="120" x2="120" y2="150" stroke="#1c1c1c" stroke-width="3"/>
        </svg>
      </div>

      <div class="palabra">
        <span v-for="(l, i) in palabra" :key="i">
          {{ letrasUsadas.includes(l) ? l : "_" }}
        </span>
      </div>

      <div v-if="dificultad === 'facil' && mostrarPista" class="pista">
        <div class="pista-contenido">
          <span class="pista-icono">💡</span>
          <span class="pista-texto">{{ obtenerPista() }}</span>
        </div>
      </div>

      <div v-else-if="dificultad === 'facil'" class="pista-btn-container">
        <button @click="mostrarPista = true" class="btn-pista">
          💡 Mostrar pista
        </button>
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
const mostrarPista = ref(false);

const dificultad = route.query.dif || "facil";

let maxIntentos;
const intentosRestantes = ref(0);
const intentosFallidos = ref(0);

if (dificultad === 'facil') {
  maxIntentos = 12;
} else if (dificultad === 'media') {
  maxIntentos = 8;
} else {
  maxIntentos = 6;
}
intentosRestantes.value = maxIntentos;

const obtenerPista = () => {
  return pistas[cat]?.[palabra] || "Pista no disponible";
};

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
      router.push(`/scores?estado=perdio&palabra=${palabra}&cat=${cat}&dif=${dificultad}`);
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

const pistas = {
  animales: {
    "GATO": "Animal doméstico que maúlla",
    "PERRO": "Mejor amigo del hombre",
    "TIBURON": "Peligroso depredador marino",
    "ELEFANTE": "Mamífero terrestre más grande",
    "LEON": "Rey de la selva"
  },
  frutas: {
    "MANGO": "Fruta tropical de pulpa jugosa",
    "SANDIA": "Fruta grande y refrescante de verano",
    "PERA": "Fruta de forma similar a una lágrima",
    "NARANJA": "Fruta cítrica de color naranja",
    "FRESA": "Pequeña fruta roja con semillas externas"
  },
  paises: {
    "COLOMBIA": "País sudamericano conocido por su café",
    "ARGENTINA": "País del tango y el asado",
    "MEXICO": "País norteamericano de los tacos",
    "BRASIL": "País del carnaval y fútbol",
    "CHILE": "País largo y angosto de Sudamérica"
  },
  deportes: {
    "FUTBOL": "Deporte con balón y 11 jugadores por equipo",
    "TENIS": "Deporte con raqueta y pelota amarilla",
    "NATACION": "Deporte acuático en piscina",
    "CICLISMO": "Deporte con bicicleta",
    "ATLETISMO": "Deporte que incluye correr, saltar y lanzar"
  },
  peliculas: {
    "AVATAR": "Película de ciencia ficción con humanos azules",
    "TITANIC": "Película romántica sobre un barco hundido",
    "STARWARS": "Saga de ciencia ficción con la Fuerza",
    "HARRYPOTTER": "Película sobre un joven mago",
    "MATRIX": "Película de Keanu Reeves sobre realidad virtual"
  },
  ciencia: {
    "ATOMO": "Unidad básica de la materia",
    "GENETICA": "Ciencia que estudia la herencia",
    "FISICA": "Ciencia que estudia la materia y energía",
    "QUIMICA": "Ciencia que estudia las sustancias",
    "BIOLOGIA": "Ciencia que estudia los seres vivos"
  }
};

const cat = route.query.cat || "animales";
const palabra = palabras[cat][Math.floor(Math.random() * palabras[cat].length)];
const letrasUsadas = ref([]);
const abecedario = "ABCDEFGHIJKLMNÑOPQRSTUVWXYZ".split("");

function usarLetra(l) {
  letrasUsadas.value.push(l);

  if (!palabra.includes(l)) {
    intentosRestantes.value--;
    intentosFallidos.value++;
    
    if (intentosRestantes.value <= 0) {
      const tiempoUsado = calcularTiempoTranscurrido();
      guardarScore('perdio', tiempoUsado);
      clearInterval(intervalo.value);
      router.push(`/scores?estado=perdio&palabra=${palabra}&cat=${cat}&dif=${dificultad}`);
    }
  } else {
    const gano = palabra.split("").every(x => letrasUsadas.value.includes(x));
    if (gano) {
      const tiempoUsado = calcularTiempoTranscurrido();
      guardarScore('ganó', tiempoUsado);
      clearInterval(intervalo.value);
      router.push(`/scores?estado=ganó&palabra=${palabra}&cat=${cat}&dif=${dificultad}`);
    }
  }
}

function guardarScore(estado, tiempoUsado) {
  const score = {
    nombre: nombreUsuario.value,
    resultado: estado,
    palabra: palabra,
    categoria: cat,
    dificultad: dificultad,
    tiempo: tiempoUsado,
    intentosRestantes: intentosRestantes.value,
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

.info-dificultad {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 30px;
  margin-bottom: 20px;
}

.dificultad-badge {
  padding: 8px 20px;
  border-radius: 20px;
  font-weight: bold;
  font-size: 26px;
  color: white;
}

.dificultad-badge.facil {
  background: #2ecc71;
}

.dificultad-badge.media {
  background: #f39c12;
}

.dificultad-badge.dificil {
  background: #e74c3c;
}

.intentos-restantes {
  background: #ecf0f1;
  padding: 8px 20px;
  border-radius: 20px;
  font-weight: bold;
  font-size: 26px;
  color: #2c3e50;
  border: 2px solid #bdc3c7;
}

.cronometro {
  margin-bottom: 80px;
}

.tiempo {
  font-size: 50px;
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
  transform: scale(1.6);
  transform-origin: center;
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

.pista {
  background: linear-gradient(135deg, #e3f2fd, #bbdefb);
  border-left: 4px solid #2196f3;
  padding: 12px 20px;
  margin: 20px auto 30px;
  border-radius: 8px;
  max-width: 500px;
  animation: fadeIn 0.5s ease;
}

.pista-contenido {
  display: flex;
  align-items: center;
  gap: 12px;
}

.pista-icono {
  font-size: 24px;
  flex-shrink: 0;
}

.pista-texto {
  font-size: 20px;
  color: #1565c0;
  font-weight: 500;
  text-align: left;
}

.pista-btn-container {
  margin: 20px auto 30px;
}

.btn-pista {
  background: linear-gradient(135deg, #4a90e2, #357ae8);
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 25px;
  cursor: pointer;
  font-size: 15px;
  font-weight: 500;
  display: inline-flex;
  align-items: center;
  gap: 8px;
  transition: all 0.3s ease;
  box-shadow: 0 4px 12px rgba(52, 152, 219, 0.3);
}

.btn-pista:hover {
  background: linear-gradient(135deg, #357ae8, #2a65cc);
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(52, 152, 219, 0.4);
}

.btn-pista:active {
  transform: translateY(0);
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
