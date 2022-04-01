# BoxRayTracer

## Description
This application is a CPU-side 3D rendering engine with accompanying WPF front-end. Allows for static visualization of multiple objects and lights in a given scene, with configurable parameters for objects, lighting, and camera. 

## Purpose
This application is being developed with twofold purpose:
1. As a learning opportunity. This application's development has required that I either expand my prior knowledge or learn from scratch in the followign areas:
    * C# development
    * Top-down object-oriented design and solution structure
    * 3D rendering via Distance Estimation (and associated vector math)
    * Lighting principles and implementation
    * ...and more to come!
2. The rendering core is being built to accomodate a subsequent effort which will implement a GPU-side rendering pipeline.

## Technologies/principles used
* C#
* Distance Estimation based raytracing model
* Blinn-Phong lighting model for ambient, diffuse, and specular components
* Shadow and reflection rays

## Development
The remainder of this document will serve as a development journal for recording progress, challenges, and milestones.

### C# Practice

Step 1, before starting development on the ray tracer, was to tackle a few smaller C# projects. I had prior experience with object-oriented programming in Java (among others), but had no direct experience with C#. The aim with these initial exercises was to familiarize myself with C# syntax and Solution structure, with some helpful exposure to WPF/XAML along the way. Some of these exercises were inspired by tutorial materials, and others were self-guided. These projects can be found in my ["Csharp-Practice" repo](https://github.com/crichards17/Csharp-Practice).<br><br>

### WPF Setup and Bitmap generation

The first objective was to set up the WPF app, as well as much of the initial rendering backend and Bitmap pipeline. The milestone here was the ability to return an pixel array from the raytracer (in this case, returning a single color value for all pixels), form a Bitmap from the array, and display it in the WPF.

<img src="https://github.com/crichards17/BoxRayTracer/blob/main/Progression/Captures/02-13_Red_Box.PNG?raw=true" height="300" name="Red_Box" style="display:block;">
<br><br>

### The first sphere

The next goal was to be able to render a simple scene -- a single sphere. Visually the milestone of this work isn't overly impressive, but it required a large bulk of the backend framework to be put in place. These items saw their initial implementations: 
* Camera object and associated functions
* Vector and Color structs
* Sphere DE
* Ray marching
* Pixel array construction

The result is a rendered 3D sphere, though with no lighting effects it appears on screen as a flat circle.

<img src="https://github.com/crichards17/BoxRayTracer/blob/main/Progression/Captures/02-13_First_Sphere.PNG?raw=true" height="300" name="First_Sphere" style="display:block;">
<br><br>

### Front-end controls for Camera and Object parameters

Next was some WPF / XAML work to implement Camera and Shape inputs and outputs. This initial pass is relatively simplistic. My priority at this stage was to learn the XAML and get a working implementation so that I cuold continue forward with the rendering development which was otherwise blocked. I'll revisit the frontend later on once I have a better idea of the specific functionality I want.

<img src="https://github.com/crichards17/BoxRayTracer/blob/main/Progression/Captures/02-15_Rendering_Controls.PNG?raw=true" height="300" name="Rendering_Controls" style="display:block;">
<br><br>

### Lighting model (Blinn-Phong)

The purple circle was great, but now it was time to tackle the lighting to give me some real 3D object renders. This required a new class of objects in the SceneLights, and then illumination calculations for the ray tracer to return pixel color when intersecting an object fragment. This ray tracer uses the Blinn-Phong lighting model, which calculates illumination from additive ambient, diffuse, and specular components. These are multiplied by the base color of the object (later on replaced by material properties for the respective illumination components) and returned to the pixel array. 

This first image was taken after the "Blinn" model had been implemented. You can see that the specular highlight has a rather sharp dropoff, producing a noticeable "edge."

<img src="https://github.com/crichards17/BoxRayTracer/blob/main/Progression/Captures/02-18_First_Lighting.PNG?raw=true" height="300" name="First_Lighting" style="display:block;">

This second image demonstrates the difference that the "Phong" specular calculation makes to the illumination effect. The Phong calculation references the half-angle to the viewing plane and produces a smoother, more natural-looking specular highlight.

<img src="https://github.com/crichards17/BoxRayTracer/blob/main/Progression/Captures/02-18_Blinn_Specular.PNG?raw=true" height="300" name="Blinn_Specular" style="display:block;">
<br><br>

### Multiple Objects in the scene

So far, the scene had been exclusively composed of a single image. This next bit of work involved reconfiguring the object structure for the Scene and its constituent components, including how those components were processed by the ray tracer. Because of the nature of distance estimation, each ray march needs to check all Scene objects for proximity, and respond accordingly. The result here was the ability to render multiple objects in the scene 

<img src="https://github.com/crichards17/BoxRayTracer/blob/main/Progression/Captures/02-19_Multiple_Objects.PNG?raw=true" height="300" name="Multiple_Objects" style="display:block;">
<br><br>

### Shadow Rays

The next component of the lighting model to be implemented were shadow rays. Shadows in this rendering model are calculated by ray marching from the incident object fragment toward each light source, and checking for intersection with any of the scene objects along the way. If an object is intersected, then the diffuse and specular components from that light are not added to the color that's returned to the pixel array. The result is a shadow cast from each object onto the others, as demonstrated in the below image which has the purple light axis-aligned with the two objects and casting a shadow onto the square on the right.

<img src="https://github.com/crichards17/BoxRayTracer/blob/main/Progression/Captures/02-26_First_Shadows.PNG?raw=true" height="300" name="Multiple_Objects" style="display:block;">
<br><br>

### Box DE and Mixed Objects

This renderer uses distance estimation for its object definitions. The reason this rendering style was chosen is mostly due to the eventual goal of the project (fractal rendering). Rather than objects being defined by sets of vertices that are then intersected by the cast rays, the distance estimator defines each shape by an equation that returns the minimum distance from a given point to the surface of the object.

For spheres, this is simple -- the minimum distance from any point outside of a sphere to its surface is the distance from the point to the sphere's center minus its radius. Spheres are fully symmetrical, so translation and rotation has no impact on this math. You can see now why all of the above screenshots show spheres.

The box shape is more challenging. The distance from a given point to the surface of a non-spherical shape is not so straightforward. The approach used here is to translate the given point to a fixed quadrant, do some math to determine the nearest surface, and then translate back. A similar process is needed for calculating the Normal vector for a given fragment, which is required for the diffuse and specular lighting components. In the screenshot below we can see a symmetrical cube with an accompanying sphere, showing both DEs working as expected.

<img src="https://github.com/crichards17/BoxRayTracer/blob/main/Progression/Captures/03-02_Mixed_Objects.PNG?raw=true" height="300" name="Multiple_Objects" style="display:block;">
<br><br>

### Box DE Transforms

The initial box implementation above show a symmetrical cube, axis-aligned at the global origin. This was a good example of narrowing scope to allow me to tackle a problem one portion at a time. The next step, then, was to expand the Box functionality to allow for transformrations and translations. This work will also be generalized for use in other shapes, as it mostly involves re-framing the DE calculation around the object's local coordinates by transposign the input vector from the global space -- something that's aplicable to any complex shape. 

<img src="https://github.com/crichards17/BoxRayTracer/blob/main/Progression/Captures/03-10_Scale_Transform_Box.PNG?raw=true" height="300" name="Box Transforms" style="display:block;">
<br><br>

### Reflections

The next step in the lighting model was to implement reflections. Reflections work by casting a ray from the incident object fragment and checking for collision with another scene object, similar to the shadow rays. Unlike the shadow rays, however, the reflection ray adds the illumination value at that fragment to the ray's origin fragment -- multiplied by a "reflectivity" scalar -- and then recursively calls the same process again. An important inclusion here is a maximum-reflections parameter which persists through the recursive reflections calls, preventing what could otherwise be a prohibitively large number of reflection calls depending on the scene geometry.

This first screenshot shows the first pass at reflections in the box-and-sphere scene, with the reflection count limited to 2.

<img src="https://github.com/crichards17/BoxRayTracer/blob/main/Progression/Captures/03-13_Single_Reflections.PNG?raw=true" height="300" name="Single Reflections" style="display:block;">
<br><br>

### Fractals

Now that the lighting model supports the major components I had wanted (BP ambient/diffuse/specular, shadows, reflections), it was a good time to dive in and try implementing the initial impetus for the whole project -- 3D fractals! These are modeled by DE equations that predominantly come from prior art. However, they required a fair amount of troubleshooting to make adjustments to the configuration and implementation details. 

The first shape I attempted was the "Mandlebox." My initial attempt at translating the DE for this one into my model -- pictured below -- did not work as expected. After much troubleshooting I decided to set it aside for the time being and try a different DE to see if I had a pervasive issue in my model that would affect all estimated shapes.

My next attempt, then, was to implement the "Mandlebulb." Much to my surprise (and troubleshooting chagrin), this one worked beautifully after some tweaking of the various variables that make up the shape definition.

<img src="https://github.com/crichards17/BoxRayTracer/blob/main/Progression/Captures/03-20_Mandlebulb_Initial.PNG?raw=true" height="300" name="Initial Mandlebulb" style="display:block;">
<br><br>

### Ambient Occlusion

The Mandlebulb shape itself was displaying well, but certainly missing some visual depth.Next on my list was to add ambient occlusion, which has the effect of adding contrast to nooks and crannies, as well as darkening portions of a shape that are further away from the camera. The method I've opted for is to reference the number of marches the ray march takes to reach a given object fragment, with the resulting pixel color value returning as darker for those that required more marches. Because of the way that the distance estimation march works, this has the effect of darkening fragments that are further away, as well as fragments that are near other faces or features. This resulting effect can be seen in the below image.   

<img src="https://github.com/crichards17/BoxRayTracer/blob/main/Progression/Captures/03-25_Mandlebulb_Occluded.PNG?raw=true" height="300" name="Occluded Mandlebulb" style="display:block;">
<br><br>

