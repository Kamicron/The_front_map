<template>
<div id="map-container" @wheel="zoom($event)" @mousedown="startDrag($event)" @mousemove="drag($event); updateCursorPosition($event)" @mouseup="stopDrag" @mouseleave="stopDrag">
  <div id="coordinates-box">
      X: {{ cursorPosition.x }}, Y: {{ cursorPosition.y }}
    </div>  
  <div id="zoom-container" :style="mapStyle">
      <img id="map" src="/map.jpg" alt="Map" />
      <div v-for="point in pointsOfInterest" :key="point.x + '-' + point.y" :style="positionPoint(point)" class="poi">
        <img :src="'/logo/' + pointTypes[point.type][point.subtype]" :alt="point.subtype" />
      </div>
    </div>


  </div>
</template>

<script setup lang='ts'>
import { ref, onMounted, onUnmounted } from 'vue';
const point = ref({ x: 100, y: 200 });
const scale = ref(1);
const position = ref({ x: 0, y: 0 });
const mapStyle = ref({
  transform: `scale(${scale.value}) translate(${position.value.x}px, ${position.value.y}px)`,
  transformOrigin: '0 0'
});
const isDragging = ref(false);
const dragStart = ref({ x: 0, y: 0 });

const cursorPosition = ref({ x: 0, y: 0 });

const updateCursorPosition = (event) => {
  cursorPosition.value = { x: event.clientX, y: event.clientY };
};

const windowDimensions = ref({ width: 0, height: 0 }); // initialise à 0,0

const updateWindowDimensions = () => {
  windowDimensions.value = { width: window.innerWidth, height: window.innerHeight };
};


const positionPoint = (point) => {
  return {
    left: `${(point.x + position.value.x) * scale.value}px`,
    top: `${(point.y + position.value.y) * scale.value}px`,
  };
};


const zoom = (event) => {
  event.preventDefault();

  const delta = event.deltaY > 0 ? -0.1 : 0.1;
  scale.value += delta;
  scale.value = Math.min(Math.max(scale.value, 0.5), 3);

  updateMapStyle();
};

const startDrag = (event) => {
  isDragging.value = true;
  dragStart.value = { x: event.clientX - position.value.x, y: event.clientY - position.value.y };
};

const drag = (event) => {
  if (!isDragging.value) return;
  position.value = { x: event.clientX - dragStart.value.x, y: event.clientY - dragStart.value.y };
  updateMapStyle();
};

const stopDrag = () => {
  isDragging.value = false;
};

const updateMapStyle = () => {
  mapStyle.value = {
    transform: `scale(${scale.value}) translate(${position.value.x}px, ${position.value.y}px)`,
    transformOrigin: '0 0'
  };
};

const pointsOfInterest = ref([
  {
    type: 'Mines',
    subtype: 'Iron',
    x: 1560,
    y: 750
  },
  // Vous pouvez ajouter d'autres points d'intérêt ici.
]);


const pointTypes = {
  'Mines': {
    'Iron': 'iron.png',
    'Copper': 'copper.png',
    'Lead': 'lead.png',
    'Tungsten': 'tungsten.png',
    'Titanium': 'titanium.png',
    'Inorganic salt': 'inorganic_salt.png',
    'General mine': 'general_mine.png'
  },
  'Farms': {
    'Wheat': 'wheat.png',
    'Dandelions': 'dandelions.png',
    'Sugarcane': 'sugarcane.png',
    'Tomatoes': 'tomatoes.png',
    'Poppies': 'poppies.png',
    'Watermelons': 'watermelons.png',
    'Corn': 'corn.png',
    'Chili Peppers': 'chili_peppers.png'
  },
  'Oil Fields': {
    'Default': 'oil_field.png'
  },
  'Gathering Points': {
    'Default': 'gathering_point.png'
  },
  'Animal Dwellings': {
    'Rabbit': 'rabbit.png',
    'Deer': 'deer.png',
    'Wild Boar': 'wild_boar.png',
    'Wolf': 'wolf.png',
    'Bear': 'bear.png',
    'Weak': 'weak.png',
    'Lion': 'lion.png'
  },
  'Rebel Camps': {
    'Default': 'rebel_camp.png'
  },
  'Imperial Territory': {
    'Default': 'imperial_territory.png'
  },
  'Restricted Areas': {
    'Default': 'restricted_area.png'
  }
};

onMounted(() => {
  if (process.client) {
    // Initialise les dimensions au montage du composant
    updateWindowDimensions();

    // Ajoute un écouteur d'événement de redimensionnement de la fenêtre
    window.addEventListener('resize', updateWindowDimensions);
  }
});

onUnmounted(() => {
  if (process.client) {
    // Supprime l'écouteur d'événement de redimensionnement de la fenêtre
    window.removeEventListener('resize', updateWindowDimensions);
  }
});

</script>

<style lang="scss" scoped>
/* Réinitialisation du CSS */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body {
  width: 100%;
  height: 100%;
}

#map-container {
  position: relative;
  overflow: hidden;
  width: 100%; /* S'assurer que le conteneur prend toute la largeur */
  height: 100vh; /* Pour prendre toute la hauteur de la fenêtre du navigateur */
}

#zoom-container {
  transition: transform 0.3s ease;
  width: 100%; /* S'assurer que le conteneur prend toute la largeur */
}

#map {
  width: 100%; /* S'assurer que l'image prend toute la largeur */
  height: auto;
  display: block; /* Éliminer l'espacement en bas de l'image */
}

#point {
  position: absolute;
  width: 20px;
  height: 20px;
  background-color: red;
  border-radius: 50%;
}

.poi {
  position: absolute;
  z-index: 2;
}

#coordinates-box {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 500;
  background-color: white;
}
</style>
