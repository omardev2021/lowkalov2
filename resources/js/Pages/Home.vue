<template>
    <div ref="container" class="z-20"></div>
<!--    <div ref="overlay" class="overlay"></div>-->
    <div class="top-nav flex items-center gap-10 z-90">
        <div class="nav bg-white p-4 rounded-xl flex gap-6">
            <a href="#" class="nav-link">Home</a>
            <a href="#" class="nav-link">About</a>
            <a href="#" class="nav-link" @click.prevent="showModal">Contact</a>
            <a href="#" class="nav-link">Course</a>
        </div>
        <button
            @click="toggleTooltips"
            @mouseenter="toggleHover(true)"
            @mouseleave="toggleHover(false)"
            :class="['p-4 rounded-full transition duration-300 ease-in-out', buttonClass]">
            <img :src="currentIconSrc"
                 width="30"
                 height="30"
                 alt="eye icon">
        </button> </div>

<img src="../assets/lowkalo.png" width="220" class="logo ">

    <transition-group name="tooltip" tag="div">
        <div
            v-for="tooltip in tooltips"
            :key="tooltip.id"
            class="tooltip z-20"
            :style="tooltip.style"
        >
            {{ tooltip.name }}
        </div>
    </transition-group>

    <Transition
    enter-from-class="transition duration-500 opacity-0 translate-y-20 scale-95"
    enter-to-class="transition duration-500 opacity-100 translate-y-0 scale-100">
        <div v-if="isModalVisible" class="bg-white modal-content z-50">
            <h2 class="text-2xl mb-4">Contact Us</h2>
            <p class="mb-4">This is a full-screen modal. Add your contact form or information here.</p>
            <button @click="closeModal" class="bg-red-500 text-white p-2 rounded">Close</button>
        </div>
    </Transition>
</template>



<script setup>
import { onMounted, ref, computed } from 'vue';
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
const tooltips = ref([]);
const persistentTooltips = ref(false);
const isModalVisible = ref(false);

const isToggled = ref(false);
const isHovered = ref(false);

const iconPaths = {
    eyeclosed: '../assets/eyeclosed.png',
    eyeopened: '../assets/eyeopened.png',
    eyeopenedWhite: '../assets/eyeopened-white.png'
};


// Compute the current icon source based on hover and toggle states
const currentIconSrc = computed(() => {
    if (isToggled.value) {
        return iconPaths.eyeopenedWhite;
    }
    return isHovered.value ? iconPaths.eyeopened : iconPaths.eyeclosed;
});

// Compute the button class based on hover and toggle states
const buttonClass = computed(() => {
    if (isToggled.value) {
        return 'bg-[#4215ba]';
    }
    return isHovered.value ? 'hover:bg-gray-200 bg-white' : 'bg-white';
});
const toggleHover = (value) => {
    if (!isToggled.value) {
        isHovered.value = value;
    }
};





const showModal = () => {
    isModalVisible.value = true;
};

const closeModal = () => {
    isModalVisible.value = false;
};



function toggleTooltips() {
    isToggled.value = !isToggled.value;
    persistentTooltips.value = !persistentTooltips.value;
    overlayMesh.visible = persistentTooltips.value;
    tooltips.value.forEach(tooltip => {
        tooltip.style.display = persistentTooltips.value ? 'block' : 'none';
    });
    if (persistentTooltips.value) {
        updateAllTooltipPositions();
    }
}

function updateAllTooltipPositions() {
    tooltips.value.forEach(tooltip => {
        const building = scene.children.find(child => child.userData.id === tooltip.id);
        if (building) {
            const canvasBounds = renderer.domElement.getBoundingClientRect();
            const vector = building.position.clone();
            vector.project(camera);
            tooltip.style.left = `${(vector.x + 1) / 2 * canvasBounds.width}px`;
            tooltip.style.top = `${-(vector.y - 1) / 2 * canvasBounds.height - 60}px`; // Adjust -60 to move the tooltip above the building
        }
    });
}

let overlayMesh;
let scene, camera, renderer;

onMounted(() => {
    scene = new THREE.Scene();

    const aspect = window.innerWidth / window.innerHeight;
    camera = new THREE.OrthographicCamera(
        -window.innerWidth / 2, window.innerWidth / 2,
        window.innerHeight / 2, -window.innerHeight / 2,
        1, 1000
    );
    camera.position.z = 10;

    renderer = new THREE.WebGLRenderer();
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
        adjustOverlaySize();
    }

    function adjustOverlaySize() {
        const overlayWidth = window.innerWidth * zoomLevel;
        const overlayHeight = window.innerHeight * zoomLevel;
        overlayMesh.scale.set(overlayWidth, overlayHeight, 1);
        overlayMesh.position.set(0, 0, 5); // Ensure overlay is in front
    }

    function zoom(direction) {
        if (direction === 'in') {
            zoomLevel = Math.max(minZoomLevel, zoomLevel * 0.96);
        } else {
            zoomLevel = Math.min(maxZoomLevel, zoomLevel / 0.96);
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

            // Ignore overlay mesh for raycasting
            const objectsToIntersect = scene.children.filter(obj => obj !== overlayMesh);
            const intersects = raycaster.intersectObjects(objectsToIntersect);

            tooltips.value.forEach(tooltip => {
                tooltip.style.display = 'none';
            });

            if (intersects.length > 0) {
                const intersectedObject = intersects[0].object;
                const tooltip = tooltips.value.find(t => t.id === intersectedObject.userData.id);
                if (tooltip && tooltip.name !== null) {
                    const canvasBounds = renderer.domElement.getBoundingClientRect();
                    const vector = intersectedObject.position.clone();
                    vector.project(camera);
                    tooltip.style.left = `${(vector.x + 1) / 2 * canvasBounds.width}px`;
                    tooltip.style.top = `${-(vector.y - 1) / 2 * canvasBounds.height - 60}px`; // Adjust -60 to move the tooltip above the building
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

    // Overlay Mesh
    const overlayGeometry = new THREE.PlaneGeometry(window.innerWidth, window.innerHeight);
    const overlayMaterial = new THREE.MeshBasicMaterial({ color: 0x093DA8, opacity: 0.66, transparent: true });
    overlayMesh = new THREE.Mesh(overlayGeometry, overlayMaterial);
    overlayMesh.position.z = 5; // Ensure overlay is in front
    overlayMesh.visible = false; // Initially hidden
    scene.add(overlayMesh);


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
            if (name !== null) {
                tooltips.value.push({
                    id,
                    name,
                    ref: `tooltip${id}`,
                    style: { display: 'none', position: 'absolute' }
                });
            }
        });
    }

    createBuilding(Building1, null, [300, -1000, 4.8], 1);
    createBuilding(Building2, 'Central Warehouse', [2500, -200, 5.8], 2);
    createBuilding(Building3, 'Main Resturant', [-200, 300, 5.8], 3);
    createBuilding(Building4, 'Delivery Aggregators', [-1250, 980, 5.8], 4);
    createBuilding(Home2, null, [530, -1720, 4.8], 5);
    createBuilding(Home3, null, [-480, -1520, 4.8], 6);
    createBuilding(FacncyResturant, null, [-950, -730, 5.8], 7);
    createBuilding(Burger, 'Burger Joint', [-1900, -400, 5.7], 8);
    createBuilding(Backery, 'Backery', [-1380, 159, 5.7], 9);
    createBuilding(StreetFood, 'Street Food', [-2860, -170, 5.7], 10);
    createBuilding(CoffeeShop, 'Coffee Shop', [-2250, 350, 5.4], 11);
    createBuilding(Manufacture, 'Central Kitchen', [1650, 480, 5.4], 12);
    createBuilding(Building, null, [1350, -500, 5.9], 13);
    createBuilding(Kitchen, 'Backery', [980, 970, 5.9], 14);
    createBuilding(Supermarket, 'Supermarket', [50, 1110, 5.9], 15);
    createBuilding(Branch, null, [550, 1460, 4.4], 16);
    createBuilding(Party, null, [-400, 1840, 4.4], 17);

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
    z-index: 20;
    transform: translate(-50%, -100%);
    transition: opacity 0.4s, transform 0.4s;
}

.tooltip-enter-active, .tooltip-leave-active {
    transition: opacity 0.5s;
}

.tooltip-enter-from, .tooltip-leave-to {
    opacity: 0;
}

/*.tooltip-enter-active, .tooltip-leave-active {*/
/*    transition: opacity 0.5s, transform 0.5s;*/
/*}*/
/*.tooltip-enter-from {*/
/*    opacity: 0;*/
/*    transform: translateY(-10px);*/
/*}*/
/*.tooltip-leave-to {*/
/*    opacity: 0;*/
/*    transform: translateY(-10px);*/
/*}*/

.top-nav {
    position: absolute;
    top: 30px;
    right: 40px;


    z-index: 90;
}
.logo {
    position: absolute;
    top: 30px;
    left: 10px;


    z-index: 90;
}
.nav-link {
    transition: all 0.3s ease;
}

.nav-link:hover {
    background-color: #edf2f7;
    color: #2d3748;
}

.modal {
    display: flex;
    align-items: center;
    justify-content: center;
}

.modal-content {
    position: absolute;
    top: 30px;
    right: 40px;
    left: 40px;
    bottom: 30px;
    background-color: white;
    padding: 1rem;
    border-radius: 0.5rem;
    z-index: 100;
}

.modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.75);
    z-index: 999;
}


</style>
