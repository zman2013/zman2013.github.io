---
title: Unity 2D 基础知识
tags:
  - 2d
id: '327'
categories:
  - - unity
date: 2019-11-01 21:39:02
---

# 1\. 层级关系

2D世界的物体都在Z轴同一位面（z=0），摄像机在z=-10进行拍摄。物体之间的层级关系通过Y轴的高地来进行计算遮挡关系，从而产生立体感。 设置步骤： 1. 设置根据Y轴进行排序 Edit -> Project Setting -> Graphics -> Camera Settings -> Transparent Sort Mode => (x=0,y=1,z=0) 2. 设置排序为Sprite中心点 Inspector -> Sprite Render -> Sprite Sort Point -> Pivot 3. 调整Sprite的中心点 Assets -> 选中目标Sprite -> Edit Sprite -> 调整Pivot

# 2\. 刚体、碰撞

UnityEngine模拟了物理世界基本定律：可以通过给GameObject添加刚体（Rigidbody 2D）组件，GameObject便开始收到物理定律的影响（记得把Gravity置为0）。通过添加Collider2D实现碰撞效果。 1. 防止GameObject沿Z轴旋转 Inspector -> Rigidbody 2D -> Constraints -> Freeze Rotation -> 勾选z 2. Collider调整到物体下半部分效果比较真实，即正好是物体的占地面积。 3. OnTriggerEnter2D 任意一方勾选了Collider的trigger属性，使用此方法。物体之间可以发生碰撞，同时可以互相进入对方空间内部。 OnCollisionEnter2D不勾选trigger时使用，物体之间发生碰撞，无法进入对方空间。

# 3\. 动画

Animation -> 动画（内容） Animator -> GameObject的Component Animation Controller -> 动画状态控制器，可以从一个状态转变为另一个状态 Blend State 挺有趣的（组合多个状态进行叠加）

# 4\. 声音

Audio Source -> 声源 Audio Listener -> 喇叭 Audio Clip -> 声音内容 Clip 放到 Source 中进行播放，Listener负责接收

# 5\. RectTransform

UI对象的中心点(Pivot) -> Unity开始渲染图像时从中心点开始渲染 UI对象的锚点(Anchor) -> 与父对象的相对位置是通过计算与Anchor的相对位置计算的 UI的对象在Screen坐标中，不在World坐标，需要进行转换。

# 6\. Tile

Tile的依存关系： Grid - TileMap1 - Tile1 - Tile2 - TileMap2 - Tile3 这样组织场景会比较方便管理。 Pallete挺好用的，尤其RulePallete、RandomPallete、LinePallete。