---
layout: ../layouts/BlogPost.astro
title: Cel-Shading Techniques Achieving Cartoon-Like Visuals with Cel-Shading
slug: "cel-shading-techniques-achieving-cartoon-like"
description: >-
  Cel-shading, also known as toon shading, is a non-photorealistic rendering technique that gives 3D models a flat, cartoon-like appearance. This style mimics the look of hand-drawn animation, making it popular in video games, animation, and graphic design. In this blog, we'll explore the key concepts of cel-shading, how to implement it in shaders, and various techniques to achieve different cartoon-like effects.
tags:
  - technical
  - shader
  - gamedev
added: "Jun 2 2024"
---

Cel-shading, also known as toon shading, is a popular non-photorealistic rendering technique used to create cartoon-like visuals in games and animations. This technique simplifies shading by using flat colors and bold outlines, giving a hand-drawn, comic book-like appearance to 3D models. This guide will explore the fundamental concepts of cel-shading, its implementation in shaders, and practical tips to achieve captivating cartoon-like visuals.

## Introduction to Cel-Shading

Cel-shading mimics the look of traditional 2D animation and comics, characterized by:

- Flat Color Areas: Instead of smooth gradients, colors are rendered in discrete bands or flat shades.
- Bold Outlines: Thick black lines outline the edges of objects, emphasizing their shapes and contours.
- Simplified Lighting: Lighting effects are simplified to enhance the stylized look, focusing on sharp contrasts between light and shadow.

## Implementing Cel-Shading in Shaders

To achieve cel-shading, we need to handle two main aspects: shading the surfaces with flat colors and generating bold outlines.

### Flat Color Areas

The key to flat color shading is quantizing the lighting calculations to create discrete steps. This can be achieved by dividing the light intensity into a few distinct levels.

### Basic Cel-Shading Shader

**Vertex Shader:**

```c
#version 330 core
layout(location = 0) in vec3 aPos;
layout(location = 1) in vec3 aNormal;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;
uniform vec3 lightDir;

out vec3 Normal;
out vec3 LightDir;

void main() {
    gl_Position = projection * view * model * vec4(aPos, 1.0);
    Normal = mat3(transpose(inverse(model))) * aNormal; // Transform normals to world space
    LightDir = normalize(lightDir); // Normalize light direction
}
```

**Fragment Shader:**

```c
#version 330 core
in vec3 Normal;
in vec3 LightDir;

out vec4 FragColor;

uniform vec3 baseColor;

void main() {
    float intensity = max(dot(normalize(Normal), LightDir), 0.0);
    // Quantize the intensity to create discrete steps
    float levels = 3.0;
    intensity = floor(intensity * levels) / levels;

    vec3 color = baseColor * intensity;
    FragColor = vec4(color, 1.0);
}
```

In this shader:

- The vertex shader calculates the normal and light direction in world space.
- The fragment shader computes the light intensity and quantizes it into discrete levels to achieve the flat shading effect.

### Bold Outlines

To create bold outlines, we can use two common techniques: edge detection and backface culling.

Backface Culling Method
This method involves rendering the backfaces of the model with a thick outline. The vertices are pushed along their normals to create the effect of an outline.

**Outline Vertex Shader:**

```c
#version 330 core
layout(location = 0) in vec3 aPos;
layout(location = 1) in vec3 aNormal;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;
uniform float outlineThickness;

void main() {
    vec3 normal = normalize(mat3(model) * aNormal);
    vec3 position = aPos + normal * outlineThickness;
    gl_Position = projection * view * model * vec4(position, 1.0);
}
```

**Outline Fragment Shader:**

```c
#version 330 core
out vec4 FragColor;

void main() {
    FragColor = vec4(0.0, 0.0, 0.0, 1.0); // Black outline
}
```

## Combining Flat Shading and Outlines

To achieve the final cel-shaded look, we render the scene twice: first for the outlines and then for the cel-shading.

Main Rendering Loop

```cpp
void render() {
    // Render outlines
    glEnable(GL_CULL_FACE);
    glCullFace(GL_FRONT); // Cull front faces
    glUseProgram(outlineShader);
    setUniformsForOutline();
    renderModel();

    // Render cel-shading
    glCullFace(GL_BACK); // Cull back faces
    glUseProgram(celShader);
    setUniformsForCelShading();
    renderModel();
}
```

In this rendering loop:

- We first render the outlines by culling the front faces and using the outline shader.
- Then, we render the cel-shading by culling the back faces and using the cel-shading shader.

## Practical Tips for Effective Cel-Shading

### 1. Design with Exaggerated Shapes

Cel-shading works best with bold, exaggerated shapes. Simplify your models and focus on strong, clear silhouettes to enhance the cartoon-like appearance.

### 2. Use Saturated Colors

Bright, saturated colors enhance the stylized look of cel-shading. Avoid complex textures and focus on flat, vibrant colors that pop.

### 3. Enhance with Additional Effects

Consider adding post-processing effects like bloom or vignette to enhance the overall visual style. A subtle paper texture overlay can also add to the hand-drawn feel.

### 4. Pay Attention to Lighting

Even though cel-shading simplifies lighting, it's important to place lights thoughtfully to highlight your characters' features and maintain the cartoon aesthetic.

### 5. Iterate and Experiment

Experiment with different quantization levels, outline thicknesses, and color palettes. Iteration is key to finding the perfect balance for your game's visual style.

## Conclusion

Cel-shading is a powerful technique for creating stylized, cartoon-like visuals in games. By quantizing lighting into flat color bands and adding bold outlines, you can achieve a distinctive look that sets your game apart. Understanding and implementing cel-shading in shaders, combined with thoughtful character design and lighting, will enable you to create captivating and memorable game characters. Dive into the world of cel-shading and unleash your creativity to bring your stylized visions to life.
