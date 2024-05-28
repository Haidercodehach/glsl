---
layout: ../layouts/BlogPost.astro
title: Physics-Based Shading Implementing Realistic Lighting and Materials Using Physics-Based Shading Models
slug: "physics-based-shading-implementing-realistic-lighting-and-materials-using-physics-based-shading-models"
description: >-
  Physics-based shading (PBS) is a rendering technique that simulates the way light interacts with surfaces in the real world. By using physically accurate lighting models and material definitions, PBS allows for the creation of highly realistic visuals in computer graphics.
tags:
  - shader
  - gamedev
added: "May 28 2024"
---

Physics-based shading (PBS) is a rendering technique that simulates the way light interacts with surfaces in the real world. By using physically accurate lighting models and material definitions, PBS allows for the creation of highly realistic visuals in computer graphics. This blog explores the fundamentals of physics-based shading, its importance in achieving realism, and provides a guide on implementing PBS models in your rendering pipeline.

## Understanding Physics-Based Shading

Physics-based shading is grounded in the principles of physical light behavior, ensuring that materials react to light in a way that mimics the real world. This approach contrasts with traditional shading models, which often rely on approximations that can produce less realistic results.

### Key Components of PBS

- Energy Conservation: Ensures that the amount of reflected light never exceeds the amount of incoming light.
- Microfacet Theory: Models surfaces as a collection of tiny facets, each interacting with light.
- BRDF (Bidirectional Reflectance Distribution Function): Describes how light is reflected at an opaque surface.
- Fresnel Effect: Describes how light reflection varies with the angle of incidence.

## Importance of PBS

- Realism: Creates lifelike images by accurately simulating light interaction with materials.
- Consistency: Produces consistent visual results under varying lighting conditions.
- Efficiency: Modern graphics hardware is optimized for PBS, allowing for efficient rendering.

## Implementing Physics-Based Shading Models

To implement physics-based shading, you need to integrate several components into your rendering pipeline. The following sections outline the key elements and steps involved in this process.

## 1. Material Models

Materials in PBS are defined by their intrinsic properties, which determine how they interact with light. The most common parameters include:

- Albedo: The base color of the material.
- Metalness: Indicates whether the material is metallic or non-metallic.
- Roughness: Describes the microfacet roughness, affecting the sharpness of reflections.
- Ambient Occlusion: Simulates the occlusion of ambient light in crevices and corners.

## 2. Lighting Models

PBS uses advanced lighting models to simulate realistic light behavior. The most widely used model is the Cook-Torrance model, which incorporates several key components:

- Diffuse Reflection: Simulated using Lambertian reflection, representing light scattered uniformly in all directions.
- Specular Reflection: Modeled using the microfacet theory, where each facet reflects light according to the Fresnel effect and normal distribution function (NDF).

## 3. BRDF Implementation

The Bidirectional Reflectance Distribution Function (BRDF) is central to PBS, defining how light is reflected at an opaque surface. The Cook-Torrance BRDF is commonly used, combining the following components:

- Normal Distribution Function (NDF): Describes the distribution of microfacet normals. The GGX (Trowbridge-Reitz) distribution is popular for its realistic appearance.
- Fresnel Term: Calculates the amount of light reflected based on the viewing angle. The Schlick approximation is commonly used for its efficiency.
- Geometry Function: Accounts for the shadowing and masking effects of microfacets. The Smith function is a typical choice.

```c
float D_GGX(float NdotH, float roughness) {
    float a = roughness * roughness;
    float a2 = a * a;
    float NdotH2 = NdotH * NdotH;
    float denom = (NdotH2 * (a2 - 1.0) + 1.0);
    return a2 / (PI * denom * denom);
}

float G_SchlickGGX(float NdotV, float roughness) {
    float r = roughness + 1.0;
    float k = (r * r) / 8.0;
    return NdotV / (NdotV * (1.0 - k) + k);
}

vec3 F_Schlick(vec3 F0, float HdotV) {
    return F0 + (1.0 - F0) * pow(1.0 - HdotV, 5.0);
}

vec3 cookTorranceBRDF(vec3 F0, float roughness, float NdotV, float NdotL, float NdotH, float HdotV) {
    float D = D_GGX(NdotH, roughness);
    float G = G_SchlickGGX(NdotV, roughness) * G_SchlickGGX(NdotL, roughness);
    vec3 F = F_Schlick(F0, HdotV);
    return (D * G * F) / (4.0 * NdotV * NdotL);
}
```

## Implementing Physics-Based Shading

To implement PBS in shaders, you need to write GLSL or HLSL code that calculates lighting and material interactions based on the aforementioned models.

## Step 1: Define Material Properties

Define the properties required for PBS, including albedo, metallic, roughness, and ambient occlusion. These properties will drive the shading calculations.

```c
struct Material {
    vec3 albedo;
    float metallic;
    float roughness;
    float ao;
};
```

## Step 2: Calculate Lighting

Calculate the lighting for each light source in the scene. This involves computing the diffuse and specular components and combining them.

```c
vec3 calculateLighting(Material material, vec3 normal, vec3 viewDir, vec3 lightDir, vec3 lightColor) {
    vec3 halfDir = normalize(lightDir + viewDir);
    float NdotL = max(dot(normal, lightDir), 0.0);
    float NdotV = max(dot(normal, viewDir), 0.0);
    float NdotH = max(dot(normal, halfDir), 0.0);
    float HdotV = max(dot(halfDir, viewDir), 0.0);

    vec3 F0 = mix(vec3(0.04), material.albedo, material.metallic);
    vec3 F = cookTorranceBRDF(F0, material.roughness, NdotV, NdotL, NdotH, HdotV);
    vec3 kS = F;
    vec3 kD = vec3(1.0) - kS;
    kD *= 1.0 - material.metallic;

    vec3 diffuse = lambertian(material.albedo, lightColor, NdotL);
    vec3 specular = F * D_GGX(NdotH, material.roughness) * G_SchlickGGX(NdotV, material.roughness) * G_SchlickGGX(NdotL, material.roughness);

    return (kD * diffuse + specular) * NdotL;
}
```

## Step 3: Combine Ambient and Direct Lighting

Combine ambient occlusion with direct lighting to get the final color of the surface.

```c
vec3 shade(Material material, vec3 normal, vec3 viewDir, vec3 lightDir, vec3 lightColor, vec3 ambientLight) {
    vec3 ambient = material.albedo * ambientLight * material.ao;
    vec3 lighting = calculateLighting(material, normal, viewDir, lightDir, lightColor);
    return ambient + lighting;
}
```

## Step 4: Integrate into the Rendering Loop

Integrate the shading calculations into the rendering loop, applying them to each pixel.

```c
void main() {
    Material material = Material(vec3(1.0, 0.5, 0.31), 0.5, 0.1, 1.0);
    vec3 normal = normalize(vNormal);
    vec3 viewDir = normalize(viewPos - vFragPos);
    vec3 lightDir = normalize(lightPos - vFragPos);
    vec3 lightColor = vec3(1.0);
    vec3 ambientLight = vec3(0.2);

    vec3 color = shade(material, normal, viewDir, lightDir, lightColor, ambientLight);
    FragColor = vec4(color, 1.0);
}
```

## Conclusion

Physics-based shading provides a powerful framework for creating realistic lighting and materials in computer graphics. By adhering to physical principles, PBS ensures that materials behave consistently under various lighting conditions, resulting in more believable and immersive visuals. Understanding and implementing PBS involves grasping key concepts like energy conservation, microfacet theory, and the Fresnel effect, and integrating these into your shading pipeline. Whether you're developing for games, films, or virtual reality, physics-based shading is an essential tool for achieving high levels of realism and visual fidelity.

## Frequently Asked Questions (FAQs)

- What is Physics-Based Shading (PBS)?
  - Physics-based shading (PBS), also known as physically-based rendering (PBR), is a shading model that aims to simulate the interaction of light with surfaces using principles derived from real-world physics. PBS ensures consistent and realistic rendering by adhering to physical laws, making it a standard in modern graphics engines.
- What are the key concepts in Physics-Based Shading?
  - The key concepts in PBS include:
  - Energy Conservation: The total amount of light reflected by a surface should not exceed the amount of incoming light, ensuring realistic brightness levels.
  - Microfacet Theory: Surfaces are made up of tiny microfacets, each reflecting light differently based on their orientation and distribution.
  - Fresnel Effect: Reflectance varies with the angle of incidence, increasing at grazing angles, which is crucial for materials like metals and water.
  - BRDF (Bidirectional Reflectance Distribution Function): A function that defines how light is reflected at an opaque surface, considering the incident and outgoing angles.
- What are the main components of Physics-Based Shading?
  - The main components of PBS are:
  - Diffuse Reflection: Models how light scatters in many directions when it hits a rough surface. This component is responsible for the base color of the material.
  - Specular Reflection: Models the shiny highlights on a surface, crucial for materials like metals and plastics. It is often modeled using the Cook-Torrance BRDF.
  - Ambient Occlusion: Simulates the soft shadows that occur in creases, holes, and surfaces that are close to each other, adding depth and realism to the scene.
- How do you calculate Diffuse Reflection in PBS? - Diffuse reflection is commonly modeled using the Lambertian reflectance model, which assumes uniform scattering. The formula for Lambertian reflectance is:
  ```c
  vec3 lambertian(vec3 albedo, vec3 lightColor, float NdotL) {
  return albedo _ lightColor _ NdotL / PI;
  }

```
- Why is Physics-Based Shading important?
    - PBS is important because it ensures that materials and lighting behave realistically under various conditions, leading to more immersive and believable visuals. This consistency is crucial for applications in games, films, virtual reality, and any other medium that requires high levels of visual fidelity.
```
