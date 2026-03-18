# Graphics Performance and Rendering for Browser-based Racing Games

> Comprehensive research on Three.js/WebGL/WebGPU optimization, shader techniques, particle systems, post-processing, LOD, performance budgets, and the SuperTuxKart rendering pipeline.

---

## Table of Contents

1. [Three.js / WebGL / WebGPU Performance Optimization](#1-threejs--webgl--webgpu-performance-optimization-for-racing-games)
2. [Shader Techniques: Toon Shading, Cel Shading, PBR](#2-shader-techniques-toon-shading-cel-shading-pbr)
3. [Particle Systems: Boost, Transformations, Exhaust](#3-particle-systems-boost-transformations-exhaust)
4. [Post-Processing Pipeline](#4-post-processing-pipeline)
5. [LOD Systems and Asset Streaming](#5-lod-systems-and-asset-streaming)
6. [Target Frame Rates and Performance Budgets](#6-target-frame-rates-and-performance-budgets)
7. [WebGL vs WebGPU Performance in 2025/2026](#7-webgl-vs-webgpu-performance-in-20252026)
8. [SuperTuxKart Rendering Pipeline — What Can Be Adapted](#8-supertuxkart-rendering-pipeline--what-can-be-adapted)
9. [Synthesis: Recommended Rendering Architecture](#9-synthesis-recommended-rendering-architecture)

---

## 1. Three.js / WebGL / WebGPU Performance Optimization for Racing Games

### The Golden Rule: Draw Calls

The single most impactful optimization in Three.js is **minimizing draw calls**. From [100 Three.js Tips That Actually Improve Performance (2026, Utsubo)](https://www.utsubo.com/blog/threejs-best-practices-100-tips):

> *"Target under 100 draw calls per frame. Below 100 draw calls, most devices maintain smooth 60fps. Above 500, even powerful GPUs struggle."*
> — Check with `renderer.info.render.calls`

| Draw Call Count | Performance Impact |
|----------------|-------------------|
| < 100 | Smooth 60fps on most devices |
| 100–300 | Acceptable on mid-range hardware |
| 300–500 | Noticeable performance issues |
| 500+ | Even powerful GPUs struggle |

### Instancing and Batching

For a racing game with repeated geometry (trees, cones, track barriers, spectators):

| Technique | Use Case | Draw Call Reduction |
|-----------|----------|-------------------|
| **InstancedMesh** | Many identical objects (trees, cones) | 1000 objects → 1 draw call (90%+ reduction) |
| **BatchedMesh** | Different car models sharing material | N models → 1 draw call |
| **Material sharing** | Multiple meshes, identical material | Auto-batched by Three.js |
| **BufferGeometryUtils.mergeGeometries()** | Static track geometry (fences, walls) | All merged into 1 draw call |

A [Three.js forum post (Feb 2026)](https://discourse.threejs.org/t/one-draw-call-massive-crowd-performance-engineering-in-three-js/89928) demonstrates rendering a massive animated crowd in a **single draw call** using `InstancedMesh`:
- All characters share one merged geometry
- Per-vertex ID in shader differentiates head/body/legs without separate meshes
- Character colors packed into minimal instance attributes, reconstructed in shader
- Updates amortized over multiple frames (not every character every frame)
- Expensive rotation trig cached and reused — alone recovered ~25fps at high instance counts

### Asset Optimization

| Technique | Method | Impact |
|-----------|--------|--------|
| **Geometry compression** | Draco compression | 60–80% size reduction |
| **Texture compression** | KTX2 format (UASTC for quality, ETC1S for size) | 4–8x reduction, GPU-native decompression |
| **Texture atlases** | Pack car liveries/road textures into one | Fewer bind calls, better cache usage |
| **Pack RGBA channels** | Store multiple data maps in one texture | Store grip, damage, wetness in R/G/B/A |
| **Array textures** | For modern browsers | Single bind for multiple related textures |

### Key Three.js Configuration Tips (2026)

From [Utsubo 2026 guide](https://www.utsubo.com/blog/threejs-best-practices-100-tips):

```javascript
// Material precision for mobile
material.precision = 'mediump'; // 2x faster for car paint/reflections on mobile

// Frustum culling (auto-enabled, verify it's on)
mesh.frustumCulled = true;

// Dispose unused resources
geometry.dispose();
material.dispose();
texture.dispose();
renderTarget.dispose(); // Critical — memory leaks tank long sessions

// Use BufferGeometryUtils for static track geometry
import { mergeGeometries } from 'three/examples/jsm/utils/BufferGeometryUtils.js';
const mergedTrack = mergeGeometries([segment1, segment2, segment3]);

// Replace conditionals in shaders with branchless math
// AVOID: if (u_damaged) color = red; else color = normal;
// USE: color = mix(normal, red, step(0.5, u_damaged));
```

### Memory Management

- **Dispose everything** when unloading a scene or race
- Common leak pattern: render targets for post-processing effects never disposed between races
- Use `renderer.info` to monitor draw calls, triangles, programs, and texture count in real time

---

## 2. Shader Techniques: Toon Shading, Cel Shading, PBR

### Toon / Cel Shading in Three.js

Toon shading is ideal for a stylized kart racing game. It creates discrete bands of light/shadow for a cartoon aesthetic. [Tutorial by Maya Nedeljkovich](https://www.maya-ndljk.com/blog/threejs-basic-toon-shader):

**Core Theory**: Apply a `smoothstep` function to the diffuse light intensity to create discrete, crisp shadow bands instead of gradual falloff.

Five key components of a complete toon shader:
1. **Flat color base** — single base color for the mesh
2. **Core shadow** — sharp cutoff between lit and shadow areas using `dot(normal, lightDir)` + `smoothstep`
3. **Specular reflection** — Blinn-Phong specular with `smoothstep` for crisp highlight
4. **Rim light** — edge highlight using `1.0 - dot(viewDir, normal)` for cartoon depth
5. **Received shadows** — `getShadow()` with `PCFSoftShadowMap` for soft cartoon shadows

```glsl
// Core shadow band — fragment shader
float lightIntensity = dot(vNormal, lightDir);
float shadow = smoothstep(0.0, 0.01, lightIntensity); // crisp cutoff
gl_FragColor = vec4(uColor * shadow, 1.0);
```

**Three.js built-in toon material**: [`MeshToonMaterial`](https://threejs.org/docs/#api/materials/MeshToonMaterial) provides a simple 3-tone or 5-tone toon shader without custom shader code:

```javascript
const material = new THREE.MeshToonMaterial({
  color: 0xff0000,
  gradientMap: toneTexture // 3-tone or 5-tone gradient texture
});
```

**Outline effect** for the classic cel-shaded look: Render a slightly scaled-up, back-face-only black version of each model. Three.js's `OutlineEffect` handles this automatically.

### PBR (Physically Based Rendering)

For realistic car materials (metallic paint, rubber tires, glass):

```javascript
const carPaint = new THREE.MeshStandardMaterial({
  color: 0x1a3a8f,
  metalness: 0.3,   // slight metallic sheen for car paint
  roughness: 0.4,   // semi-glossy
  envMapIntensity: 1.0 // reflection from environment map
});

const tire = new THREE.MeshStandardMaterial({
  color: 0x222222,
  metalness: 0.0,
  roughness: 0.9    // matte rubber
});
```

**PBR vs Toon choice**: For a kart racing game inspired by Nintendo/Pixar aesthetics, **toon shading** is recommended. For a realistic simulator, **PBR with MeshStandardMaterial** is the standard.

### TSL (Three Shader Language) — Future-Proof Shaders

[TSL](https://www.utsubo.com/blog/threejs-best-practices-100-tips) is Three.js's node-based material system that compiles to either **WGSL** (WebGPU) or **GLSL** (WebGL) from the same code:

```javascript
import { Fn, vec4, uniform, positionLocal } from 'three/tsl';

const toonMaterial = new THREE.NodeMaterial();
toonMaterial.colorNode = Fn(() => {
  const light = dot(normalView, directionalLight.direction).clamp(0, 1);
  const bands = floor(light.mul(3.0)).div(3.0); // 3 toon bands
  return vec4(baseColor.mul(bands), 1.0);
})();
```

TSL is the recommended approach for new Three.js projects (r171+) targeting both WebGL and WebGPU.

---

## 3. Particle Systems: Boost, Transformations, Exhaust

### CPU vs GPU Particle Systems

| Approach | Max Particles | CPU Cost | GPU Cost | Browser Support |
|----------|--------------|----------|----------|----------------|
| **CPU (JS) particles** | ~50,000 | High | Low | All |
| **GPGPU (fragment shader)** | ~500,000 | Minimal | Medium | WebGL 2+ |
| **WebGPU compute shaders** | Millions | Near-zero | High | Chrome 113+, Firefox 141+, Safari 26+ |

From [Utsubo 2026 guide](https://www.utsubo.com/blog/threejs-best-practices-100-tips):
> *"CPU-based particle updates hit bottlenecks around 50,000 particles on typical hardware. WebGPU compute shaders push this to millions."*

### GPGPU Pattern (WebGL — current baseline)

Using Three.js `GPUComputationRenderer` for exhaust/boost particles:

```javascript
import { GPUComputationRenderer } from 'three/examples/jsm/misc/GPUComputationRenderer.js';

const gpgpu = new GPUComputationRenderer(textureSize, textureSize, renderer);
const positionVariable = gpgpu.addVariable('uCurrentPosition', positionFragmentShader, positionTexture);
const velocityVariable = gpgpu.addVariable('uCurrentVelocity', velocityFragmentShader, velocityTexture);
gpgpu.setVariableDependencies(positionVariable, [positionVariable, velocityVariable]);
gpgpu.init();

// Each frame:
gpgpu.compute(); // GPU updates positions/velocities in fragment shader
```

Detailed tutorial by [Codrops (Dec 2024)](https://tympanus.net/codrops/2024/12/19/crafting-a-dreamy-particle-effect-with-three-js-and-gpgpu/).

### WebGPU Compute Shader Pattern (recommended for 2025+)

```javascript
import { instancedArray, storage, uniform } from 'three/tsl';

const positions = instancedArray(particleCount, 'vec3');
const velocities = instancedArray(particleCount, 'vec3');

const updateParticles = Fn(() => {
  const pos = positions.element(instanceIndex);
  const vel = velocities.element(instanceIndex);
  const gravity = vec3(0, -9.8, 0);
  
  // Apply boost exhaust behavior
  velocities.element(instanceIndex).assign(vel.add(gravity.mul(deltaTime)));
  positions.element(instanceIndex).assign(pos.add(vel.mul(deltaTime)));
})().compute(particleCount);
```

[WebGPU Experts (Jan 2025)](https://www.webgpuexperts.com/best-webgpu-updates-january-2025) document fluid simulations handling **300,000 particles** on mid-range GPUs in real-time using WebGPU.

### Racing Game Particle Effects

| Effect | Technique | Notes |
|--------|-----------|-------|
| **Exhaust smoke** | GPU particles (billboard quads) | Velocity-based lifetime, alpha fade |
| **Boost flames** | GPU particles + additive blending | Orange/blue color gradient by velocity |
| **Tire smoke/drift** | GPGPU fragment shader | Ground contact detection, white/gray color |
| **Water splash** | CPU particles (rare, short-lived) | Triggered by track wetness zones |
| **Explosion (item)** | GPU particles + camera shake | Bright flash → debris → smoke |
| **Sparkle (speed boost pad)** | InstancedMesh with animated UV | Very low draw call cost |
| **Item transformation effects** | TSL node materials + morphing | Per-kart transformation animation |

---

## 4. Post-Processing Pipeline

### Three.js Post-Processing Stack

[Three.js Journey post-processing lesson](https://threejs-journey.com/lessons/post-processing) explains the multi-pass architecture:
- Each pass renders to a **render target** (off-screen texture)
- Passes chain: Scene Render → Pass A → Pass B → ... → Screen
- Two render targets swap: can't read and write the same texture simultaneously

### Available Effects (pmndrs/postprocessing library)

The [pmndrs/postprocessing](https://github.com/pmndrs/postprocessing) library provides optimized passes that merge multiple effects into fewer GPU passes:

```javascript
import { EffectComposer, BloomEffect, MotionBlurEffect, DepthOfFieldEffect } from 'postprocessing';

const composer = new EffectComposer(renderer);
composer.addPass(new RenderPass(scene, camera));
composer.addPass(new EffectPass(camera,
  new BloomEffect({ intensity: 1.5, threshold: 0.6 }),
  new MotionBlurEffect({ samples: 8 }),
  // Combined into ONE extra draw pass
));
```

### Recommended Effects for a Kart Racing Game

| Effect | Purpose | Performance Cost | Priority |
|--------|---------|------------------|----------|
| **Bloom (Unreal)** | Glowing boost flames, speed pads, item collection | Medium | High |
| **Motion blur** | Speed sensation; per-object blur for fast karts | High | Medium |
| **Depth of field** | Cinematic replay mode, intro cameras | High | Low (replay only) |
| **FXAA/SMAA** | Anti-aliasing without MSAA cost | Low | High |
| **Color grading (LUT)** | Stylized color palette per track biome | Very low | High |
| **Vignette** | Frame edges darken; focuses attention | Near-zero | Medium |
| **Chromatic aberration** | Impact feedback, item effect screen flash | Near-zero | Low |
| **Screen space reflections** | Shiny track surfaces | Very high | Avoid in racing |

From [Utsubo 2026](https://www.utsubo.com/blog/threejs-best-practices-100-tips):
- Use `pmndrs/postprocessing` to merge bloom + motion blur into **fewer passes**
- Disable multisampling (MSAA) and handle anti-aliasing in the final post pass
- For WebGPU, use TSL's native `bloom()` and `fxaa()` functions for lower overhead

### WebGPU Post-Processing (TresJS / native TSL)

[TresJS post-processing](https://post-processing.tresjs.org/guide/) provides a Vue.js-friendly wrapper:

```html
<EffectComposerPmndrs>
  <BloomPmndrs :intensity="1.5" />
  <MotionBlurPmndrs />
</EffectComposerPmndrs>
```

### Performance-Aware Post-Processing Strategy

1. **Profile first**: Use `stats-gl` or `Spector.js` to identify which passes consume most GPU time
2. **Merge passes**: Single `EffectPass` with multiple effects > multiple `EffectPass` instances
3. **Conditional quality**: Disable motion blur and depth of field on low-end devices (detect via `renderer.info.memory`, frame time)
4. **Reduce resolution**: Run bloom at half resolution, upsample in final composite
5. **Limit bloom**: High bloom threshold (> 0.7) catches only bright highlights (flames, sparks) rather than the entire scene

---

## 5. LOD Systems and Asset Streaming

### Level of Detail (LOD)

LOD automatically swaps high-polygon models for lower-detail versions based on camera distance, reducing GPU work for distant objects.

```javascript
// Three.js LOD
const lod = new THREE.LOD();
lod.addLevel(highDetailKart, 0);    // High poly within 50 units
lod.addLevel(medDetailKart, 50);    // Medium poly 50–150 units
lod.addLevel(lowDetailKart, 150);   // Low poly 150–300 units
lod.addLevel(billboard, 300);       // Billboard/impostor beyond 300 units
// LOD auto-updates in render loop via lod.update(camera)
```

LOD impact per [Unity WebGL guide (DEV.to, 2025)](https://dev.to/alok_krishali/making-unity-webgl-games-run-smoothly-on-low-end-browsers-58g7):
- Reduces draw calls for distant objects
- Reduces triangle count (especially for complex car models with 10k–50k triangles)
- Reduces memory bandwidth on GPU

**For racing games**: other karts are ideal LOD candidates — up to 7 other karts on screen at varying distances. Using a 3-level LOD (10k → 2k → 500 triangles) can dramatically reduce GPU load on complex tracks.

### Track / Asset Streaming

For large open tracks:

| Technique | Description |
|-----------|-------------|
| **Sector-based loading** | Divide track into sectors; load next sector async while racing current one |
| **Object pooling** | Pre-allocate particle effects, item pickups, track decorations; reuse don't create |
| **Occlusion culling** | Don't render track segments behind buildings/walls (expensive to set up, high payoff) |
| **Frustum culling** | Three.js default: skip objects outside camera frustum |
| **Progressive loading** | Use placeholder geometry during asset load (`Suspense` with R3F or manual) |
| **Draco + KTX2** | Compressed geometry and textures; GPU decompresses natively |

For a **lap-based racing game** (not open world), sector streaming is straightforward: preload the entire track at race start (since it loops), and only stream visual decorations (crowd, vegetation) based on current track position.

### SuperTuxKart LOD Improvements (2025)

From [SuperTuxKart 2025 Development Blog](https://blog.supertuxkart.net/2025/):
- **Improved LOD and shadow mapping logic** significantly reduces pop-in
- Higher settings can **eliminate pop-in entirely** (aggressive LOD transition distances)
- x16 anisotropic filtering now default (vs. previous x2/x4) with only 1–2% FPS cost
- SSAA (render at higher-than-native resolution) available for high-end GPUs

---

## 6. Target Frame Rates and Performance Budgets

### Frame Budget Fundamentals

[Reddit r/gamedev on frame rates](https://www.reddit.com/r/gamedev/comments/z47khe/a_question_about_frame_rates/):

| Target FPS | Frame Budget | Use Case |
|-----------|-------------|----------|
| **60 fps** | **16.67 ms** | Standard target for browser games |
| **30 fps** | **33.33 ms** | Minimum acceptable; common with dynamic lighting |
| **120 fps** | **8.33 ms** | High refresh displays (desktops with 120Hz) |

The frame budget must cover: physics simulation, AI, networking, JavaScript logic, AND rendering.

### Typical Budget Breakdown (60 fps target, 16.67ms)

| System | Budget Allocation | Notes |
|--------|-----------------|-------|
| **CPU: Physics** | 2–4 ms | Amortize across ticks if needed |
| **CPU: JavaScript / game logic** | 2–3 ms | Input, AI, item effects |
| **CPU: Draw call submission** | 1–2 ms | Keep draw calls < 100 |
| **GPU: Geometry + shaders** | 4–6 ms | Instancing critical |
| **GPU: Post-processing** | 2–4 ms | Merge passes; skip on low-end |
| **Browser overhead** | 1–2 ms | Garbage collection, event loop |

**Key insight**: At < 100 draw calls (achievable with instancing), GPU time drops significantly, giving more headroom for post-processing and particles.

### Device Tier Targets

| Device Tier | Target FPS | Draw Calls | Post-FX | Particles |
|-------------|-----------|-----------|---------|----------|
| **High-end desktop** (RTX, M2) | 60+ fps | < 200 | Full pipeline | GPU compute shaders |
| **Mid-range desktop** | 60 fps | < 100 | Bloom + AA | GPGPU fragment |
| **Low-end desktop / laptop** | 30–60 fps | < 60 | AA only | CPU, < 20k |
| **Mobile browser** | 30 fps | < 50 | Minimal | CPU, < 5k |

### Profiling Tools

| Tool | Use Case |
|------|----------|
| `stats-gl` | Real-time fps, draw calls, GPU time overlay |
| `renderer.info` | Draw calls, triangles, programs, texture count |
| `Spector.js` | WebGL/WebGPU frame capture, per-draw profiling |
| Chrome DevTools → Performance | CPU timeline, GC pauses, frame timing |
| Chrome WebGPU DevTools | GPU-specific profiling for WebGPU renderer |
| GPU timing queries | Precise GPU time per render pass |

---

## 7. WebGL vs WebGPU Performance in 2025/2026

### Browser Support Status (as of March 2026)

[WebGPU hits critical mass — WebGPU.com (Dec 2025)](https://www.webgpu.com/news/webgpu-hits-critical-mass-all-major-browsers/):
- **Chrome 113+** (shipped May 2023) — Windows, macOS, Android
- **Firefox 141** (July 2025) — Windows; Firefox 145 — macOS Apple Silicon
- **Safari 26.0** (September 2025) — macOS Tahoe 26, iOS 26, iPadOS 26, visionOS 26
- **Edge**: Follows Chrome (Dawn engine)

**As of November 25, 2025**: WebGPU ships by default in all four major browsers.

> *"You can now write WebGPU code with reasonable confidence that your users will be able to run it."*
> — [WebGPU.com (December 2025)](https://www.webgpu.com/news/webgpu-hits-critical-mass-all-major-browsers/)

### Performance Benchmarks

#### GEMM / Compute Performance (SitePoint, Feb 2026)

[WebGPU vs WebGL: Performance Benchmarks for Client-Side Inference — SitePoint (Feb 2026)](https://www.sitepoint.com/webgpu-vs-webgl-inference-benchmarks/) — median of 50 iterations, FP32 matrix multiplication:

| Matrix Size | Hardware | WebGL (ms) | WebGPU (ms) | Speedup |
|-------------|----------|-----------|------------|---------|
| 512 × 512 | Apple M2 | 4.2 | 2.6 | **1.6×** |
| 512 × 512 | RTX 3060 | 2.8 | 1.5 | **1.9×** |
| 1024 × 1024 | Apple M2 | 28.5 | 8.1 | **3.5×** |
| 1024 × 1024 | RTX 3060 | 15.3 | 3.9 | **3.9×** |
| 2048 × 2048 | Apple M2 | 210 | 42 | **5.0×** |
| **2048 × 2048** | **RTX 3060** | **98** | **14.2** | **6.9×** |
| 4096 × 4096 | RTX 3060 | 720 | 89 | **8.1×** |

**Speedup is largest** (3–8×) for large, compute-intensive workloads. For small matrices (< 512×512), gain narrows to 1.5–2× due to WebGPU pipeline initialization overhead.

#### Real-World Game Rendering

[MayhemCode WebGPU benchmarks (Dec 2025)](https://www.mayhemcode.com/2025/12/gpu-acceleration-in-browsers-webgpu.html):
- **Babylon.js with WebGPU Render Bundles**: ~**10× faster** than previous WebGL rendering
- **Particle systems**: Console-quality particle counts now achievable in browsers
- **Physics simulation**: Heavy physics no longer causes frame drops

[4D Pipeline blog (Oct 2025)](https://blog.4dpipeline.com/webgpu-the-next-generation-of-browser-graphics-and-compute):
- Three.js WebGPU Compute Particles demo: **500,000 particles** fully GPU-updated in real time
- WebGPU fluid simulation: **300,000 particles** on mid-range GPU at real-time framerates
- Techniques previously limited to offline/native renderers (global illumination, path tracing) now feasible in browsers

### Key Architectural Differences

| Aspect | WebGL | WebGPU |
|--------|-------|--------|
| API generation | OpenGL ES binding | Clean-slate (Vulkan/Metal/D3D12 inspired) |
| Shader language | GLSL | WGSL (verifiably safe, text-based) |
| Compute | Fragment shader hack (textures as buffers) | First-class compute shaders with workgroups |
| GPU state management | Implicit (driver guesses state) | Explicit pipelines and bind groups |
| CPU overhead | High (driver synchronization) | Low (explicit, predictable) |
| Memory model | Implicit | Explicit (storage buffers, storage textures) |
| Multi-threading | Very limited | Compute workgroups + shared memory |

### Three.js WebGPU Renderer (r171+)

From [Utsubo 2026 guide](https://www.utsubo.com/blog/threejs-best-practices-100-tips):

```javascript
import { WebGPURenderer } from 'three/webgpu';
const renderer = new WebGPURenderer();
await renderer.init(); // Required — async GPU adapter acquisition
// Automatically falls back to WebGL 2 if WebGPU unavailable
```

- **Production-ready since r171** (Three.js version released 2024)
- **Automatic WebGL 2 fallback**: ship one codebase, handles compatibility
- **TSL (Three Shader Language)**: write shaders once, compile to WGSL (WebGPU) or GLSL (WebGL)
- **2–10× performance gains** in specific scenarios (draw-call-heavy, compute-intensive)

### When to Migrate from WebGL to WebGPU

Per [Utsubo recommendations](https://www.utsubo.com/blog/threejs-best-practices-100-tips):

| Migrate When | Stay on WebGL When |
|-------------|-------------------|
| Draw-call-heavy scenes drop frames | WebGL project runs smoothly |
| Need compute shaders (particles, physics) | Targeting older devices (pre-2023 hardware) |
| Complex post-processing chains stutter | Bundle size is critical (WebGLRenderer is smaller) |
| Want GPU-driven rendering with indirect draws | Need specific WebGL extensions |

---

## 8. SuperTuxKart Rendering Pipeline — What Can Be Adapted

### History of the STK Graphics Engine

[SuperTuxKart's 2014 graphical engine introduction](https://blog.supertuxkart.net/2014/05/introducing-supertuxkarts-new-graphical.html):
- Originally used **Irrlicht** (third-party engine since STK 0.7) with fixed pipeline rendering
- 2013 GSoC: Lauri Kasanen added **shader support** and dynamic lights
- Vincent LeJeune built a new **shader-based modern pipeline** bypassing Irrlicht's fixed pipeline
- Irrlicht retained only for asset loading and scene management
- Added: **dynamic lights**, **Image Based Lighting (IBL)**, **hand-painted texture style**

### STK 1.5 Rendering Features (2025)

From [SuperTuxKart 2025 Development Blog](https://blog.supertuxkart.net/2025/):

| Feature | Implementation |
|---------|---------------|
| **Default renderer** | OpenGL (most feature-complete) |
| **Vulkan renderer** | Available (translated to Metal via MoltenVK on iOS); approaching OpenGL parity |
| **Anisotropic filtering** | x16 default (was x2/x4) — 1–2% FPS cost only |
| **Shadow resolution** | Up to 2048 (high-end setting) |
| **Shadow softness** | Percentage-Closer Soft Shadows (PCSS) option |
| **Supersampling (SSAA)** | Render above native resolution for high-end GPUs |
| **LOD system** | Improved LoD + shadow mapping; near-eliminates pop-in at high settings |
| **Frame limiter** | Configurable 30–1000 FPS (default 120) |
| **Spotlight** | New spotlight for night tracks |
| **Spawn animations** | Parachute and bubblegum with new spawn animation |
| **Benchmark mode** | Built-in benchmark for settings comparison |

### What the STK Rendering Pipeline Demonstrates

| STK Technique | Browser Adaptation |
|--------------|-------------------|
| **Shader-based pipeline** (bypass fixed function) | `MeshStandardMaterial` / `ShaderMaterial` / TSL |
| **IBL (Image Based Lighting)** | `RGBELoader` + `PMREMGenerator` for environment maps |
| **Dynamic lights** | Three.js `PointLight`, `SpotLight` with shadow maps |
| **Configurable quality tiers** | Detect device capability; offer Low/Medium/High presets |
| **LOD + shadow LOD** | Three.js `LOD` + tiered shadow map resolution |
| **PCSS soft shadows** | Three.js `PCFSoftShadowMap` (equivalent quality) |
| **Vulkan renderer** | WebGPU renderer (same generation API philosophy) |
| **Irrlicht for asset loading** | Three.js `GLTFLoader`, `DRACOLoader` (equivalent role) |
| **Hand-painted texture style** | Toon materials + baked ambient occlusion maps |

### STK Open-Source Code

SuperTuxKart's complete source code (GPL v3) is at [github.com/supertuxkart/stk-code](https://github.com/supertuxkart/stk-code). Key directories for rendering:
- `src/graphics/` — shader sources, rendering pipeline
- `data/shaders/` — GLSL shader files for tracks and karts
- `data/tracks/` — track assets with LOD configuration

---

## 9. Synthesis: Recommended Rendering Architecture

### Technology Stack

| Component | Recommendation | Rationale |
|-----------|---------------|-----------|
| **Renderer** | `WebGPURenderer` with WebGL 2 fallback | Future-proof; production-ready since r171 |
| **Shader authoring** | TSL (Three Shader Language) | Single source, targets both backends |
| **Style** | Toon shading + bloom | Kart game aesthetic; performant |
| **Particles** | WebGPU compute shaders (primary) / GPGPU fragment (fallback) | Millions on WebGPU; 500k on WebGL |
| **Post-processing** | pmndrs/postprocessing — bloom + FXAA merged | Minimum passes; most impact |
| **LOD** | Three.js `LOD` on all karts and track decorations | 3 levels; eliminates pop-in |
| **Asset format** | GLTF + Draco geometry + KTX2 textures | Industry standard; GPU-native decompression |
| **Draw calls** | Target < 100 via InstancedMesh + geometry merge | Golden rule |

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    SCENE GRAPH                               │
│  Track (merged static geometry — 1 draw call)               │
│  Karts (InstancedMesh or LOD per kart model)                │
│  Particles (WebGPU compute → InstancedMesh render)          │
│  Lights (3 max: sun + 2 dynamic)                            │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│               WebGPURenderer (r171+)                         │
│   ↳ Automatic WebGL 2 fallback                               │
│   ↳ TSL shaders compile to WGSL or GLSL                      │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│            POST-PROCESSING (pmndrs/postprocessing)            │
│  Pass 1: Render scene → offscreen target                     │
│  Pass 2: EffectPass [Bloom + FXAA + Color Grading]          │
│          (single combined pass = 1 extra draw)               │
│  Pass 3 (optional, high-end): MotionBlur                     │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│                    CANVAS OUTPUT                              │
└─────────────────────────────────────────────────────────────┘
```

### Performance Tier Configuration

```javascript
const qualityPresets = {
  low: {
    shadowMapSize: 512,
    particles: 5000,
    postProcessing: false,
    lodDistances: [0, 30, 100],
    anisotropy: 2
  },
  medium: {
    shadowMapSize: 1024,
    particles: 50000,
    postProcessing: ['bloom', 'fxaa'],
    lodDistances: [0, 50, 200],
    anisotropy: 8
  },
  high: {
    shadowMapSize: 2048,
    particles: Infinity, // WebGPU compute
    postProcessing: ['bloom', 'motionBlur', 'fxaa', 'dof'],
    lodDistances: [0, 80, 300],
    anisotropy: 16
  }
};
```

---

## Sources

- [100 Three.js Tips That Actually Improve Performance (2026) — Utsubo](https://www.utsubo.com/blog/threejs-best-practices-100-tips)
- [WebGPU vs. WebGL: Performance Benchmarks for Client-Side Inference — SitePoint (Feb 2026)](https://www.sitepoint.com/webgpu-vs-webgl-inference-benchmarks/)
- [GPU Acceleration in Browsers: WebGPU Performance Benchmarks — MayhemCode (Dec 2025)](https://www.mayhemcode.com/2025/12/gpu-acceleration-in-browsers-webgpu.html)
- [WebGPU Hits Critical Mass: All Major Browsers Now Ship It — WebGPU.com (Dec 2025)](https://www.webgpu.com/news/webgpu-hits-critical-mass-all-major-browsers/)
- [WebGPU: The Next Generation of Browser Graphics and Compute — 4D Pipeline (Oct 2025)](https://blog.4dpipeline.com/webgpu-the-next-generation-of-browser-graphics-and-compute)
- [The Best of WebGPU in January 2025 — WebGPU Experts](https://www.webgpuexperts.com/best-webgpu-updates-january-2025)
- [From WebGL to WebGPU: A Reality Check — ACM DL (Nov 2025)](https://dl.acm.org/doi/10.1145/3730567.3764504)
- [Custom Toon Shader in Three.js Tutorial — Maya Nedeljkovich](https://www.maya-ndljk.com/blog/threejs-basic-toon-shader)
- [Three.js MeshToonMaterial — Three.js Documentation](https://threejs.org/docs/#api/materials/MeshToonMaterial)
- [Three.js WebGL Toon Materials Example](https://threejs.org/examples/#webgl_materials_variations_toon)
- [Comicbook/Cel Shader for Three.js — Stack Overflow](https://stackoverflow.com/questions/35995261/comicbook-shader-for-real-time-with-three-js-cel-shading)
- [Stylized Cartoon Shader in THREE.JS — YouTube](https://www.youtube.com/watch?v=V5UllFImvoE)
- [Post-processing — Three.js Journey](https://threejs-journey.com/lessons/post-processing)
- [Post-processing Guide — TresJS](https://post-processing.tresjs.org/guide/)
- [Crafting a Dreamy Particle Effect with Three.js and GPGPU — Codrops (Dec 2024)](https://tympanus.net/codrops/2024/12/19/crafting-a-dreamy-particle-effect-with-three-js-and-gpgpu/)
- [WebGPU Particle System on Reddit — r/GraphicsProgramming (May 2025)](https://www.reddit.com/r/GraphicsProgramming/comments/1kgsfyu/i_built_this_interactive_webgpu_particle_system/)
- [One Draw Call, Massive Crowd — Three.js Forum (Feb 2026)](https://discourse.threejs.org/t/one-draw-call-massive-crowd-performance-engineering-in-three-js/89928)
- [How to Optimize Three.js Rendering — Three.js Forum](https://discourse.threejs.org/t/how-can-i-optimise-my-three-js-rendering/42251)
- [Performance Tips — Three.js Journey](https://threejs-journey.com/lessons/performance-tips)
- [Making Unity WebGL Games Run Smoothly on Low-End Browsers — DEV.to (Apr 2025)](https://dev.to/alok_krishali/making-unity-webgl-games-run-smoothly-on-low-end-browsers-58g7)
- [WebGL Game Engines: High-Performance Browser Games — KiteMetric](https://kitemetric.com/blogs/webgl-game-engines-high-performance-browser-games)
- [A Question About Frame Rates — Reddit r/gamedev](https://www.reddit.com/r/gamedev/comments/z47khe/a_question_about_frame_rates/)
- [Introducing SuperTuxKart's New Graphical Engine — STK Development Blog (May 2014)](https://blog.supertuxkart.net/2014/05/introducing-supertuxkarts-new-graphical.html)
- [SuperTuxKart Development Blog: 2025](https://blog.supertuxkart.net/2025/)
- [STK with --render-driver=vulkan — GitHub Issue #4815](https://github.com/supertuxkart/stk-code/issues/4815)
- [SuperTuxKart Source Code — GitHub](https://github.com/supertuxkart/stk-code)
- [How to Optimize Project for Low-End Computers — Reddit r/threejs](https://www.reddit.com/r/threejs/comments/skk0f3/how_to_optimize_project_for_lowend_computers/)
- [GPU Particle System Rain/Snow Discussion — GitHub Issue (Dec 2024)](https://github.com/erichlof/THREE.js-PathTracing-Renderer/issues/82)
- [Improve Game FPS With LOD Groups in Unity — YouTube (Oct 2025)](https://www.youtube.com/watch?v=olT3Ke1d_Zo)
