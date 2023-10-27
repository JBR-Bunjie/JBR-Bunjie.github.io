---
title: CatlikeCoding - Chapter01. Basics
date: 2022-12-23 12:23:23
tags:
  - CatlikeCoding
  - Rendering
categories:
  - CatlikeCoding
  - Rendering
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# Chapter01：Basics

> 对于基础部分，除了这些内容外，仅记录了有用的 QA 内容：

## Game Objects and Scripts

> **1. Which Unity versions are appropriate?**
>
> ...A further f1 suffix indicates an official final release...
> 说明：因为 Unity 同时维护了多个版本，只有发行版会有 f1 后缀，测试版分别为 b-beta、a-alpha，如：Beta 通道的 2023.1.0b20 和 Alpha 通道的 2023.2.0a19
>
> **2. What is albedo?**
>
> Albedo is a Latin word which means whiteness. It's the color of something when illuminated by white light.
>
> **3. What is the default access modifier for classes?**
>
> Without the access modifier, it would be as if we had written **internal class Clock**. That would restrict access to code from the same assembly, which becomes relevant when you use code packaged in separate assemblies. To make sure it always works, make classes public by default.
>
> **4. What does mono-behavior mean?**
>
> **The idea is that we can program our own components to add custom behavior to game objects. That's what the behavior part refers to.** It just happens to use the British spelling, which is an oddity. **The mono part refers to the way in which support for custom code was added to Unity.** It used the Mono project, which is a multi-platform implementation of the .NET framework. Hence, `MonoBehaviour`. It's an old name that we're stuck with due to backwards-compatibility.
>
> **5. Doesn't `Awake` have to be `public`?**
>
> `Awake` and a collection of other methods are considered special Unity event methods. The Unity engine will find them and invoke them when appropriate, no matter how we declare them. This happens from outside the managed .NET environment.
>
> **6. What's the difference between `localRotation` and `rotation`?**
>
> The `localRotation` property represents the rotation described by the `Transform` component in isolation, thus it is a rotation relative to its parent. It's the rotation that you see in its inspector. In contrast, the `rotation` property represents the final rotation in world space, taking the entire object hierarchy into account. Setting that property would produce weird results if we rotate the clock as a whole, because the arm would ignore that as the property compensates for the rotation of the clock.
>
> **7. Shouldn't there be a warning that `hoursPivot` is never initialized?**
>
> The compiler can detect that no code assigns anything to the field and could indeed issue such a warning, because it is unaware that we set it up via Unity's inspector. However, this warning is suppressed by default. The suppression can be controlled via the project settings. There's a _Suppress Common Warnings_ toggle under _Player / Other Settings / Script Compilation_. It suppresses warnings about both uninitialized and unused private fields.
>
> **8. What's special about `const` values?**
>
> The **`const`** keyword indicates that a value will never change and doesn't need to be a field. Instead, its value will be computed during compilation and is substituted for all usage of the constant. This is only possible for primitive types like numbers.
>
> **9. What's a variable?**
>
> A variable acts like a field, except that it exists only while a method is being executed. It belongs to the method, not the class.
> Fields、Properties、Variables

## Building A Graph

> **1. A good understanding of mathematics is essential when programming.**
>
> **2. It is a pre-fabricated game object that exists in the project, not in a scene.**
>
> **3. Why is the background of the prefab scene uniform dark blue?**
>
> If you open a prefab instance that's part of a scene then the scene window will display its surroundings depending on the _Context_ settings shown at the top of the window. If you open the prefab asset then there is no context. In the case of assets the skybox is disabled by default in the prefab scene, along with some other things. You can configure this via the scene window's toolbar, just like you can for the regular scene window. The skybox can be toggled via the dropdown menu that looks like a stack with a star on top of it. Notice how the scene toolbar settings change when you jump in and out of prefab asset mode.
>
> **4. What is the full inheritance chain of `MonoBehaviour`?**
>
> `MonoBehaviour` extends `Behaviour`, which extends `Component`, which extends `Object`.

## Mathematical Surface

> 关于 Delegate，可用的拓展内容：[(82) What are Delegates? (C# Basics, Lambda, Action, Func) - YouTube](https://www.youtube.com/watch?v=3ZfwqWl-YI0)

## Measuring Performance

> **1. Whether a target frame rate can be achieved depends on how long it takes to process individual frames. To reach 60FPS we must update and render each frame in less than 16.67 milliseconds. The time budget for 30FPS is double that, thus 33.33ms per frame.**
>
> > The statistics show a frame during which the CPU main thread took 31.7ms and the render thread took 29.2ms. You'll likely get different results, depending on your hardware and the game window screen size. In my case it suggests that the entire frame took 60.9ms to render, but the statistics panel reported 31.5FPS, matching the CPU time. **The FPS indicator seems to takes the worst time and assumes that matches the frame rate. This is an oversimplification that only takes the CPU side into account, ignoring the GPU and display. The real frame rate is likely lower.**
>
> **2. **
>
> Part1:Text strings are objects. When we create a new one via SetText this produces a new string object, which is responsible for the allocation of 106 bytes. Unity's UI refresh then increases this to 4.5 kilobytes. While this isn't much it will accumulate, <u>**triggering a memory garbage collection process at some point which will result in an undesired frame duration spike.**</u>
>
> Part2: _It is important to be aware of memory allocation for temporary objects and eliminate recurring ones as much as possible._ Fortunately SetText and Unity's UI update only perform these memory allocations in the editor, for various reasons, like updating the text input field. If we profile a build then we will find some initial allocations but then no more. So **it is essential to profile builds. Profiling editor play mode is only good for a first impression.**

### Game Window Statistics

> **GPU Instancing, SRP Batching and Dynamic Batching**
>
> Another way to improve rendering performance is by enabling GPU instancing. This makes it possible to use a single draw command to tell the GPU to draw many instances of one mesh with the same material, providing an array of transformation matrices and optionally other instance data. In this case we have to enable it per material. Ours have an _Enable GPU Instancing_ toggle for it.

### Difference between Lerp and SmoothStep

**_Lerp_ is shorthand for linear interpolation.** It will produce a straight constant-speed transition between the functions. We can make it look a bit smoother by slowing down the progress near the start and end. This is done by replacing the raw progress with an invocation of `Mathf.Smoothstep` with zero, one, and the progress as arguments. **It applies the 3x^2^−2x^3^ function, commonly known as smoothstep.** The first two parameter of `Smoothstep` are an offset and scale for this function, which we don't need so use 0 and 1. [Unity - Scripting API: Mathf.SmoothStep (unity3d.com)](https://docs.unity3d.com/ScriptReference/Mathf.SmoothStep.html)

![img](https://catlikecoding.com/unity/tutorials/basics/measuring-performance/automatic-function-switching/smoothstep.png)
0–1 Smoothstep and linear.

**More: **The `Lerp` method clamps its third argument so it falls in the 0–1 range. The `Smoothstep` method does this as well. We configured the latter to output a 0–1 value, so the extra clamp of `Lerp` is not needed. For cases like this there is an alternative `LerpUnclamped` method, so let's use that one instead.

## Compute Shaders

> **1. What's a MiB?**
>
> Because computer hardware uses binary numbers to address memory it's partitioned in powers of two, not powers of ten. MiB is the suffix for mebibyte, which is 2^20^ = 1,024^2^ = 1,048,576 bytes. This was originally known as a megabyte—indicated with MB—but that's now supposed to indicate 10^6^ bytes, matching the official definition of a million. However, MB, GB, etc. are often still used instead of MiB, GiB, etc.
> 关于 MiB 与 MB：[Mebibyte - 维基百科，自由的百科全书 (wikipedia.org)](https://zh.wikipedia.org/wiki/Mebibyte)

## References

- https://catlikecoding.com/unity/tutorials/basics/
-
