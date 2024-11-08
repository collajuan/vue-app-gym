<script setup>
import { ref, watch, computed } from "vue";

const ejercicio = ref("");
const descripcion = ref("");
const intensidad = ref("");
const tiempo = ref("");
const filtroTiempo = ref("");
const filtroIntensidad = ref("")

const showFiltros = ref(false);
const showTime = ref(false);
const showModal = ref(false);
const editingCard = ref(null)

// Estados para los mensajes de error
const formSubmitted = ref(false);
const errors = ref({
  titulo: "",
  descripcion: "",
  campos: ""
});

function validateForm() {
  if (!formSubmitted.value) return true;

  errors.value = { titulo: "", descripcion: "", campos: "" };

  if (!ejercicio.value) errors.value.titulo = "Por favor, elige un ejercicio";
  if (!descripcion.value || descripcion.value.length < 5 || descripcion.value.length > 73)
    errors.value.descripcion = "La descripción tiene que tener entre 5 y 74 caracteres";
  if (!intensidad.value || !tiempo.value)
    errors.value.campos = "Por favor, completa todos los campos";

  return !errors.value.titulo && !errors.value.descripcion && !errors.value.campos;
}

function toggleFiltros(tipo) {
  if (tipo === "intensidad") {
    showFiltros.value = !showFiltros.value;
    showTime.value = false;
  
  if (showFiltros.value) {
      filtroTiempo.value = "";
    }
  } else if (tipo === "tiempo") {
    showTime.value = !showTime.value;
    showFiltros.value = false;
   
  if (showTime.value) {
      filtroIntensidad.value = "";
    }
  }
}

function loadCards() {
  const storedData = localStorage.getItem("ejercicio");
  return storedData ? JSON.parse(storedData) : [];
}

function saveCards(newCards) {
  localStorage.setItem("ejercicio", JSON.stringify(newCards));
}

const cards = ref(loadCards());

watch(cards, saveCards, { deep: true });


function modalIntensidad(value) {
  intensidad.value = value;
}

function setIntensidad(value) {
  filtroIntensidad.value = value;
}

function setTiempo(value) {
  filtroTiempo.value = value;
}

function openAddCardModal() {
  resetForm();
  showModal.value = true;
}

function addCard() {
  formSubmitted.value = true;

  if (!validateForm()) return;

  if (editingCard.value !== null) {
    // Modo edición: busca y actualiza la card existente
    const cardIndex = cards.value.findIndex((card) => card.id === editingCard.value);
    if (cardIndex !== -1) {
      cards.value[cardIndex] = {
        ...cards.value[cardIndex],
        ejercicio: ejercicio.value,
        descripcion: descripcion.value,
        intensidad: intensidad.value,
        tiempo: tiempo.value,
      };
      showModal.value = false;
      resetForm()
    }
  } else {
      cards.value.push({
      id: Math.floor(Math.random() * 1000000),
      ejercicio: ejercicio.value,
      descripcion: descripcion.value,
      intensidad: intensidad.value,
      tiempo: tiempo.value,
  });
      showModal.value = false;
      resetForm()
}
}

function editCard (card) {
  showModal.value = true;
  editingCard.value = card.id
  ejercicio.value = card.ejercicio;
  descripcion.value = card.descripcion,
  intensidad.value = card.intensidad,
  tiempo.value = card.tiempo
};

function deleteCard(value) {
  alert("Vas a borrar este ejercicio"),
  cards.value.splice(value, 1)
}

function resetForm() {
  ejercicio.value = "";
  descripcion.value = "";
  intensidad.value = "";
  tiempo.value = "";
  formSubmitted.value = false; 
  validateForm()
}

// Computada para filtrar las cards
const filteredCards = computed(() => {
  return cards.value.filter(card => {
    const matchesIntensidad = !filtroIntensidad.value || card.intensidad === filtroIntensidad.value;
    const matchesTiempo = !filtroTiempo.value || card.tiempo === filtroTiempo.value;
    return matchesIntensidad && matchesTiempo;
  });
});

// Computada para verificar si `cards` está vacío
  const isCardsEmpty = computed(() => cards.value.length === 0);

</script>

<template>
  <main>
    <div v-if="showModal" class="overlay">
      <div class="modal">
        <p class="exercice">Ejercicio: <input v-model="ejercicio" type="text" /></p>
        <div v-if="errors.titulo" class="error-message">{{ errors.titulo }}</div>
        <textarea
          v-model="descripcion"
          placeholder="Detalles del ejercicio..."
        ></textarea>
        <div v-if="errors.descripcion" class="error-message">{{ errors.descripcion }}</div>
        <div class="intensidad">
          <p>Intensidad:</p>
          <button @click="modalIntensidad('Baja')" :class="['btn-intensidad baja', { active: intensidad === 'Baja' }]">Baja</button>
          <button @click="modalIntensidad('Media')" :class="['btn-intensidad media', { active: intensidad === 'Media' }]">Media</button>
          <button @click="modalIntensidad('Alta')" :class="['btn-intensidad alta', { active: intensidad === 'Alta' }]">Alta</button>
        </div>
        <div class="tiempo">
          <p class="p-tiempo">Duración del ejercicio</p>
          <select v-model="tiempo" >
            <option disabled>Sellecione un intervalo</option>
            <option value="15-30 minutos">15-30 minutos</option>
            <option value="30-45 minutos">30-45 minutos</option>
            <option value="45-60 minutos">45-60 minutos</option>
          </select>
        </div>
        <div v-if="errors.campos" class="error-message">{{ errors.campos }}</div>
        <div class="btn-modal">
          <button @click="addCard" class="btn-add">Añadir</button>
          <button @click="showModal = false" class="btn-close">Cerrar</button>
        </div>
      </div>
    </div>
    <h1>FitMotion</h1>
    <div class="container">
      <div class="portada">
        <h2>Filtros:</h2>
        <button @click="toggleFiltros('intensidad')" class="btn-filtros">
          Intensidad
        </button>
        <div v-if="showFiltros" class="filtros">
          <button @click="setIntensidad('Baja')" :class="['btn-intensidad baja', { active: filtroIntensidad === 'Baja' }]">Baja</button>
          <button @click="setIntensidad('Media')" :class="['btn-intensidad media', { active: filtroIntensidad === 'Media' }]">Media</button>
          <button @click="setIntensidad('Alta')" :class="['btn-intensidad alta', { active: filtroIntensidad === 'Alta' }]">Alta</button>
        </div>
        <button @click="toggleFiltros('tiempo')" class="btn-filtros">
          Tiempo
        </button>
        <div v-if="showTime" class="filtros">
          <button @click="setTiempo('15-30 minutos')" :class="['btn-intensidad baja', { active: filtroTiempo === '15-30 minutos' }]">15-30</button>
          <button @click="setTiempo('30-45 minutos')" :class="['btn-intensidad media', { active: filtroTiempo === '30-45 minutos' }]">30-45</button>
          <button @click="setTiempo('45-60 minutos')" :class="['btn-intensidad alta', { active: filtroTiempo === '45-60 minutos' }]">45-60</button>
        </div>
      </div>
      <div class="agregar">
        <button @click="openAddCardModal" class="btn-agregar">Agregar</button>
      </div>
    </div>

    <div class="cards-container">

      <div v-for="card in filteredCards" :key="card.id" class="cards">
        <div class="card">
          <img
            src="https://canalsalud.imq.es/hubfs/Imported_Blog_Media/crossfit-entrenamiento-1.jpg"
            alt=""
          />
          
          <p class="card-titulo">{{ card.ejercicio }}</p>
          <p class="card-descripcion">{{ card.descripcion }}</p>
          <div class="card-filtros">
            <p :class="`${card.intensidad.toLowerCase()}`">Intensidad: {{ card.intensidad }}</p>
            <p class="card-tiempo">{{ card.tiempo }}</p>
          </div>
          <div class="card-btn">
            <button @click="editCard(card)" class="editar">Editar</button>
            <button @click="deleteCard(card)" class="eliminar">Eliminar</button>
          </div>
        
        </div>
      </div>
    </div>
    <footer :class="{ marginTop: isCardsEmpty }">
      <h4>FitMotion&#169;​</h4>
      <h4>Instagram</h4>
      <h4>Contacto</h4>
    </footer>
  </main>
</template>

<style scoped>
main {
  height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
}
.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  z-index: 10;
}
.modal {
  width: 40%;
  height: 50%;
  background-color: white;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-evenly;
  border-radius: 20px;
  position: fixed;
  top: 28%;
  left: 30%;
}
.exercice {
  font-weight: bold;
  font-size: 20px;
}
input {
  width: 284px;
  font-size: 17px;
}
.modal textarea {
  text-indent: 8px;
  font-size: 14px;
  width: 50%;
  height: 14%;
  resize: none;
  border: 1px solid black;
  border-radius: 10px;
}
.intensidad {
  font-weight: bold;
  font-size: 20px;
  margin: 0;
  display: flex;
  align-items: center;
  gap: 70px;
}
.btn-intensidad {
  height: 28px;
  width: 60px;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  color: #fff;
  opacity: 0.5;
  transition: opacity 0.3s;
}
.active {
  opacity: 1;
}
.baja {
  background-color: green;
}
.media {
  background-color: rgb(255, 102, 0);
}
.alta {
  background-color: red;
}
.tiempo {
  display: flex;
  align-items: center;
  gap: 100px;
  font-weight: bold;
  font-size: 20px;
}
select {
  height: 24px;
  width: 160px;
}
.btn-modal {
  display: flex;
  gap: 150px;
  margin: 20px;
}
.btn-add,
.btn-close {
  font-size: 18px;
  border: none;
  width: 100px;
  height: 44px;
  background-color: rgba(0, 106, 255, 0.6);
  border-radius: 10px;
  cursor: pointer;
}
.btn-close {
  background-color: rgba(255, 0, 0, 0.9);
  color: #fff;
}
.error-message {
  color: red;
  margin-top: 10px;
}
h1 {
  text-align: center;
  font-size: 60px;
}
h2 {
  font-size: 20px;
  margin-left: 20px;
}
h4 {
  font-size: 20px;
  margin: 40px;
  padding: 10px;
  margin: 100px 0 0 0;
}
button {
  font-weight: bold;
}
.container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 20px;
  border: none;
  border-radius: 20px;
  box-shadow: 10px 10px 10px rgb(0, 0, 0);
  background-color: #fff;
  width: 60%;
}
.portada {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 20px;
}
.btn-agregar,
.btn-filtros {
  font-size: 14px;
  margin-right: 10px;
  border: none;
  border-radius: 20px;
  background-color: #ddd;
  height: 40px;
  width: 90px;
  cursor: pointer;
}
.filtros {
  display: flex;
  gap: 10px;
}
/* .btn-time {
  height: 32px;
  width: 70px;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  border: 1px solid #ddd;
  background-color: #fff;
  opacity: 0.5;
  transition: opacity 0.3s;
} */
.cards-container {
  margin-top: 250px;
  width: 90%;
  height: fit-content;
  background-color: #fff;
  border-radius: 30px;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 40px;
}
.cards {
  margin: 40px 0 40px 0;
}
.card {
  width: 280px;
  height: 280px;
  background-color: #fff;
  border: 2px solid black;
  border-radius: 30px;
}
.card img {
  width: 280px;
  height: 140px;
  border-radius: 30px 30px 0px 0px;
}
.card-titulo {
  text-align: center;
  font-weight: bold;
  margin: 0;
  font-size: 18px;
}
.card-descripcion {
  margin: 0 0 0 5px;
  word-wrap: break-word;
  min-height: 37px;
}
.card-intensidad {
  background-color: red;
}
.card-filtros {
  display: flex;
  justify-content: space-around;
  margin: 0;
  padding: 0;
}
.card-btn {
  display: flex;
  justify-content: center;
}
.editar {
  color: rgb(32, 97, 194);
  border-radius: 8px;
  height: 24px;
  margin-right: 60px;
  cursor: pointer;
}
.eliminar {
  color: red;
  border-radius: 8px;
  height: 24px;
  margin-left: 60px;
  cursor: pointer;
}
footer {
  display: flex;
  gap: 20px;
}
.marginTop {
  margin-top: 300px;
}
@media (max-width: 1200px) {
  main {
  height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  
  }
  .modal {
    height: 80%;
    width: 80%; 
    top: 10%;
    left: 5%;
    padding: 10px;
  }
  .intensidad,
  .tiempo,
  .btn-modal {
    flex-direction: column;
    gap: 10px;
  }
  .intensidad button,
  .tiempo select,
  .btn-modal button {
    width: 100%;
  }
  .portada{
    justify-content: center;
    flex-direction: column;
  }
  .container {
    justify-content: center;
    flex-direction: column;
  }
  h2 {
  margin-left: 0;
  font-size: 18px;
  }
  .btn-agregar,
  .btn-filtros {
  font-size: 12px;
  height: 35px;
  width: 75px;
  }
  .filtros {
  display: flex;
  gap: 10px;
  }
  .btn-agregar {
    margin-bottom: 10px;
  }
  .btn-intensidad {
  height: 24px;
  width: 48px;
  }
  .cards-container {
    margin-top: 100px;
  }
  .card {
  width: 240px;
  height: 280px;
  }
  .card img {
  width: 240px;
  }
  .card-titulo {
  font-size: 16px;
  }
  .card-descripcion {
  font-size: 14px;
  }
  .editar {
  margin-right: 40px;
  }
  .eliminar {
  margin-left: 40px;
  }
}
</style>
