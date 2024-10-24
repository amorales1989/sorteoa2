<script setup>
import { ref, computed } from 'vue'

defineProps({
  msg: String,
})

const minNumber = ref(1);  // Número inicial predeterminado
const maxNumber = ref(100);  // Número máximo predeterminado
const numPrizes = ref(0);  // Cantidad predeterminada de premios
const sortFromLast = ref(false); // Controla si el sorteo empieza desde el último premio

const count = ref(0);  // Controla cuál premio estamos sorteando
const randomNumber = ref(0);  // Número objetivo final
const currentNumber = ref(0); // Número que cambia de manera aleatoria
const isGenerating = ref(false);  // Bandera para verificar si se está generando un número
const showWinnerText = ref(false); // Bandera para mostrar el texto "El número ganador es:"
const gameCompleted = ref(false);  // Bandera para indicar que el sorteo ha finalizado

// Generar lista de premios en base al número de premios
const generatedNumbers = computed(() => Array.from({ length: numPrizes.value }, () => null));

// Números que no pueden salir en el sorteo
const invalidNumbers = [];

// Función para generar un número aleatorio que no esté en la lista de números inválidos
const generateValidRandomNumber = () => {
  let number;
  do {
    number = Math.floor(Math.random() * (maxNumber.value - minNumber.value + 1)) + minNumber.value;
  } while (invalidNumbers.includes(number));
  return number;
};

// Función para determinar el índice del premio actual según el valor de sortFromLast
const getCurrentIndex = () => {
  return sortFromLast.value ? numPrizes.value - 1 - count.value : count.value;
};

// Función para generar un número aleatorio con efecto de animación
const getRandomNumber = () => {
  if (isGenerating.value || count.value >= numPrizes.value) return;

  isGenerating.value = true;
  showWinnerText.value = false;
  const targetNumber = generateValidRandomNumber();
  randomNumber.value = targetNumber;
  currentNumber.value = 0;

  const animationDuration = 6000;
  const steps = 50;
  const stepDuration = animationDuration / steps;

  let step = 0;

  const button = document.querySelector('.random-number button');
  button.innerText = 'Buscando al ganador...';
  button.disabled = true;

  const interval = setInterval(() => {
    currentNumber.value = Math.floor(Math.random() * (maxNumber.value - minNumber.value + 1)) + minNumber.value;

    if (step >= steps) {
      clearInterval(interval);
      currentNumber.value = targetNumber;

      generatedNumbers.value[getCurrentIndex()] = targetNumber;
      count.value += 1;

      if (count.value >= numPrizes.value) {
        button.innerText = 'Todos los premios sorteados';
        button.disabled = true;
      } else {
        button.innerText = 'Buscar otro ganador.';
        button.disabled = false;
      }

      isGenerating.value = false;
      showWinnerText.value = true;
      gameCompleted.value = count.value >= numPrizes.value;
    }

    step++;
  }, stepDuration);
};

// Función para repetir el sorteo de un premio específico
const resetDrawForPrize = (index) => {
  if (isGenerating.value || generatedNumbers.value[index] === null) return;

  isGenerating.value = true;
  showWinnerText.value = false;

  const targetNumber = generateValidRandomNumber();
  randomNumber.value = targetNumber;
  currentNumber.value = 0;

  const animationDuration = 6000;
  const steps = 50;
  const stepDuration = animationDuration / steps;

  let step = 0;

  const interval = setInterval(() => {
    currentNumber.value = Math.floor(Math.random() * (maxNumber.value - minNumber.value + 1)) + minNumber.value;

    if (step >= steps) {
      clearInterval(interval);
      currentNumber.value = targetNumber;

      generatedNumbers.value[index] = targetNumber;

      isGenerating.value = false;
      showWinnerText.value = true;
    }

    step++;
  }, stepDuration);
};
</script>


<template>
  <!-- Grilla con los premios -->
  <div class="prize-grid">
    <div v-for="(number, index) in generatedNumbers" :key="index" class="prize">
      <div class="prize-title">{{ index + 1 }}° premio</div>
      <div class="prize-number">{{ number ?? '---' }}</div>
      <button v-if="number" @click="resetDrawForPrize(index)">R</button>
    </div>
  </div>

  <!-- Texto que muestra el número ganador -->
  <div v-if="showWinnerText" class="winner-text">
    <p>El número ganador es:</p>
  </div>
  <div v-else class="winner-text">
    <p>&nbsp;</p>
  </div>

  <!-- Número grande arriba del botón -->
  <div class="number-display">
    <h1>{{ currentNumber }}</h1>
  </div>

  <!-- Botón para generar un número aleatorio -->
  <div class="random-number">
    <button type="button" @click="getRandomNumber">Comenzar</button>
  </div>

  <div class="range-inputs">
    <label>
      Número inicial:
      <input type="number" v-model.number="minNumber" :min="1" :max="maxNumber - 1" />
    </label>
    <label>
      Número máximo:
      <input type="number" v-model.number="maxNumber" :min="minNumber + 1" :max="1000" />
    </label>
    <label>
      Cantidad de premios:
      <input type="number" v-model.number="numPrizes" :min="1" :max="10" />
    </label>
  </div>

  <!-- Interruptor para elegir si sortear desde el primer premio o desde el último -->
  <div class="toggle-sort">
  <label>
    <input 
      type="checkbox" 
      v-model="sortFromLast" 
      :disabled="isGenerating || count > 0" 
    />
    Comenzar desde el último premio
  </label>
</div>
</template>


<style scoped>

.prize-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(190px, 1fr)); 
  grid-gap: 10px;
  margin-bottom: 20px;
  justify-items: center;
  align-items: center;
  text-align: center;
  justify-content: center; 
}



/* Estilos para los premios */
.prize {
  font-size: 1.5rem;
  padding: 10px;
  background-color: #f0f0f0;
  border-radius: 5px;
  border: 1px solid #ddd;
  color: #333;
  min-width: 190px;
  min-height: 140px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

/* Título del premio */
.prize-title {
  font-size: 1.2rem;
  margin-bottom: 10px;
}

/* Número sorteado */
.prize-number {
  font-size: 2.5rem;
  font-weight: bold;
}

/* Estilos para los botones de repetir */
.prize button {
  margin-top: 10px;
  padding: 5px 10px;
  font-size: 1rem;
  cursor: pointer;
}

/* Centrar el número grande en la pantalla */
.number-display {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}

.number-display h1 {
  font-size: 7rem;
  color: #333;
  line-height: 0;
}

/* Estilos para el texto del número ganador */
.winner-text {
  text-align: center;
  margin-top: 20px;
  margin-bottom: 0px;
}

.winner-text p {
  font-size: 2rem;
  color: #333;
  font-weight: bold;
  margin-bottom: 0px;
}

/* Estilos para el botón */
.random-number {
  text-align: center;
  margin-top: 20px;
  display: flex;
  justify-content: center;
}

button {
  padding: 10px 20px;
  font-size: 1rem;
  cursor: pointer;
}

.range-inputs {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
  margin-top: 20px;
  gap: 15px;
}

.range-inputs label {
  font-size: 1rem;
}

.range-inputs input {
  margin-left: 5px;
  padding: 5px;
  font-size: 1rem;
  width: 80px;
}
</style>
