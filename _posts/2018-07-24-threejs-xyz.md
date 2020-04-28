---
layout: post
date:   2018-07-24
categories: [js]
title: "Three.js坐标转换"
---

 有些情况下，我们可能需要获取Three中物体在屏幕上显示的大小。

 为了得到这个值，我们需要先理解以下内容：
- 首先，Three中的物体是三维的，可以在空间范围内任意旋转。
- 其次，随着物体的旋转，屏幕上显示的区域只是通过Camera投影过去的那部分。

```
        var positionOfCamera = camera.position;
        var positionOfMesh = mesh.position;

        // Mesh的大小，BBox
        var sizeOfMesh = getSize(mesh);
        // 计算相机到mesh表面的距离，注意：Z轴垂直与mesh表面。
        var dist = positionOfCamera.z - positionOfMesh.z - sizeOfMesh.z / 2;

        // convert vertical fov to radians
        var vFOV = THREE.Math.degToRad(camera.fov);

        // 计算高度
        var height = 2 * Math.tan(vFOV / 2) * dist;
        // 在Three中实际高度与投影高度的一个比值
        var fractionHeight = sizeOfMesh.y / height;
        // 屏幕上的实际高度
        var heightPixels = window.innerHeight * fractionHeight;

        // 计算宽度
        var width = height * camera.aspect;
        var fractionWidth = sizeOfMesh.x / width;
        //屏幕上的宽度
        var widthPixels = window.innerWidth * fractionWidth;

        // Mesh的中心点，BBox
        var center = getCenter(mesh);
        //将3D坐标转换到屏幕坐标
        var centerOnScreen = toScreenPosition(center);
```

### 完整示例

- [https://ecsoya.github.io/SVG-Three-Map/demo/point-convert.html](https://ecsoya.github.io/SVG-Three-Map/demo/point-convert.html)

### 参考资料
- [https://stackoverflow.com/questions/13350875/three-js-width-of-view](https://stackoverflow.com/questions/13350875/three-js-width-of-view) 