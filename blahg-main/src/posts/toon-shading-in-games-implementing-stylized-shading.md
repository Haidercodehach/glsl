---
layout: ../layouts/BlogPost.astro
title: Toon Shading in Games Implementing Stylized Shading for Game Characters
slug: "toon-shading-in-games-implementing-stylized-shading"
description: >-
  Toon shading, also known as cel shading, is a non-photorealistic rendering technique used to give 3D models a flat, cartoon-like appearance. This technique is widely used in games to achieve a stylized look that mimics traditional hand-drawn animations.
tags:
  - technical
  - shader
  - gamedev
added: "Jun 2 2024"
---

Toon shading, also known as cel shading, is a non-photorealistic rendering technique used to give 3D models a flat, cartoon-like appearance. This technique is widely used in games to achieve a stylized look that mimics traditional hand-drawn animations. In this blog, we'll explore the fundamentals of toon shading, its implementation in shaders, and tips for creating captivating stylized game characters.

## Introduction to Toon Shading

Toon shading simplifies the color gradient on surfaces to create distinct bands of color, giving the appearance of flat shading with sharp edges. This effect is often combined with bold outlines to emphasize the shapes and contours of characters and objects.

### Key Characteristics of Toon Shading

- Flat Color Bands: Unlike smooth gradients in traditional shading, toon shading uses discrete color steps to create a flat, layered look.
- Bold Outlines: To enhance the cartoon effect, objects are often outlined with thick, dark lines.
- Simplified Lighting: Lighting calculations are simplified to produce the flat color bands and reduce complexity.

## Implementing Toon Shading in Shaders

### 1. Flat Color Bands

To achieve flat color bands, the continuous lighting calculation is quantized into discrete steps. This is done by mapping the dot product of the light direction and surface normal to a limited set of brightness levels.

#### Basic Toon Shading Shader

```c
// Vertex Shader
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

// Fragment Shader
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

- The vertex shader transforms the vertex positions and normals to world space.
- The fragment shader calculates the dot product of the normalized light direction and surface normal, then quantizes it into discrete steps to create the flat color bands.

### 2. Bold Outlines

To create bold outlines, we can use a technique called edge detection in screen space or backface culling in object space.

**Backface Culling Method**
This method involves rendering the backfaces of the model with a thick line and a solid color.

```c
// Outline Vertex Shader
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

// Outline Fragment Shader
#version 330 core
out vec4 FragColor;

void main() {
    FragColor = vec4(0.0, 0.0, 0.0, 1.0); // Black outline
}
```

**In this shader:**

- The vertex shader moves the vertices along their normals to create the outline effect.
- The fragment shader renders the backfaces with a solid black color to create the outline.

### 3. Combining Flat Color Bands and Outlines

To achieve the final toon shading effect, we render the scene twice: first for the outlines and then for the toon shading.

**Main Rendering Loop**

```cpp
void render() {
    // Render outlines
    glEnable(GL_CULL_FACE);
    glCullFace(GL_FRONT); // Cull front faces
    glUseProgram(outlineShader);
    setUniformsForOutline();
    renderModel();

    // Render toon shading
    glCullFace(GL_BACK); // Cull back faces
    glUseProgram(toonShader);
    setUniformsForToonShading();
    renderModel();
}
```

**In this rendering loop:**

We first render the outlines by culling the front faces and using the outline shader.
Then, we render the toon shading by culling the back faces and using the toon shading shader.

## Tips for Creating Captivating Stylized Game Characters

### 1. Design with Bold Shapes

Toon shading works best with characters and objects that have bold, exaggerated shapes. Simplify the geometry and focus on strong silhouettes.

### 2. Use Saturated Colors

Bright, saturated colors enhance the cartoon-like appearance. Avoid complex textures and focus on flat, vibrant colors.

### 3. Enhance with Additional Effects

Consider adding additional post-processing effects like bloom, vignette, or even a paper texture overlay to enhance the stylized look.

### 4. Pay Attention to Lighting

Even though toon shading simplifies lighting, it’s crucial to place lights thoughtfully to accentuate the character’s features and maintain the cartoon aesthetic.

### 5. Iterate and Experiment

Experiment with different levels of quantization, outline thickness, and color palettes. Iteration is key to finding the perfect balance for your game’s visual style.

## Conclusion

Toon shading is a powerful technique for creating stylized, cartoon-like visuals in games. By quantizing lighting into flat color bands and adding bold outlines, you can achieve a distinctive look that sets your game apart. Understanding and implementing toon shading in shaders, combined with thoughtful character design and lighting, will enable you to create captivating and memorable game characters. Dive into the world of toon shading and unleash your creativity to bring your stylized visions to life.
