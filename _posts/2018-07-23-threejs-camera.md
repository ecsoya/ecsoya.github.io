---
layout: post
date:   2018-07-23
categories: [js]
title: "Three.js移动相机"
---

 移动相机照到物体上是Three中一个常用的功能。

 默认情况下，camera照在（0，0，0）点，移动camera之后，必须调用lookAt更新照射位置。

 涉及到的内容有两个：
- 改变camera的position，移动相机。
- 调用lookAt方法，照到物体表面。

具体的方法如下：

```
	var mouse = {};

    function onDocumentMouseDown(event) {
        // event.preventDefault();
        mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
        mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

        raycaster.setFromCamera(mouse, camera);
        var intersects = raycaster.intersectObjects(group.children, false);
        if (intersects.length > 0) {
            //Move camera to the position of intersect object.
            var position = intersects[0].object.position.clone();
            // Move it without animation.
            camera.position = position;
            camera.lookAt(position);
        }

    }
```

由于这个方法是直接改变camera的位置，看上去不太友好，我们可以用[Tween.js](https://github.com/sole/tween.js)来添加点动画效果。

```
    function tweenCamera(position, target) {
        new TWEEN.Tween(camera.position).to({
            x: position.x,
            y: position.y,
            z: position.z
        }, 600)
            .easing(TWEEN.Easing.Sinusoidal.InOut).start();
        new TWEEN.Tween(controls.target).to({
            x: target.x,
            y: target.y,
            z: target.z
        }, 600)
            .easing(TWEEN.Easing.Sinusoidal.InOut).start();
    }
```
### 完整示例

- [https://ecsoya.github.io/SVG-Three-Map/demo/camera.html](https://ecsoya.github.io/SVG-Three-Map/demo/camera.html)

### 参考资料
- [https://github.com/mrdoob/three.js/issues/1689](https://github.com/mrdoob/three.js/issues/1689) 
- [https://stackoverflow.com/questions/37154008/click-to-place-camera-near-an-object-in-three-js](https://stackoverflow.com/questions/37154008/click-to-place-camera-near-an-object-in-three-js)