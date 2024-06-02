---
layout: ../layouts/BlogPost.astro
title: Noise Generation in Shaders Using Noise Functions for Texture Generation and Procedural Content
slug: "noise-generation-in-shaders"
description: >-
  Noise generation is a fundamental technique in computer graphics used to create textures and procedural content. By employing noise functions within shaders, developers can produce a wide variety of visual effects, ranging from simple textures to complex natural phenomena.
tags:
  - technical
  - shader
  - gamedev
added: "Jun 2 2024"
---

Noise generation is a fundamental technique in computer graphics used to create textures and procedural content. By employing noise functions within shaders, developers can produce a wide variety of visual effects, ranging from simple textures to complex natural phenomena. This article explores the use of noise functions in shaders, detailing their implementation and application in texture generation and procedural content creation.

## Introduction to Noise Functions

Noise functions generate pseudo-random values that can be used to simulate natural variations in textures. These functions are deterministic, meaning the same input will always produce the same output, which is crucial for consistency in graphics rendering.

### Common Noise Functions

- Perlin Noise: Smooth gradient noise often used for natural textures like terrain and clouds.
- Simplex Noise: An improved version of Perlin noise, more computationally efficient and with fewer artifacts.
- Worley Noise: Generates cell-like patterns, useful for simulating textures like stone and skin.
- Value Noise: Produces smooth gradients similar to Perlin noise but with a simpler implementation.

### Implementing Noise Functions in GLSL

**Perlin Noise**
Perlin noise is a gradient noise function that produces smooth, continuous patterns. Here’s an implementation in GLSL:

```c
// Perlin Noise Function
vec2 fade(vec2 t) {
    return t * t * t * (t * (t * 6.0 - 15.0) + 10.0);
}

float grad(int hash, vec2 p) {
    int h = hash & 7;
    float u = h < 4 ? p.x : p.y;
    float v = h < 4 ? p.y : p.x;
    return ((h & 1) == 0 ? u : -u) + ((h & 2) == 0 ? v : -v);
}

float perlinNoise(vec2 p) {
    vec2 P = floor(p);
    vec2 f = fade(fract(p));
    int A = int(P.x) + int(P.y) * 57;
    int B = A + 57;
    return mix(mix(grad(A, p - P), grad(B, p - P - vec2(1.0, 0.0)), f.x),
               mix(grad(A + 1, p - P - vec2(0.0, 1.0)), grad(B + 1, p - P - vec2(1.0, 1.0)), f.x), f.y);
}
```

## Simplex Noise

Simplex noise is an improvement over Perlin noise, providing better visual quality and performance:

```c
// Simplex Noise Function
vec3 mod289(vec3 x) {
    return x - floor(x * (1.0 / 289.0)) * 289.0;
}

vec3 permute(vec3 x) {
    return mod289(((x*34.0)+1.0)*x);
}

float simplexNoise(vec2 v) {
    const vec4 C = vec4(0.211324865405187, 0.366025403784439, -0.577350269189626, 0.024390243902439);
    vec2 i  = floor(v + dot(v, C.yy) );
    vec2 x0 = v -   i + dot(i, C.xx);
    vec2 i1;
    i1.x = step( x0.y, x0.x );
    i1.y = 1.0 - i1.x;
    vec4 x12 = x0.xyxy + C.xxzz;
    x12.xy -= i1;
    i = mod289(i);
    vec3 p = permute( permute( i.y + vec3(0.0, i1.y, 1.0 ))
            + i.x + vec3(0.0, i1.x, 1.0 ));
    vec3 m = max(0.5 - vec3(dot(x0,x0), dot(x12.xy,x12.xy), dot(x12.zw,x12.zw)), 0.0);
    m = m*m ;
    m = m*m ;
    vec3 x = 2.0 * fract(p * C.www) - 1.0;
    vec3 h = abs(x) - 0.5;
    vec3 ox = floor(x + 0.5);
    vec3 a0 = x - ox;
    m *= 1.79284291400159 - 0.85373472095314 * ( a0*a0 + h*h );
    vec3 g;
    g.x  = a0.x  * x0.x  + h.x  * x0.y;
    g.yz = a0.yz * x12.xz + h.yz * x12.yw;
    return 130.0 * dot(m, g);
}
```

## Worley Noise

Worley noise, or cellular noise, generates cell-like patterns:

```c
// Worley Noise Function
float worleyNoise(vec2 uv) {
    vec2 gv = fract(uv) - 0.5;
    vec2 id = floor(uv);
    float minDist = 1.0;
    for (int j = -1; j <= 1; j++) {
        for (int i = -1; i <= 1; i++) {
            vec2 offset = vec2(float(i), float(j));
            vec2 neighbor = offset + hash2(id + offset);
            vec2 point = gv - neighbor;
            float dist = dot(point, point);
            minDist = min(minDist, dist);
        }
    }
    return minDist;
}
```

## Applications of Noise in Procedural Texturing

**Terrain Generation**
Noise functions are commonly used to generate realistic terrain. By combining multiple layers of noise at different scales, you can create complex and detailed landscapes.

```c
float terrainHeight(vec2 uv) {
    float height = 0.0;
    float frequency = 1.0;
    float amplitude = 1.0;
    for (int i = 0; i < 5; i++) {
        height += amplitude * simplexNoise(uv * frequency);
        frequency *= 2.0;
        amplitude *= 0.5;
    }
    return height;
}
```

## Wood Grain Texture

Simulate wood grain by combining noise with trigonometric functions:

```c
float woodGrain(vec2 uv) {
    float noise = perlinNoise(uv * 10.0);
    float grain = sin(uv.x * 20.0 + noise * 5.0);
    return grain;
}

void main() {
    vec2 uv = gl_FragCoord.xy / resolution.xy;
    float wood = woodGrain(uv);
    gl_FragColor = vec4(vec3(wood), 1.0);
}
```

## Marble Texture

Create a marble texture by blending noise with sine functions to simulate the veining:

```c
float marble(vec2 uv) {
    float noise = perlinNoise(uv * 5.0);
    float veins = sin(uv.y * 10.0 + noise * 5.0);
    return veins;
}

void main() {
    vec2 uv = gl_FragCoord.xy / resolution.xy;
    float marblePattern = marble(uv);
    gl_FragColor = vec4(vec3(marblePattern), 1.0);
}
```

## Cloud Texture

Simulate clouds using layered Perlin noise:

```c
float clouds(vec2 uv) {
    float cloud = 0.0;
    float frequency = 1.0;
    float amplitude = 1.0;
    for (int i = 0; i < 5; i++) {
        cloud += amplitude * perlinNoise(uv * frequency);
        frequency *= 2.0;
        amplitude *= 0.5;
    }
    return cloud;
}

void main() {
    vec2 uv = gl_FragCoord.xy / resolution.xy;
    float cloudPattern = clouds(uv);
    gl_FragColor = vec4(vec3(cloudPattern), 1.0);
}
```

## Tips for Using Noise Functions in Shaders

- Optimize Performance: Noise functions can be computationally expensive. Optimize by minimizing function calls and using efficient implementations.
- Use Multiple Layers: Combine multiple layers of noise at different frequencies and amplitudes to create more complex and interesting patterns.
- Experiment with Parameters: Adjust noise function parameters to achieve different effects. Small changes can lead to significant variations in the final texture.
- Blend Different Noise Types: Combining different types of noise functions (e.g., Perlin and Worley) can produce unique textures.

## Conclusion

Noise generation in shaders is a powerful tool for creating procedural textures and content. By understanding and implementing various noise functions, such as Perlin, Simplex, and Worley noise, you can create a wide range of natural-looking textures and effects. Experiment with different techniques, optimize your shaders for performance, and unleash your creativity to generate stunning procedural content for your graphics applications.
