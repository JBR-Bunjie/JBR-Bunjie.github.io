---
title: learningOpenGl Chapter 1.8
date: 2023-3-8 10:27:08
tags:
  - Opengl
  - Shader
categories:
  - TA
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# Coordinate Systems


## categories

请主要注意这中间的MVP变换与最后的视口变换

![coordinate_systems](http://learnopengl.com/img/getting-started/coordinate_systems.png)

```glsl
// final tranformation looks like:
layout (location = 0) in vec3 Position;
...
gl_Position = Project * View * Model * Move * Rotate * Translate * vec4(Position, 1.0f);
```

