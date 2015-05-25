---
layout: post  
title: OpenGL 2： 着色语言  
category: 游戏开发 
tags: OpenGL  
keywords: OpenGL Shader  
description:   
---


##着色语言

着色语言是一种类C语言，但不像C语言一样支持双精度浮点型(double)、字节型(byte)、短整型(short)、长整型(long)，并且取消了C中的联合体(union)、枚举类型(enum)、无符号数(unsigned)以及位运算等特性。

着色语言中有许多内建的原生数据类型以及构建数据类型，如：浮点型(float)、布尔型(bool)、整型(int)、矩阵型(matrix)以及向量型(vec2、vec3等)等。总体来说，这些数据类型可以分为标量、向量、矩阵、采样器、结构体以及数组等。

 
###shader支持下面数据类型：

```
Float //浮点数
 
bool  //布尔数

int		//整数


vec2	//包含了2个浮点数的向量    

vec3	//包含了3个浮点数的向量    

vec4	//包含了4个浮点数的向量    


ivec2	//包含了2个整数的向量         

ivec3	//包含了3个整数的向量                  

ivec4  //包含了4个整数的向量

 

bvec2  //包含了2个布尔数的向量

bvec3  //包含了3个布尔数的向量

bvec4  //包含了4个布尔数的向量

 

mat2   //2*2维矩阵

mat3   //3*3维矩阵

mat4   //4*4维矩阵

 

sampler1D	//1D纹理着色器

sampler2D	//2D纹理着色器

sampler3D	//3D纹理着色器
```
###  顶点着色器

> 顶点着色器代码  :

```C
uniform mat4 uMVPMatrix;  //应用程序传入顶点着色器的总变换矩阵

attribute vec4 aPosition;	//应用程序传入顶点着色器的顶点位置

attribute vec2 aTextureCoord;	//应用程序传入顶点着色器的顶点纹理坐标

attribute vec4 aColor	//应用程序传入顶点着色器的顶点颜色变量

varying vec4 vColor	//用于传递给片元着色器的顶点颜色数据

varying vec2 vTextureCoord;	//用于传递给片元着色器的顶点纹理数据

void main()

{

    gl_Position = uMVPMatrix * aPosition;//根据总变换矩阵计算此次绘制此顶点位置                          

    vColor = aColor;	//将顶点颜色数据传入片元着色器

    vTextureCoord = aTextureCoord;	//将接收的纹理坐标传递给片元着色器

}        
```

---








