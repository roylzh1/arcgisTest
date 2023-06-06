<template>
	<div class="main-view">
		<div id="viewDiv"></div>
	</div>
</template>

<script setup>
import Map from '@arcgis/core/Map.js';
import MapView from '@arcgis/core/views/MapView.js';
import { onMounted } from 'vue';
onMounted(() => {
	const map = new Map({
		basemap: 'topo-vector',
	});
	const view = new MapView({
		container: 'viewDiv', // Reference to the DOM node that will contain the view
		map: map, // References the map object created in step 3
	});
});
</script>

<style scoped>
#viewDiv {
	height: 100%;
	width: 100%;
}
.main-view{
  height: 100vh;
  width: 100vw;
}
</style>
