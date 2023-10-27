---
title: Volumetric Cloud
date: 2023-3-31 12:23:23
tags:
  - Unity
  - Shader
categories:
  - Unity
  - Shader
<!--feature: true-->
cover: https://raw.githubusercontent.com/JBR-Bunjie/JBR-Bunjie/main/back.jpg
---

# Volumetric Cloud

## Based On RayMarching

### 求包围盒

"从摄像机发出射线，每次前进一点判断是否在体积雾包围盒范围内然后再采样计算颜色。我们通过计算摄像机与体积雾包围盒相交的点，直接从该交点沿射线方向前进，来减少无效的步进。"

# Referece

- [(80) FMX2017 Technical Directing Special: Real-time Volumetric Cloud Rendering - YouTube](https://www.youtube.com/watch?v=8OrvIQUFptA)
