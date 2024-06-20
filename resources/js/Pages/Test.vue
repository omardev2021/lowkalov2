<template>
    <div ref="container"></div>
    <div ref="overlay" class="overlay"></div>
    <div class="top-nav flex items-center gap-10">
        <div class="nav bg-white p-4 rounded-xl flex gap-6">
            <a href="#" class="nav-link">Home</a>
            <a href="#" class="nav-link">About</a>
            <a href="#" class="nav-link">Contact</a>
            <a href="#" class="nav-link">Course</a>
        </div>
        <button @click="toggleTooltips" class="bg-white p-4 rounded-full transition duration-300 ease-in-out hover:bg-gray-200">Show</button>
    </div>

    <img src="../assets/lowkalo.png" width="250" class="logo ">
    <div v-for="tooltip in tooltips" :ref="tooltip.ref" :key="tooltip.id" class="tooltip" :style="tooltip.style">{{ tooltip.name }}</div>
</template>




<script setup>
import { onMounted, ref } from 'vue';
import * as THREE from 'three';
import background from '../assets/street-background.png';
import Building1 from '../assets/home1.png';
import Building2 from '../assets/warehous.png';
import Building3 from '../assets/resturant.png';
import Building4 from '../assets/food.png';
import Home2 from '../assets/home 2.png';
import Home3 from '../assets/home 3.png';
import FacncyResturant from '../assets/fancy resturant.png';
import Burger from '../assets/burger resturant.png';
import StreetFood from '../assets/street food.png';
import CoffeeShop from '../assets/coffe shop.png';
import Backery from '../assets/beakery.png';
import Manufacture from '../assets/manufacture.png';
import Building from '../assets/bulding.png';
import Kitchen from '../assets/kitchen.png';
import Supermarket from '../assets/super market.png';
import Branch from '../assets/branch.png';
import Party from '../assets/party.png';



const container = ref(null);
const overlay = ref(null);
const tooltips = ref([]);
const persistentTooltips = ref(false);

function toggleTooltips() {
    persistentTooltips.value = !persistentTooltips.value;
    overlay.value.style.display = persistentTooltips.value ? 'block' : 'none';
    tooltips.value.forEach(tooltip => {
        tooltip.style.display = persistentTooltips.value ? 'block' : 'none';
    });
}

onMounted(() => {
    const scene = new THREE.Scene();

    const aspect = window.innerWidth / window.innerHeight;
    const camera = new THREE.OrthographicCamera(
        -window.innerWidth / 2, window.innerWidth / 2,
        window.innerHeight / 2, -window.innerHeight / 2,
        1, 1000
    );
    camera.position.z = 10;

    const renderer = new THREE.WebGLRenderer();
    renderer.setSize(window.innerWidth, window.innerHeight);
    container.value.appendChild(renderer.domElement);

    let imageWidth, imageHeight;

    function adjustInitialZoom() {
        const aspectRatio = imageWidth / imageHeight;
        const windowAspectRatio = window.innerWidth / window.innerHeight;

        if (aspectRatio > windowAspectRatio) {
            zoomLevel = window.innerWidth / imageWidth;
        } else {
            zoomLevel = window.innerHeight / imageHeight;
        }

        zoomLevel = 5;
        updateCamera();
    }

    function updateCamera() {
        const width = window.innerWidth / 2 * zoomLevel;
        const height = window.innerHeight / 2 * zoomLevel;
        camera.left = -width;
        camera.right = width;
        camera.top = height;
        camera.bottom = -height;
        camera.updateProjectionMatrix();

        restrictCamera();
    }

    function zoom(direction) {
        if (direction === 'in') {
            zoomLevel = Math.max(minZoomLevel, zoomLevel * 0.95);
        } else {
            zoomLevel = Math.min(maxZoomLevel, zoomLevel / 0.95);
        }
        updateCamera();
    }

    function pan(deltaX, deltaY) {
        camera.position.x -= deltaX * zoomLevel;
        camera.position.y += deltaY * zoomLevel;

        restrictCamera();
    }

    function restrictCamera() {
        const halfImageWidth = imageWidth / 2;
        const halfImageHeight = imageHeight / 2;

        const halfViewportWidth = (window.innerWidth / 2) * zoomLevel;
        const halfViewportHeight = (window.innerHeight / 2) * zoomLevel;

        const minX = halfViewportWidth - halfImageWidth;
        const maxX = halfImageWidth - halfViewportWidth;
        const minY = halfViewportHeight - halfImageHeight;
        const maxY = halfImageHeight - halfViewportHeight;

        camera.position.x = Math.max(minX, Math.min(maxX, camera.position.x));
        camera.position.y = Math.max(minY, Math.min(maxY, camera.position.y));
    }

    function onMouseWheel(event) {
        event.preventDefault();
        const direction = (event.deltaY > 0) ? 'out' : 'in';
        zoom(direction);
    }

    function onMouseMove(event) {
        if (!persistentTooltips.value) {
            event.preventDefault();
            if (event.buttons === 1) {
                const deltaX = event.movementX;
                const deltaY = event.movementY;
                pan(deltaX, deltaY);
            }

            const mouse = new THREE.Vector2();
            mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
            mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

            raycaster.setFromCamera(mouse, camera);
            const intersects = raycaster.intersectObjects(scene.children);

            tooltips.value.forEach(tooltip => {
                tooltip.style.display = 'none';
            });

            if (intersects.length > 0) {
                const intersectedObject = intersects[0].object;
                const tooltip = tooltips.value.find(t => t.id === intersectedObject.userData.id);
                if (tooltip) {
                    const canvasBounds = renderer.domElement.getBoundingClientRect();
                    const vector = intersectedObject.position.clone();
                    vector.project(camera);
                    tooltip.style.left = `${(vector.x + 1) / 2 * canvasBounds.width}px`;
                    tooltip.style.top = `${-(vector.y - 1) / 2 * canvasBounds.height - 60}px`; // Adjust -20 to move the tooltip above the building
                    tooltip.style.display = 'block';
                }
            }
        }
    }

    document.addEventListener('wheel', onMouseWheel, { passive: false });
    document.addEventListener('mousemove', onMouseMove, { passive: false });

    window.addEventListener('resize', function() {
        const aspect = window.innerWidth / window.innerHeight;
        updateCamera();
        renderer.setSize(window.innerWidth, window.innerHeight);
        restrictCamera();
    });

    function animate() {
        requestAnimationFrame(animate);
        renderer.render(scene, camera);
    }
    animate();

    const bgTexture = new THREE.TextureLoader().load(background, function(texture) {
        texture.minFilter = THREE.LinearFilter;
        texture.magFilter = THREE.LinearFilter;
        const bgGeometry = new THREE.PlaneGeometry(texture.image.width, texture.image.height);
        const bgMaterial = new THREE.MeshBasicMaterial({ map: texture, side: THREE.DoubleSide });
        const background = new THREE.Mesh(bgGeometry, bgMaterial);
        background.position.z = -10;
        scene.add(background);

        imageWidth = texture.image.width;
        imageHeight = texture.image.height;

        adjustInitialZoom();
    });

    function createBuilding(texturePath, name, position, id) {
        const planeTexture = new THREE.TextureLoader().load(texturePath, function(texture) {
            texture.minFilter = THREE.NearestFilter;
            texture.magFilter = THREE.NearestFilter;
            const planeGeometry = new THREE.PlaneGeometry(texture.image.width, texture.image.height);
            const planeMaterial = new THREE.MeshBasicMaterial({ map: texture, side: THREE.DoubleSide, transparent: true });
            const plane = new THREE.Mesh(planeGeometry, planeMaterial);
            plane.position.z = 40;
            plane.position.set(...position);
            scene.add(plane);

            plane.userData.id = id;
            tooltips.value.push({
                id,
                name,
                ref: `tooltip${id}`,
                style: { display: 'none', position: 'absolute' }
            });
        });
    }


    // Function to trigger mouse movement event




    createBuilding(Building1, 'Front of house', [300, -1000, 0.8], 1);
    createBuilding(Building2, 'Warehouse', [2500, -200, 0.8], 2);
    createBuilding(Building3, 'Main Resturant', [-200, 300, 0.8], 3);
    createBuilding(Building4, 'Food Aggregtor', [-1250, 980, 0.8], 4);
    createBuilding(Home2, 'Home2', [530, -1720, 0.8], 5);
    createBuilding(Home3, 'Home3', [-480, -1520, 0.8], 6);
    createBuilding(FacncyResturant, 'Facncy Resturant', [-950, -730, 0.8], 7);
    createBuilding(Burger, 'Burger Resturant', [-1900, -400, 0.7], 8);
    createBuilding(Backery, 'Backery', [-1380, 159, 0.7], 9);
    createBuilding(StreetFood, 'Street Food', [-2860, -170, 0.7], 10);
    createBuilding(CoffeeShop, 'Coffee Shop', [-2250, 350, 0.4], 11);
    createBuilding(Manufacture, 'Manufacture', [1650, 480, 0.4], 12);
    createBuilding(Building, 'Building', [1350, -500, 0.9], 13);
    createBuilding(Kitchen, 'Kitchen', [980, 970, 0.9], 14);
    createBuilding(Supermarket, 'Supermarket', [50, 1110, 0.9], 15);
    createBuilding(Branch, 'Branch', [550, 1460, 0.4], 16);
    createBuilding(Party, 'Party', [-400, 1840, 0.4], 17);


    const raycaster = new THREE.Raycaster();

    let zoomLevel = 1;
    const maxZoomLevel = 5;
    const minZoomLevel = 1;
});
</script>









<style scoped>
body {
    margin: 0;
    overflow: hidden;
}
canvas {
    display: block;
}
.overlay {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(9, 61, 168, 0.66);
    display: none;
    z-index: 50;
}

.tooltip {
    padding: 10px 20px;
    background-color: white;
    border-radius: 20px;
    font-weight: bold;
    cursor: pointer;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
    font-size: 14px;
    z-index: 100;
    transform: translate(-50%, -100%);
}



.top-nav {
    position: absolute;
    top: 30px;
    right: 40px;


    z-index: 100;
}
.logo {
    position: absolute;
    top: 30px;
    left: 10px;


    z-index: 100;
}
.nav-link {
    transition: all 0.3s ease;
}

.nav-link:hover {
    background-color: #edf2f7;
    color: #2d3748;
}
</style>
