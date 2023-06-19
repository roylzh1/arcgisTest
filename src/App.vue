<template>
	<div class="main-view">
		<div id="viewDiv"></div>
	</div>
</template>

<script setup>
import Map from '@arcgis/core/Map.js';
import MapView from '@arcgis/core/views/MapView.js';
import MapImageLayer from '@arcgis/core/layers/MapImageLayer.js'; //layers/GraphicsLayer
import GraphicsLayer from '@arcgis/core/layers/GraphicsLayer.js'; //Graphic
import Graphic from '@arcgis/core/Graphic.js';
import geometryEngine from '@arcgis/geometry/geometryEngine.js';
import * as identify from '@arcgis/core/rest/identify.js';
import IdentifyParameters from '@arcgis/core/rest/support/IdentifyParameters.js';
import { onMounted } from 'vue';

onMounted(() => {
	const map = new Map({
		basemap: 'satellite',
	});
	// Add two graphics layers to map: one for points, another for buffers
	const bufferLayer = new GraphicsLayer();
	const pointLayer = new GraphicsLayer();
	map.addMany([bufferLayer, pointLayer]);
	const viewOptions = {
		map: map,
		zoom: 3,
		center: [0, 60],
	};
	const view3d = new SceneView(viewOptions);
	view3d.container = 'viewDiv3d';
	view3d.ui.add('info', 'top-right');
	const view2d = new MapView(viewOptions);
	view2d.container = 'viewDiv2d';

	const polySym = {
		type: 'simple-fill', // autocasts as new SimpleFillSymbol()
		color: [140, 140, 222, 0.5],
		outline: {
			color: [0, 0, 0, 0.5],
			width: 2,
		},
	};

	const pointSym = {
		type: 'simple-marker', // autocasts as new SimpleMarkerSymbol()
		color: [255, 0, 0],
		outline: {
			color: [255, 255, 255],
			width: 1,
		},
		size: 7,
	};
	// Indicates whether buffering is enabled
	let bufferEnabled = false;

	function keyDownListener(event) {
		const keyInput = event.key;
		bufferEnabled = keyInput === 'b' && !bufferEnabled;
		document.getElementById('mode').innerHTML = bufferEnabled
			? 'navigation'
			: 'buffering';
	}

	view2d.on('key-down', keyDownListener);
	view3d.on('key-down', keyDownListener);
	view2d.on(['click', 'pointer-move'], event => {
		if (bufferEnabled) {
			createBuffer(event, view2d);
		}
	});

	view3d.on(['click', 'pointer-move'], event => {
		if (bufferEnabled) {
			createBuffer(event, view3d);
		}
	});

	function createBuffer(event, view) {
		// prevent further propagation of the current event bubbling up the event chain.
		// in this case, it will prevent default `drag` event behavior for the MapView
		// which is to move around the view by dragging the pointer.
		event.stopPropagation();

		// convert screen coordinates to map coordinates
		const point = view.toMap(event);

		if (point) {
			bufferPoint(point);
		}
	}

	function bufferPoint(point) {
		if (!bufferEnabled) {
			console.log(
				'buffering not enabled. Hit the b key and click/drag to buffer.'
			);
			return;
		}

		// removes z-values from the point when taken from a SceneView.
		// GeometryEngine does not support 3D geometries.
		point.hasZ = false;
		point.z = undefined;

		if (pointLayer.graphics.length === 0) {
			pointLayer.add(
				new Graphic({
					geometry: point,
					symbol: pointSym,
				})
			);
		} else {
			const graphic = pointLayer.graphics.getItemAt(0);
			graphic.geometry = point;
		}

		/********************************************************************
		 * Geodesic buffer calculates the true distance to buffer a point,
		 * minimizing the distortion that exists when buffering points away
		 * from a projection's line of tangency. This distortion is evident
		 * in the 2D view of this application. This map uses a Web Mercator
		 * spatial reference, which has a line of tangency at the equator.
		 * Buffers created on the equator have very little distortion in their
		 * shape. The further buffers are created away from the equator, the
		 * more distorted they will be in their shape.
		 *
		 * If using a standard planar buffer, the shape of the buffers won't
		 * distort in 2D views, but their size and areas will be very distorted
		 * as they move away from the equator.
		 ********************************************************************/

		const buffer = geometryEngine.geodesicBuffer(point, 560, 'kilometers');

		if (bufferLayer.graphics.length === 0) {
			bufferLayer.add(
				new Graphic({
					geometry: buffer,
					symbol: polySym,
				})
			);
		} else {
			const graphic = bufferLayer.graphics.getItemAt(0);
			graphic.geometry = buffer;
		}
	}
});
</script>

<style scoped>
#viewDiv {
	height: 100%;
	width: 100%;
}
.main-view {
	height: 100vh;
	width: 100vw;
}
.esri-popup .esri-popup-header .esri-title {
	font-size: 18px;
	font-weight: bolder;
}

.esri-popup .esri-popup-body .esri-popup-content {
	font-size: 14px;
}
</style>
