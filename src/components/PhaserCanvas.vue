<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue';
import Phaser from 'phaser';
import { OfficeScene } from '../game/OfficeScene.js';

const contenidorJoc = ref(null);
let game = null;
const emit = defineEmits(['canviZona']);

// [NOU] Funció pont: rep l'ordre de App.vue i la delega a l'escena de Phaser
function bloquejarControls(estat) {
  const escena = game?.scene.getScene('OfficeScene');
  if (escena) {
    escena.bloquejarControls(estat);
  }
}

// [NOU] Exposa el mètode perquè el pare (App.vue) hi accedeixi via ref
defineExpose({ bloquejarControls }); // <-- CLAU: Connexió Vue -> Phaser (Annex)

onMounted(() => {
  game = new Phaser.Game({
    type: Phaser.AUTO,
    width: 800,
    height: 600,
    parent: contenidorJoc.value,
    physics: { default: 'arcade', arcade: { debug: false } },
    scene: [OfficeScene]
  });

  // El pont captura l'objecte { nom, streamUrl } de Phaser i el passa a Vue
  game.events.on('canvi-zona', (dadaZona) => {
    emit('canviZona', dadaZona); // <-- IMPORTANT: Reenvia la dada cap amunt
  });
});

onBeforeUnmount(() => {
  game?.destroy(true);
});
</script>

<template>
  <div ref="contenidorJoc" class="marc-canvas"></div>
</template>
