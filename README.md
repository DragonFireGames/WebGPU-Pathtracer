# WebGPU Path Tracer

A high-performance **Monte Carlo path tracer** running on the GPU using WebGPU and WGSL. This project brings realistic physically-based rendering to the web with real-time interactive editing, physics simulation, and video rendering capabilities.

**Live Demo:** [https://webgpu-pathtracer.onrender.com](https://webgpu-pathtracer.onrender.com)

---

## Table of Contents

- [Overview](#overview)
- [Pathtracer Logic](#pathtracer-logic)
- [Features](#features)
- [System Requirements](#system-requirements)
- [Getting Started](#getting-started)
- [Scene Editing](#scene-editing)
- [Advanced Usage](#advanced-usage)

---

## Overview

This is a **progressive path tracer** that accumulates samples over many frames to produce photorealistic images. It leverages WebGPU's compute shaders (WGSL) to parallelize ray casting and shading across thousands of GPU threads, making it practical for interactive 3D scene design directly in a web browser.

**Key Technologies:**
- **WebGPU**: GPU compute and graphics API
- **WGSL**: WebGPU Shading Language
- **gl-matrix**: 3D math library
- **Cannon.js**: Physics simulation
- **Lil-GUI**: Parameter inspector

---

## Pathtracer Logic

### 1. Monte Carlo Ray Tracing

The pathtracer traces rays from the camera into the scene and recursively bounces them off surfaces. Each bounce samples the incoming light contribution at that point.

**Per-pixel algorithm:**
For each pixel: For each sample (multiple times per frame): 
1. Generate ray from camera through pixel 
2. Trace ray until it hits geometry or leaves scene
3. At each surface, evaluate: - Material properties (albedo, roughness, metallic, IOR) - Incoming light from environment or emissive surfaces - Scatter direction (using importance sampling) 
4. Accumulate radiance and bounce to next surface 
5. After N bounces, terminate ray and add to pixel value Average all samples for final pixel color


The key insight is that by averaging many random paths, the noise (variance) decreases over time, revealing the true global illumination solution.

### 2. Acceleration Structure: Bounding Volume Hierarchy (BVH)

To efficiently find ray-geometry intersections, the tracer builds a **two-level BVH**:

- **Top-Level (TLAS)**: Organizes scene primitives (spheres, cubes, cylinders, etc.) and mesh instances
- **Bottom-Level (BLAS)**: Each mesh/model has its own BVH for fast triangle intersection

**BVH Construction:**
- Uses **Surface Area Heuristic (SAH)** to recursively partition geometry
- Minimizes ray traversal cost by balancing tree structure
- Supports dynamic updates as objects move


### 3. Material & Shading

Each surface is characterized by **Disney/Principled BSDF** parameters:

| Parameter | Meaning |
|-----------|---------|
| `roughness` | Surface smoothness (0 = mirror, 1 = diffuse) |
| `metallic` | How metal-like the surface is (0 = dielectric, 1 = metal) |
| `IOR` | Index of refraction (glass, plastic, etc.) |
| `transmission` | Transparency / refractive opacity |
| `anisotropic` | Directional roughness (e.g., brushed metal) |
| `sheen` | Fabric-like secondary lobe |
| `clearcoat` | Car-paint style multi-layer |
| `subsurface` | Light scattering into material |

**Texture Support:**
- **Albedo** - Base color
- **Normal** - Per-pixel surface orientation (normal mapping)
- **Roughness** - Per-pixel surface smoothness
- **Height** - Parallax Occlusion Mapping (POM) for surface detail
- **Metallic** - Per-pixel metalness

### 4. Sampling & Noise Reduction

At each surface point, the tracer uses **importance sampling** to reduce variance:

- **Lambertian (Diffuse)**: Sample hemisphere weighted by cosine
- **Specular (Reflections)**: Sample around specular direction using GGX microfacet distribution
- **Refractive (Transmissions)**: Apply Snell's law for glass/plastic
- **Emissive**: Direct light from glowing surfaces

### 5. Camera Effects

**Advanced camera features:**
- **Depth of Field**: Simulates lens aperture with jittered lens sampling
- **Motion Blur**: Via animated camera keyframes and temporal accumulation
- **Exposure**: Post-process tone mapping control

---

## Features

### ✅ Supported Geometries

| Primitive | Supported | Notes |
|-----------|-----------|-------|
| **Sphere** | ✓ | Analytic intersection, fast |
| **Cube** | ✓ | Axis-aligned boxes |
| **Cylinder** | ✓ | Tapered cones supported |
| **Torus** | ✓ | Donut-shaped geometry |
| **Plane** | ✓ | Infinite ground plane |
| **Mesh (OBJ)** | ✓ | BVH-accelerated triangle mesh |

### ✅ Material System

- **Disney/Principled BSDF** with 14+ parameters
- **Texture Maps**: Albedo, Normal, Roughness, Metallic, Height
- **Parallax Occlusion Mapping (POM)** for surface detail
- **Emission**: Emissive surfaces for area lighting
- **Subsurface Scattering**: Light penetration into translucent materials

### ✅ Lighting

- **Image-Based Lighting (IBL)**: HDR environment maps via `.hdr` format
- **Emissive Materials**: Self-illuminating surfaces
- **Multi-bounce Global Illumination**: Accurate light reflections
- **Configurable Bounce Depth**: 1-32 bounces per ray

### ✅ Editor Features

- **3D Viewport**: Orbit/pan/zoom camera controls
  - **LMB**: Orbit around target
  - **RMB / Shift+LMB**: Pan
  - **Scroll**: Zoom
- **Scene Hierarchy**: Add/delete/select/edit objects
- **Inspector Panel**: Real-time material and geometry editing
- **Gizmos**: Transform (translate/rotate/scale) directly in viewport
- **Keyframe Animation**: Record camera and object animations
- **Asset Manager**: Manage materials, textures, and models
- **Preset Scenes**: Pre-built example scenes

### ✅ Rendering Modes

**1. Real-time Viewport**
- Progressive accumulation with live editing
- Instant feedback as you adjust parameters
- Continues improving quality as samples accumulate

**2. High-Quality Render**
- Configurable resolution (up to 4K+)
- Per-pixel sample count control
- Bounce depth override
- Progress bar with live preview

**3. Animation Render**
- Record camera keyframes and animate objects
- Export as WebM video
- Per-frame control of samples
- Frame rate configuration

**4. Physics Simulation**
- Cannon.js integration for rigid body dynamics
- Drop objects with gravity and collision
- Render physically-accurate motion blur

### ✅ Export & I/O

- **Render Output**: Save high-quality JPEG/PNG images
- **Video Export**: Render animations to WebM format
- **OBJ Import**: Load 3D models from external files
- **Material Library**: Save/load custom materials
- **HDR Support**: Load `.hdr` environment maps

---

## System Requirements

### Browser & GPU

- **Browser**: Chrome/Edge 113+, Firefox 128+ (WebGPU support)
- **GPU**: Discrete GPU recommended (dedicated VRAM)
  - **NVIDIA**: Maxwell (GTX 750) or newer
  - **AMD**: GCN or newer
  - **Apple**: M1/M2 or newer
- **RAM**: 2GB+ available
- **VRAM**: 1GB+ recommended

### Supported Platforms

| Platform | Status |
|----------|--------|
| Windows 11 (Chrome/Edge) | ✓ Full support |
| macOS 13+ (Safari/Chrome) | ✓ Full support |
| Linux (Chrome) | ✓ Full support |
| iPad (Safari) | ⚠ Limited (small viewport) |
| Mobile Phones | ⚠ Limited (small/slow) |

---

## Getting Started

### Online Demo

No installation required! Visit the live demo at:
**[https://webgpu-pathtracer.onrender.com](https://webgpu-pathtracer.onrender.com)**

### Local Development

1. **Clone the repository:**
   ```bash
   git clone https://github.com/DragonFireGames/WebGPU-Pathtracer.git
   cd WebGPU-Pathtracer