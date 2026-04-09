<script lang="ts">
    import { Scroll } from "$lib";
    import * as THREE from "three";
    import * as d3 from "d3";
    import { onMount } from "svelte";
    import { FontLoader } from "three/addons/loaders/FontLoader.js";
    import type { Font } from "three/addons/loaders/FontLoader.js";
    import { TextGeometry } from "three/addons/geometries/TextGeometry.js";
    import { GLTFLoader } from "three/addons/loaders/GLTFLoader.js";
    import type { TMovie } from "../../types";
    import { addGround, onWindowResize } from "$lib/Helper-3D";

    type Props = {
        movies: TMovie[];
    };
    let { movies }: Props = $props();

    let progress: number = $state(0);
    let threeJSContainer: HTMLElement;
    let camera: THREE.PerspectiveCamera;
    let scene: THREE.Scene;
    let renderer: THREE.WebGLRenderer;
    const FLOOR = -250;
    const clock = new THREE.Clock();

    type TStackRow = { year: string; Comedy: number; Romance: number; Drama: number };
    const genres = ["Comedy", "Romance", "Drama"];
    const genreColors: Record<string, number> = {
        Comedy: 0xf4a261,
        Romance: 0xe76f51,
        Drama: 0x2a9d8f,
    };

    function buildStackData(movies: TMovie[]): TStackRow[] {
        const yearMap: { [year: string]: TStackRow } = {};
        movies.forEach((movie) => {
            const bucket = Math.floor(movie.year.getFullYear() / 5) * 5;
            if (isNaN(bucket)) return;
            const year = bucket.toString();
            if (!yearMap[year]) yearMap[year] = { year, Comedy: 0, Romance: 0, Drama: 0 };
            if (movie.genres.includes("Comedy"))  yearMap[year].Comedy  += 1;
            if (movie.genres.includes("Romance")) yearMap[year].Romance += 1;
            if (movie.genres.includes("Drama"))   yearMap[year].Drama   += 1;
        });
        return Object.values(yearMap).sort((a, b) => Number(a.year) - Number(b.year));
    }

    const stackData = $derived(buildStackData(movies));

    const seagulls: Array<THREE.Object3D> = [];
    let seagullMixer: THREE.AnimationMixer;
    let clapperboardMixer: THREE.AnimationMixer;

    type DecadeGroup = { group: THREE.Group; targetScale: number };
    const decadeGroups: DecadeGroup[] = [];

    let loadedFont: Font | null = null;
    let unitsCreated = false;

    function tryCreateScene() {
        if (!loadedFont || !scene || stackData.length === 0 || unitsCreated) return;
        unitsCreated = true;

        const textGeo = new TextGeometry("Summer Movies", { font: loadedFont, size: 40, depth: 10 });
        textGeo.computeBoundingBox();
        const centerOffset = -0.5 * (textGeo.boundingBox!.max.x - textGeo.boundingBox!.min.x);
        const titleMesh = new THREE.Mesh(
            textGeo,
            new THREE.MeshStandardMaterial({ color: 0xd35400 })
        );
        titleMesh.position.set(centerOffset, FLOOR + 600, 0);
        titleMesh.castShadow = true;
        scene.add(titleMesh);

        createUnits(scene, loadedFont);
        createLegend(scene, loadedFont);
    }

    $effect(() => {
        if (stackData.length > 0) tryCreateScene();
    });

    const DECADE_THRESHOLDS = [18, 19, 20, 21, 22, 38, 39, 40, 41, 58, 59, 60, 61, 78, 79, 80];

    let targetCameraZ = 1800;
    let targetCameraY = 180;

    onMount(() => {
        init(window.innerWidth * 0.55, window.innerHeight * 0.88);
    });

    $effect(() => {
        if (!camera) return;
        targetCameraZ = 1800 - progress * 5;
        targetCameraY = 180 - progress * 0.3;
        decadeGroups.forEach((dg, i) => {
            const threshold = DECADE_THRESHOLDS[i] ?? 95;
            dg.targetScale = progress >= threshold ? 1 : 0;
        });
    });

    function init(W: number, H: number) {
        renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setPixelRatio(window.devicePixelRatio);
        renderer.setSize(W, H);
        renderer.shadowMap.enabled = true;
        renderer.shadowMap.type = THREE.PCFShadowMap;
        threeJSContainer.appendChild(renderer.domElement);

        camera = new THREE.PerspectiveCamera(50, W / H, 10, 5000);
        camera.position.set(0, 180, 1800);

        scene = new THREE.Scene();
        new THREE.TextureLoader().load("3d/sky.jpg", (texture) => {
            texture.repeat.set(0.8, 1);
            scene.background = texture;
        });

        scene.add(new THREE.AmbientLight(0xffffff));
        const light = new THREE.DirectionalLight(0xffffff, 3);
        light.position.set(0, 1500, 1000);
        light.castShadow = true;
        Object.assign(light.shadow.camera, {
            top: 2000, bottom: -2000, left: -2000, right: 2000, near: 1200, far: 2500,
        });
        light.shadow.bias = 0.0001;
        light.shadow.mapSize.set(2048, 1024);
        scene.add(light);

        addGround(scene, FLOOR, "3d/sand.jpg");

        const palmLoader = new GLTFLoader();
        const palmPositions = [
            { x: -900, z: 200 }, { x: 900, z: 200 },
            { x: -750, z: -200 }, { x: 750, z: -200 },
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
            if (gltf.animations.length > 0) {
                clapperboardMixer = new THREE.AnimationMixer(model);
                clapperboardMixer.clipAction(gltf.animations[0]).play();
            }
        });

        const fontLoader = new FontLoader();
        fontLoader.load("3d/helvetiker_bold.typeface.json", (font: Font) => {
            loadedFont = font;
            tryCreateScene();
        });

        window.addEventListener("resize", () =>
            onWindowResize(camera, renderer, window.innerWidth * 0.55, window.innerHeight * 0.88)
        );

        renderer.setAnimationLoop(animate);
    }

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

        stackData.forEach((row) => {
            const xPos = xScale(row.year)! + bandwidth / 2;

            const group = new THREE.Group();
            group.position.set(xPos, FLOOR, 0);
            group.scale.set(0, 0, 0);
            scene.add(group);
            decadeGroups.push({ group, targetScale: 0 });

            let currentY = 0;
            genres.forEach((genre) => {
                const count = row[genre as keyof TStackRow] as number;
                for (let j = 0; j < count; j++) {
                    const mesh = new THREE.Mesh(
                        createGenreGeometry(genre),
                        createGenreMaterial(genre)
                    );
                    mesh.position.set(0, currentY + unitSize / 2, 0);
                    mesh.castShadow = true;
                    mesh.receiveShadow = true;
                    group.add(mesh);
                    currentY += unitSize * (count > 20 ? 0.5 : 1);
                }
                currentY += 5;
            });

            const textGeo = new TextGeometry(row.year, { font, size: 15, depth: 2 });
            textGeo.computeBoundingBox();
            const textWidth = textGeo.boundingBox!.max.x - textGeo.boundingBox!.min.x;
            const textMesh = new THREE.Mesh(
                textGeo,
                new THREE.MeshPhysicalMaterial({ color: 0x000000 })
            );
            textMesh.position.set(xPos - textWidth / 2, FLOOR + 5, bandwidth / 2 + 5);
            textMesh.castShadow = true;
            scene.add(textMesh);
        });
    }

    function createLegend(scene: THREE.Scene, font: Font) {
        const legendX = 400;
        const legendStartY = FLOOR + 650;
        const spacing = 40;

        genres.forEach((genre, i) => {
            const y = legendStartY - i * spacing;
            const swatch = new THREE.Mesh(createGenreGeometry(genre), createGenreMaterial(genre));
            swatch.position.set(legendX, y, 0);
            swatch.castShadow = true;
            scene.add(swatch);

            const textGeo = new TextGeometry(genre, { font, size: 16, depth: 3 });
            const textMesh = new THREE.Mesh(
                textGeo,
                new THREE.MeshPhysicalMaterial({ color: 0x000000 })
            );
            textMesh.position.set(legendX + 18, y - 5, 0);
            textMesh.castShadow = true;
            scene.add(textMesh);
        });
    }

    function animate() {
        const delta = clock.getDelta();

        if (seagullMixer) seagullMixer.update(delta);
        if (clapperboardMixer) clapperboardMixer.update(delta);

        seagulls.forEach((s) => {
            s.position.x += 200 * delta;
            if (s.position.x > 1000) s.position.x = -1000;
        });

        camera.position.z += (targetCameraZ - camera.position.z) * 0.05;
        camera.position.y += (targetCameraY - camera.position.y) * 0.05;

        decadeGroups.forEach(({ group, targetScale }) => {
            const cur = group.scale.x;
            const next = cur + (targetScale - cur) * 0.08;
            group.scale.set(next, next, next);
        });

        renderer.render(scene, camera);
    }
</script>

<h2>A 3D Story of Summer Cinema</h2>

<Scroll bind:progress>

    <div class="section">
        <h3>Lights, Camera, Action!</h3>
        <p>This shows summer movies from 1945 to 2020, grouped into 5-year periods. Each shape on the right is one movie — Comedy is an icosahedron, Romance is a torus knot, and Drama is an octahedron.</p>
        <p>Scroll down to watch each era appear.</p>
    </div>

    <div class="section">
        <h3>The Early Years (1945–1965)</h3>
        <p>Not many summer movies were made back then. Each 5-year period had just a handful of films, which is why the first columns are so short.</p>
        <p>Drama led from the start. Comedy and Romance were around, but small. The gap between Drama and the other two was big from day one.</p>
    </div>

    <div class="section">
        <h3>The Blockbuster Era (1970–1985)</h3>
        <p>Summer became Hollywood's biggest season. Studios started releasing their biggest films between June and August, and output grew fast.</p>
        <p>Comedy picked up a lot here. Drama still leads, but the columns are much taller now compared to the early years.</p>
    </div>

    <div class="section">
        <h3>Romance Takes Off (1990–2005)</h3>
        <p>Romance grew into a real genre during the 90s — you can clearly see it stacking up now. All three genres were growing at the same time.</p>
        <p>By the early 2000s, a single 5-year period had 30 or more Drama films. The stacks are getting close to the palm trees.</p>
    </div>

    <div class="section">
        <h3>The Modern Era (2010–2020)</h3>
        <p>The 2010s had more summer movies than any other decade. Streaming created huge demand for new content, and summer releases went well beyond theaters.</p>
        <p>Drama is still the tallest by a lot, but Comedy and Romance also hit their highest counts ever in this period.</p>
    </div>

    <svelte:fragment slot="viz">
        <div bind:this={threeJSContainer}></div>
    </svelte:fragment>
</Scroll>

<style>
    .section {
        height: 100vh;
        padding: 40px 20px;
        border-left: 3px solid #e0e0e0;
        padding-left: 20px;
    }

    h3 {
        font-size: 22px;
        margin-bottom: 14px;
    }

    p {
        font-size: 18px;
        color: #444;
        line-height: 1.7;
        margin-bottom: 12px;
    }
</style>