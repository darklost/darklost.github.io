---
layout: post  
title: OpenGL 2：OpenGL ES Shader 示例（魔毯）	
category: 游戏开发 	
tags: OpenGL  
keywords: OpenGL  shader
description:   
---

折腾了一段时间mac上的OpenGL Shader Builder,写出了第一个`shader program`：魔毯(copy:))

### Program
**Demo01.fs**

```glsl
uniform sampler2D tex;

void main()
{
	gl_FragColor = texture2D(tex, gl_TexCoord[0].xy);
}

```

**Demo01.vs**

```glsl
uniform float time;
void main()
{
	vec4 vertex= gl_Vertex;
     vertex.z=sin(vertex.x*1.5+time*5.0) * 0.25;
	gl_TexCoord[0] = gl_MultiTexCoord0;
	gl_Position = gl_ModelViewProjectionMatrix*vertex;

}

```

### Sybomls
如下图：

![1](/public/img/opengl/opengl-demo01-sybomls.gif)


### 最终效果图：


![2](/public/img/opengl/opengl-demo01.gif)

---








