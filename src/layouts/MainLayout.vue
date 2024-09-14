<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="toggleLeftDrawer"
        />
        <q-toolbar-title> Cuculcan </q-toolbar-title>
        <div>By Daniel Macías</div>
      </q-toolbar>
    </q-header>

    <q-drawer v-model="leftDrawerOpen" show-if-above bordered>
      <q-list>
        <q-item-label header> Filters </q-item-label>
        <!-- Toggle for hurricane layers -->
        <q-toggle
          v-model="showHurricanes"
          label="Show Hurricanes"
          color="primary"
          @input="toggleHurricanes"
        />
      </q-list>
    </q-drawer>

    <q-page-container style="width: 100%; height: 100%">
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script setup>
import { ref, provide } from "vue";
const showHurricanes = ref(true);
provide("showHurricanes", showHurricanes); // Making it available for injection in child components

defineOptions({
  name: "MainLayout",
});

const leftDrawerOpen = ref(false);

function toggleLeftDrawer() {
  leftDrawerOpen.value = !leftDrawerOpen.value;
}

// Function to toggle hurricane visibility
function toggleHurricanes() {
  showHurricanes.value = !showHurricanes.value;
}
</script>
