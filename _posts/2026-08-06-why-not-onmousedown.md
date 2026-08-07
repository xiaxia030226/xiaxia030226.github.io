---
title: 2.为什么不用OnMouseDown
date: 2026-08-06 00:00:00 +0800
categories: [工作问题记录]
tags: [Unity, 鼠标检测, 射线检测, OnMouseDown, 性能优化]
---

上篇文章介绍了 `OnMouseDown` 的用法，末尾提了一句"性能开销可以忽略不计"——这个结论有问题，这里纠正一下。

## OnMouseDown 到底怎么工作的

Unity 官方文档说得比较模糊，实际上 `OnMouseDown` 的底层是这样的：**每帧对所有挂载了 Collider 的 GameObject 逐个发射射线**，检测鼠标是否在其范围内，如果命中且鼠标按下了，就调用对应物体上的 `OnMouseDown` 方法。

关键就在"逐个"这两个字——它是遍历，不是按需查询。

## 性能差距有多大

对比一下两种做法：

**射线检测（手动控制）：**

```csharp
void Update()
{
    if (Input.GetMouseButtonDown(0))
    {
        Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
        if (Physics.Raycast(ray, out RaycastHit hit, float.MaxValue))
        {
            // 只处理被击中的那一个物体
        }
    }
}
```

`Physics.Raycast` 利用 Unity 内部的 BVH 空间加速结构，只对射线路径上的物体做碰撞检测。鼠标没按下的帧，完全不走射线。

**OnMouseDown（引擎自动遍历）：**

```csharp
void OnMouseDown()
{
    // Unity 每帧帮你跑了 N 次射线，N = 场景中有 Collider 的物体数量
}
```

Unity 在底层每帧都要对全部有 Collider 的物体做检测，不管鼠标有没有按下。场景中 100 个物体就是 100 次检测，1000 个就是 1000 次。割草类游戏后期同屏几百个敌人是常态，这个开销就不能忽略了。

## 结论

之前说"中小型项目影响可以忽略不计"不够准确。如果你是做割草游戏、RTS 或者任何同屏物体数量不固定的项目，**不要用 OnMouseDown**。手动在 Update 里做射线检测，鼠标不按时不走射线，鼠标按下时也只做一次带空间加速的检测，性能差距不是一点半点。

我之前一直用的那种手动射线方式才是正确的做法。
