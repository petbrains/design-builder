# Three.js for production web apps

WebGL 3D scenes, particles, shader plates, depth. ~150 KB gzip just for the core. **Justify the bundle cost** — a CSS pseudo-3D card or a pre-rendered video poster is almost always lighter for marketing surfaces.

When Three.js is the right call: interactive 3D product configurators, scroll-driven model spins, shader-driven hero plates that need GPU-quality, particle fields that respond to input.

## Install

```bash
npm install three
```

```js
import * as THREE from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
```

## Basic scene pattern

```js
const canvas = document.getElementById("hero-3d");
const renderer = new THREE.WebGLRenderer({ canvas, alpha: true, antialias: true });
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

function resize() {
  const { clientWidth, clientHeight } = canvas;
  renderer.setSize(clientWidth, clientHeight, false);
  camera.aspect = clientWidth / clientHeight;
  camera.updateProjectionMatrix();
}

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(35, 1, 0.1, 100);
camera.position.set(0, 0, 6);

const mesh = new THREE.Mesh(
  new THREE.IcosahedronGeometry(1.4, 4),
  new THREE.MeshStandardMaterial({ color: 0x64d2ff, roughness: 0.38 })
);
scene.add(mesh);
scene.add(new THREE.HemisphereLight(0xffffff, 0x223344, 2));

const clock = new THREE.Clock();
function tick() {
  const t = clock.getElapsedTime();
  mesh.rotation.y = t * 0.7;
  mesh.rotation.x = Math.sin(t * 0.6) * 0.16;
  renderer.render(scene, camera);
  requestAnimationFrame(tick);
}

resize();
window.addEventListener("resize", resize);
tick();
```

```css
#hero-3d { width: 100%; height: 100%; display: block; }
```

## Performance — the things that actually matter

- **Cap pixel ratio** at 1.5–2. `window.devicePixelRatio` on a modern phone is 3, which means rendering 9× the pixels for negligible visual gain.
- **Disable rendering when off-screen.** Use `IntersectionObserver` on the canvas and stop the rAF loop when `intersectionRatio === 0`.
- **Disable rendering on `document.hidden`.** `visibilitychange` event.
- **Lazy-load Three.js** for below-the-fold scenes. Dynamic `import("three")`.
- **Single material instance** when many meshes share the same look. Material uniform updates are cheap; recompiling shaders is not.
- **Frustum cull with `mesh.frustumCulled = true`** (default true; verify after manual matrix work).

## Reduced motion

Replace the canvas with a static poster image:

```js
const reduce = matchMedia("(prefers-reduced-motion: reduce)").matches;
if (reduce) {
  canvas.outerHTML = `<img src="/hero-poster.jpg" alt="" />`;
} else {
  // ...mount Three.js
}
```

## Loading models (GLTF)

```js
const loader = new GLTFLoader();
loader.load("/models/product.glb", (gltf) => {
  scene.add(gltf.scene);
  if (gltf.animations.length) {
    const mixer = new THREE.AnimationMixer(gltf.scene);
    const action = mixer.clipAction(gltf.animations[0]);
    action.play();
    // advance mixer in tick(): mixer.update(delta)
  }
});
```

For self-hosted models: convert FBX/OBJ to glTF (`.glb`) — smaller and the only format Three.js parses without an extra loader.

## Scroll-driven 3D

Bind camera position or model rotation to scroll progress:

```js
function onScroll() {
  const progress = window.scrollY / window.innerHeight; // 0..1 over first viewport
  camera.position.z = 6 - Math.min(progress, 1) * 3;    // dolly in
  mesh.rotation.y = progress * Math.PI;
}
```

For smoothness, lerp toward the target each tick rather than setting the value directly. Or use ScrollTrigger with `scrub`.

## Good uses

- Genuine 3D where pseudo-3D would lose meaning (configurators, shoes, watches).
- Shader-driven backgrounds when CSS gradient patterns are insufficient.
- Particle fields tied to input (pointer, scroll, audio) at frame rates CSS cannot hit.

## Avoid

- Three.js for what `transform: perspective()` + a few divs can do.
- Loading remote models or HDR environments at render time without a loading state.
- Leaving a render loop running on idle tabs or off-screen canvases.
- Shipping the entire `three.js-examples` directory; tree-shake only the loaders/post-processing you use.
- Post-processing passes (bloom, SSR, DoF) without measuring frame cost on mobile.

## References

- Three.js docs: https://threejs.org/docs/
- `WebGLRenderer`: https://threejs.org/docs/pages/WebGLRenderer.html
- `AnimationMixer`: https://threejs.org/docs/pages/AnimationMixer.html
- glTF format: https://www.khronos.org/gltf/
