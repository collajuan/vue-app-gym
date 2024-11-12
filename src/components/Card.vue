<script>
  export default {
    props: {
      card: Object,
    },
  };
</script>

<template>
    <div class="cards">
        <div class="card">
            <!-- Revisar el src del la imagen -->
            <img src="https://canalsalud.imq.es/hubfs/Imported_Blog_Media/crossfit-entrenamiento-1.jpg" alt="" />
            <p class="card-titulo">{{ card.ejercicio }}</p>
            <p class="card-descripcion">{{ card.descripcion }}</p>
            <div class="card-filtros">
                <p :class="`${card.intensidad.toLowerCase()}`">Intensidad: {{ card.intensidad }}</p>
                <p class="card-tiempo">{{ card.tiempo }}</p>
            </div>
            <div class="card-btn">
                <button @click="$emit('edit', card)" class="editar">Editar</button>
                <button @click="$emit('delete', card)" class="eliminar">Eliminar</button>
            </div>
        </div>
    </div>
</template>
  
  