# Portfolio City — Technical Architecture Documentation

## 1. System Overview

**Portfolio City** is a real-time interactive 3D portfolio environment built using **Three.js (r128)** and vanilla JavaScript. The application renders a fully navigable open-world city representing structured professional metadata as spatially organized interactive entities.

The system integrates:

* WebGL rendering pipeline (via Three.js)
* Real-time IST-based astronomical lighting simulation
* Dynamic camera orbit mechanics
* Physics-inspired vehicle controller
* Spatial zone detection engine
* Interactive UI overlay system (HUD + popups)
* Procedural geometry generation
* Dynamic minimap renderer (Canvas 2D)

---

## 2. Rendering Pipeline

### 2.1 Renderer Configuration

```js
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
renderer.toneMapping = THREE.ReinhardToneMapping;
renderer.toneMappingExposure = 1.2;
```

### Key Rendering Decisions

* **Tone Mapping**: Reinhard tone mapping for HDR-style light compression
* **Shadow Mapping**: PCFSoftShadowMap for softer penumbra
* **Fog Model**: `THREE.FogExp2` exponential fog for atmospheric depth
* **Physically-based materials**: MeshStandardMaterial with controlled metalness/roughness

---

## 3. Scene Graph Architecture

### 3.1 Hierarchical Structure

```
Scene
 ├── Ground Plane
 ├── Road Network
 ├── Streetlights
 ├── Filler Buildings
 ├── Zone Buildings (Interactive)
 │     ├── Mesh
 │     ├── Edge Geometry
 │     ├── Emissive Cap
 │     ├── Point Light
 │     └── Label Sprite
 ├── Car Group
 │     ├── Body
 │     ├── Wheels
 │     ├── Cabin
 │     ├── Lights
 │     └── Underglow
 ├── Sun / Moon System
 ├── Star Field
 └── Particle System
```

All interactive buildings store metadata in:

```js
mesh.userData = { zone: z };
```

This enables spatial → semantic mapping.

---

## 4. Real-Time Astronomical Day/Night Engine

### 4.1 Time Normalization

IST is computed from UTC offset:

```js
const utcMs = now.getTime() + now.getTimezoneOffset() * 60000;
const istMs = utcMs + (5.5 * 3600000);
```

### 4.2 Solar Elevation Model

* Sunrise: 06:00
* Sunset: 18:00
* Elevation modeled as sinusoidal interpolation

```js
Math.sin(((h - 6) / 12) * Math.PI)
```

### 4.3 Dynamic Components

| Component     | Behavior                                        |
| ------------- | ----------------------------------------------- |
| Sun Light     | Directional light intensity scaled by elevation |
| Moon Light    | Activated during negative elevation             |
| Sky Color     | Keyframe interpolated color blending            |
| Fog           | Synced to sky gradient                          |
| Ambient Light | Intensity varies across 24h                     |
| Stars         | Opacity proportional to negative elevation      |
| Streetlights  | Enabled below threshold elevation               |

This system updates every animation frame.

---

## 5. Vehicle Controller Engine

### 5.1 State Model

```js
const carState = {
  speed,
  maxSpeed,
  accel,
  friction,
  turnSpeed,
  angle
}
```

### 5.2 Movement Equation

Forward vector:

```js
x += Math.sin(angle) * speed;
z += Math.cos(angle) * speed;
```

### 5.3 Friction Model

```js
speed *= friction;
```

### 5.4 Auto-Navigation Mode

Waypoint-based steering:

```js
desiredAngle = atan2(target.x - pos.x, target.z - pos.z)
```

Angle normalization ensures smooth shortest-path rotation.

---

## 6. Spatial Zone Detection Engine

Axis-Aligned Bounding Box (AABB) collision detection:

```js
if (
  cx > px - bw/2 &&
  cx < px + bw/2 &&
  cz > pz - bd/2 &&
  cz < pz + bd/2
)
```

Used for:

* Interaction triggers
* Popup activation
* Cooldown gating
* Contextual HUD hints

---

## 7. Procedural City Generation

### 7.1 Roads

Box geometries with dashed centerlines and kerb geometry.

### 7.2 Filler Buildings

Randomized:

* Height
* Width
* Depth
* HSL-based color
* Window emissive intensity

Ensures non-repetitive skyline topology.

### 7.3 Zone Buildings

Each zone dynamically constructs:

* Foundation slab
* Main volume
* Edge wireframe
* Emissive roof cap
* Antenna beacon
* Window grids (4 sides)
* PointLight halo
* Canvas-generated sprite label

This forms a modular architectural blueprint.

---

## 8. Particle Systems

### 8.1 Floating Ambient Particles

* BufferGeometry
* Custom position + velocity arrays
* Vertex-colored PointsMaterial

### 8.2 Exhaust System

Per-frame spawned sphere meshes

* Velocity vectors derived from car heading
* Lifetime-based fade-out

---

## 9. Camera Orbit System

State-driven spherical coordinates:

```js
theta  // horizontal angle
phi    // vertical angle
radius // zoom distance
```

Smoothed interpolation between target and actual values ensures:

* Drag-based orbit
* Scroll-based zoom
* Touch compatibility

---

## 10. Minimap Subsystem

Separate 2D Canvas rendering layer.

Projection:

```js
screenX = (worldX + offsetX) * scale
```

Renders:

* Road overlays
* Zone bounding rectangles
* Car position
* Car orientation vector

This avoids additional WebGL overhead.

---

## 11. Performance Considerations

* Limited geometry segmentation
* Instancing avoided due to zone-specific metadata
* Tone mapping exposure dynamically adjusted
* Particle count capped (300 ambient)
* Shadow camera bounds manually optimized

Target: 60 FPS on mid-tier integrated GPUs.

---

## 12. Interaction Model

| Input             | Action       |
| ----------------- | ------------ |
| WASD / Arrow Keys | Drive        |
| Mouse Drag        | Orbit camera |
| Scroll            | Zoom         |
| Enter             | Open zone    |
| Escape            | Close popup  |

All UI overlays are HTML/CSS, separated from WebGL canvas for layout efficiency.

---

## 13. Architectural Philosophy

This project treats professional metadata as spatially navigable state.

Instead of:

```
Resume → Static Document
```

The system implements:

```
Resume → Semantic Data Model → Spatial World Representation → Interactive Rendering Layer
```

Each zone is effectively a 3D API endpoint mapped to structured metadata.

---

## 14. Technology Stack

* Three.js r128
* WebGL
* Vanilla JavaScript (ES6)
* HTML5 Canvas
* CSS3 (HUD + overlays)
* Procedural geometry
* Real-time astronomical simulation

---

## 15. Future Enhancements

* InstancedMesh optimization
* Post-processing (Bloom, SSAO)
* GLTF asset pipeline
* Spatial audio
* Multiplayer ghost navigation
* WebGPU migration

---

## Conclusion

Portfolio City is a real-time 3D semantic rendering engine built to transform structured professional data into an interactive navigable simulation.

It is not a webpage.
It is a state-driven spatial system.

Engineered as an experiential interface over a resume data model.
