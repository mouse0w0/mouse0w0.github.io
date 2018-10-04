---
title: OpenGL在一定区域中渲染2D图形
date: 2018-10-04 10:56:08
tags: OpenGL
---

要实现该功能只需修改片元着色器，其核心代码如下：

```glsl
#version 330 core

// ins and outs ...

// uniforms...
uniform vec4 region;

void main() 
{
    if(gl_FragCoord.x < region.x || gl_FragCoord.x > region.z || gl_FragCoord.y < region.y || gl_FragCoord.y > region.w) {
        discard; // 不在区域内，丢弃片元
    }
    // Do something...
}
```

值得注意的是，OpenGL中窗口坐标的**原点**位于**屏幕左下角**，而常常有人误认为是屏幕左上角。