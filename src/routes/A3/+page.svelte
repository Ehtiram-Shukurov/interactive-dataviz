<script lang="ts">
    import { onMount } from "svelte";
    import * as THREE from "three";
    import { FontLoader, Font } from "three/addons/loaders/FontLoader.js";
    import { TextGeometry } from "three/addons/geometries/TextGeometry.js";
    import { GLTFLoader } from "three/addons/loaders/GLTFLoader.js";
    import type { TMovie } from "../../types";
    import { addGround, onWindowResize, loadModels } from "$lib/Helper-3D";

    import * as d3 from "d3";

    let movies: TMovie[] = $state([]);

    // Stacked data: each year has counts per genre
    type TStackRow = {
        year: string;
        Comedy: number;
        Romance: number;
        Drama: number;
    };

    let stackData: TStackRow[] = $state([]);
    const genres = ["Comedy", "Romance", "Drama"];
    const genreColors: Record<string, number> = {
        Comedy: 0xf4a261,  // warm orange
        Romance: 0xe76f51,  // coral red
        Drama: 0x2a9d8f,   // teal
    };

    // === Load Data ===
    async function loadCsv() {
        const csvUrl = "./summer_movies.csv";
        const movies = await d3.csv(csvUrl);
        const yearMap: { [year: string]: TStackRow } = {};

        movies.forEach((row) => {
            const decade = Math.floor(Number(row.year) / 5) * 5;
            if (isNaN(decade)) return;
            const year = decade.toString();
            const genres = row.genres ? row.genres.split(",") : [];

            if (!yearMap[year]) {
                yearMap[year] = { year, Comedy: 0, Romance: 0, Drama: 0 };
            }

            if (genres.includes("Comedy"))  yearMap[year].Comedy  += 1;
            if (genres.includes("Romance")) yearMap[year].Romance += 1;
            if (genres.includes("Drama"))   yearMap[year].Drama   += 1;
        });
        stackData = Object.values(yearMap).sort((a, b) => 
            Number(a.year) - Number(b.year)
        );

        console.log("Stack data:", stackData);
    }
    let container: HTMLElement;
    let camera: THREE.PerspectiveCamera;
    let scene: THREE.Scene;
    let renderer: THREE.WebGLRenderer;
    const FLOOR = -250;

    const morphs: Array<THREE.Mesh> = [];
    let mixer: THREE.AnimationMixer;
    const clock = new THREE.Clock();

    const seagulls: Array<THREE.Object3D> = [];
    let clapperboard: THREE.Object3D | null = null;
    let seagullMixer: THREE.AnimationMixer;
    let clapperboardMixer: THREE.AnimationMixer;

    let animating = $state(false);

    onMount(async () => {
        await loadCsv();
        init(window.innerWidth, window.innerHeight);
    });

    function init(SCREEN_WIDTH: number, SCREEN_HEIGHT: number) {
        // Renderer
        renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setPixelRatio(window.devicePixelRatio);
        renderer.setSize(SCREEN_WIDTH, SCREEN_HEIGHT);
        renderer.shadowMap.enabled = true;
        renderer.shadowMap.type = THREE.PCFShadowMap;
        container.appendChild(renderer.domElement);

        // Camera
        camera = new THREE.PerspectiveCamera(
            35,
            SCREEN_WIDTH / SCREEN_HEIGHT,
            10,
            5000,
        );
        camera.position.set(0, 130, 2000);
        camera.updateProjectionMatrix();

        // Scene
        scene = new THREE.Scene();
        new THREE.TextureLoader().load("3d/sky.jpg", (texture) => {
            texture.repeat.set(0.8, 1);
            scene.background = texture;
        });

        // Ambient Light
        const ambient = new THREE.AmbientLight(0xffffff);
        scene.add(ambient);

        // Directional Light
        const light = new THREE.DirectionalLight(0xffffff, 3);
        light.position.set(0, 1500, 1000);
        light.castShadow = true;
        Object.assign(light.shadow.camera, {
            top: 2000,
            bottom: -2000,
            left: -2000,
            right: 2000,
            near: 1200,
            far: 2500,
        });
        light.shadow.bias = 0.0001;
        light.shadow.mapSize.width = 2048;
        light.shadow.mapSize.height = 1024;
        scene.add(light);

        // Ground
        addGround(scene, FLOOR, "3d/sand.jpg");

        // Title + Stacked Bars
        const fontLoader = new FontLoader();
        fontLoader.load("3d/helvetiker_bold.typeface.json", (font: Font) => {
            // Title
            const textGeo = new TextGeometry("Summer Movies", {
                font: font,
                size: 40,
                depth: 10,
            });
            textGeo.computeBoundingBox();
            const centerOffset =
                -0.5 *
                (textGeo!.boundingBox!.max.x - textGeo!.boundingBox!.min.x);
            const textMaterial = new THREE.MeshStandardMaterial({
                color: 0xd35400,
            });
            const titleMesh = new THREE.Mesh(textGeo, textMaterial);
            titleMesh.position.x = centerOffset;
            titleMesh.position.y = FLOOR + 600;
            titleMesh.castShadow = true;
            scene.add(titleMesh);

            createUnits(scene, font);
            createLegend(scene, font);
        });

        // === Expressive 3D Objects ===

        // Load bird models (linear motion)
        const birdModels = [
            {
                path: "3d/Flamingo.glb",
                speed: 350,
                duration: 1,
                x: 300 - Math.random() * 500,
                y: FLOOR + 550,
                z: -100,
                scale: 0.9,
            },
            {
                path: "3d/Parrot.glb",
                speed: 350,
                duration: 0.5,
                x: 500 - Math.random() * 500,
                y: FLOOR + 500,
                z: 700,
                scale: 0.7,
            },
        ];
        mixer = loadModels(birdModels, scene, mixer, morphs);
        // Palm trees
            const palmLoader = new GLTFLoader();
            const palmPositions = [
                { x: -900, z: 200 },
                { x: 900, z: 200 },
                { x: -750, z: -200 },
                { x: 750, z: -200 },
            ];
            palmPositions.forEach((pos) => {
                palmLoader.load("3d/palmtree.glb", (gltf) => {
                    const model = gltf.scene;
                    model.scale.set(150, 150, 150);
                    model.position.set(pos.x, FLOOR, pos.z);
                    model.castShadow = true;
                    scene.add(model);
                });
            });
            // Seagull
            const seagullLoader = new GLTFLoader();
            seagullLoader.load("3d/seagull.glb", (gltf) => {
                if (seagulls.length > 0) return;
                const model = gltf.scene;
                model.scale.set(7, 7, 7);
                model.position.set(-800, FLOOR + 600, 300);
                model.rotation.y = Math.PI / 2;
                model.castShadow = true;
                scene.add(model);
                seagulls.push(model);
                if (gltf.animations.length > 0) {
                    seagullMixer = new THREE.AnimationMixer(model);
                    seagullMixer.clipAction(gltf.animations[0]).play();
                }
            });
            const clapperLoader = new GLTFLoader();
            clapperLoader.load("3d/clapperboard.glb", (gltf) => {
                const model = gltf.scene;
                model.scale.set(40, 40, 40);
                model.position.set(940, FLOOR + 760, 0);
                model.rotation.y = -Math.PI / 2;
                model.castShadow = true;
                scene.add(model);
                clapperboard = model;

                if (gltf.animations.length > 0) {
                    clapperboardMixer = new THREE.AnimationMixer(model);
                    clapperboardMixer.clipAction(gltf.animations[0]).play();
                }
            });

        // Handle resize
        window.addEventListener("resize", () =>
            onWindowResize(
                camera,
                renderer,
                window.innerWidth,
                window.innerHeight,
            ),
        );

        renderer.setAnimationLoop(animate);
    }

    // === Unit Visualization ===
    
    function createGenreGeometry(genre: string): THREE.BufferGeometry {
        if (genre === "Comedy")  return new THREE.IcosahedronGeometry(8);
        if (genre === "Romance") return new THREE.TorusKnotGeometry(5, 1, 50, 8);
        if (genre === "Drama")   return new THREE.OctahedronGeometry(8);
        return new THREE.SphereGeometry(5);
    }

    function createGenreMaterial(genre: string): THREE.Material {
        return new THREE.MeshPhysicalMaterial({ 
            color: genreColors[genre],
            roughness: 0.3,
            metalness: 0.2,
        });
    }

    function createUnits(scene: THREE.Scene, font: Font) {
        if (stackData.length === 0) return;
        const barMaxWidth = Math.min(stackData.length * 100, 1400);
        const unitSize = 6;
        const xScale = d3
            .scaleBand()
            .domain(stackData.map((d) => d.year))
            .range([-barMaxWidth / 2, barMaxWidth / 2])
            .padding(0.1);
    
        const bandwidth = xScale.bandwidth();
    
        // Stack unit objects per year
        stackData.forEach((row) => {
            const xPos = xScale(row.year)! + bandwidth / 2;
            let currentY = FLOOR;
    
            genres.forEach((genre) => {
                const count = row[genre as keyof TStackRow] as number;
                for (let i = 0; i < count; i++) {
                    const geo = createGenreGeometry(genre);
                    const mat = createGenreMaterial(genre);
                    const mesh = new THREE.Mesh(geo, mat);
                    mesh.position.set(xPos, currentY + unitSize / 2, 0);
                    mesh.castShadow = true;
                    mesh.receiveShadow = true;
                    scene.add(mesh);
                    currentY += unitSize * (count > 20 ? 0.5 : 1);
                }
                currentY += 5;
            });
        });
        stackData.forEach((row) => {
            const x = xScale(row.year)! + bandwidth / 2;
            const textGeo = new TextGeometry(row.year, {
                font: font,
                size: 15,
                depth: 2,
            });
            textGeo.computeBoundingBox();
            const textWidth = textGeo.boundingBox!.max.x - textGeo.boundingBox!.min.x;
            const textMat = new THREE.MeshPhysicalMaterial({ color: 0x000000 });
            const textMesh = new THREE.Mesh(textGeo, textMat);
            textMesh.position.set(x - textWidth / 2, FLOOR + 5, bandwidth / 2 + 5);
            textMesh.castShadow = true;
            scene.add(textMesh);
        });
    }
    
    // function createBars(scene: THREE.Scene, font: Font) {
    //     if (stackData.length === 0) return;

    //     const barMaxWidth = Math.min(stackData.length * 80, 1400);

    //     const xScale = d3
    //         .scaleBand()
    //         .domain(stackData.map((d) => d.year))
    //         .range([-barMaxWidth / 2, barMaxWidth / 2])
    //         .padding(0.1);

    //     const bandwidth = xScale.bandwidth();
    //     const barWidth = Math.min(bandwidth * 0.7, 25);

    //     // One colored bar segment per genre, stacked per year
    //     stackData.forEach((row) => {
    //         const xPos = xScale(row.year)! + bandwidth / 2;
    //         let currentY = FLOOR;

    //         genres.forEach((genre) => {
    //             const count = row[genre as keyof TStackRow] as number;
    //             const barHeight = count * 10;
    //             if (barHeight === 0) return;

    //             const geometry = new THREE.BoxGeometry(barWidth, barHeight, barWidth);
    //             const material = new THREE.MeshPhysicalMaterial({ color: genreColors[genre] });
    //             const mesh = new THREE.Mesh(geometry, material);
    //             mesh.position.set(xPos, currentY + barHeight / 2, 0);
    //             mesh.castShadow = true;
    //             mesh.receiveShadow = true;
    //             scene.add(mesh);
    //             currentY += barHeight;
    //         });
    //     });

    //     // Year labels
    //     stackData.forEach((row) => {
    //         const x = xScale(row.year)! + bandwidth / 2;
    //         const textGeo = new TextGeometry(row.year, {
    //             font: font,
    //             size: 6,
    //             depth: 2,
    //         });
    //         textGeo.computeBoundingBox();
    //         const textWidth = textGeo.boundingBox!.max.x - textGeo.boundingBox!.min.x;
    //         const textMat = new THREE.MeshPhysicalMaterial({ color: 0x000000 });
    //         const textMesh = new THREE.Mesh(textGeo, textMat);
    //         textMesh.position.set(x - textWidth / 2, FLOOR + 5, barWidth / 2 + 5);
    //         textMesh.castShadow = true;
    //         scene.add(textMesh);
    //     });
    // }
    function createLegend(scene: THREE.Scene, font: Font) {
        const legendX = 400;
        const legendStartY = FLOOR + 650;
        const spacing = 40;

        genres.forEach((genre, i) => {
            const y = legendStartY - i * spacing;

            const geo = createGenreGeometry(genre);
            const mat = createGenreMaterial(genre);
            const swatch = new THREE.Mesh(geo, mat);
            swatch.position.set(legendX, y, 0);
            swatch.castShadow = true;
            scene.add(swatch);

            const textGeo = new TextGeometry(genre, {
                font: font,
                size: 16,
                depth: 3,
            });
            const textMat = new THREE.MeshPhysicalMaterial({ color: 0x000000 });
            const textMesh = new THREE.Mesh(textGeo, textMat);
            textMesh.position.set(legendX + 18, y - 5, 0);
            textMesh.castShadow = true;
            scene.add(textMesh);
        });
    }

    function animate() {
        const delta = clock.getDelta();

        if (animating) {
            mixer.update(delta);
            if (seagullMixer) seagullMixer.update(delta);
            if (clapperboardMixer) clapperboardMixer.update(delta);

            // Birds - linear motion
            morphs.forEach((morph) => {
                morph.position.x += morph.speed * delta;
                if (morph.position.x > window.innerWidth / 2) {
                    morph.position.x = -window.innerWidth / 2 - Math.random() * 200;
                }
            });
            seagulls.forEach((seagull) => {
                seagull.position.x += 200 * delta;
                if (seagull.position.x > 1000) {
                    seagull.position.x = -1000;
                }
            });
        }

        renderer.render(scene, camera);
    }
</script>

<div bind:this={container} class="container"></div>

<button class="toggle-btn" onclick={() => animating = !animating}>
    {animating ? "Pause" : "Play"} Animation
</button>

<style>
    div.container {
        width: 100%;
        height: 100%;
    }

    .toggle-btn {
        position: fixed;
        bottom: 10px;
        right: 10px;
        padding: 8px 10px;
        font-size: 12px;
        border: none;
        border-radius: 20px;
        background: rgba(0, 0, 0, 0.30);
        color: white;
        cursor: pointer;
        z-index: 10;
    }

    .toggle-btn:hover {
        background: rgba(0, 0, 0, 0.8);
    }
</style>