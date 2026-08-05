---
title: OnMouseDown
date: 2026-08-05 00:00:00 +0800
categories: [工作问题记录]
tags: [Unity, 鼠标检测, 射线检测, OnMouseDown]
---

今天让 AI 帮我写 Unity 代码时，它直接用了一个我从来没用过的 API：`OnMouseDown`。

## 射线检测

之前我检测鼠标点击物体都是走这套标准流程——发射射线：

```csharp
void Update()
{
    if (Input.GetMouseButtonDown(0))
    {
        Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
        if (Physics.Raycast(ray, out RaycastHit hit, float.MaxValue))
        {
            // 处理点击 hit.collider.gameObject
        }
    }
}
```

## OnMouseDown

AI 写的代码直接在脚本里用了 `OnMouseDown`：

```csharp
void OnMouseDown()
{
    // 物体被点击时自动调用
    Debug.Log(gameObject.name + " 被点击了");
}
```

`OnMouseDown` 是 `MonoBehaviour` 内置的消息方法。当鼠标在 Collider 上按下时，Unity 自动调用该方法。底层其实也是射线检测，只是 Unity 封装好了。

### 触发条件

- 物体必须有 `Collider` 组件
- 对 2D 物体同理，对应 `OnMouseDown` + `Collider2D`
- 只在鼠标位于 Collider 范围内时触发

### 其他相关的鼠标消息

| 方法 | 触发时机 |
|------|---------|
| `OnMouseDown()` | 鼠标按下 |
| `OnMouseUp()` | 鼠标抬起 |
| `OnMouseEnter()` | 鼠标移入 |
| `OnMouseExit()` | 鼠标移出 |
| `OnMouseOver()` | 鼠标悬停（每帧） |
| `OnMouseDrag()` | 鼠标拖拽（每帧） |
| `OnMouseUpAsButton()` | 同一物体上按下并抬起 |

## 注意事项

### 问题一：UI 穿透

`OnMouseDown` **不会自动屏蔽 UI**。当鼠标点击 UI 按钮时，如果按钮背后有 3D 物体，`OnMouseDown` 依然会触发。

**原因**：`OnMouseDown` 走的是物理射线（Physics Raycast），而 UI 走的是 `EventSystem`（Graphic Raycaster），两套系统互不感知。

**解决方案**：在 `OnMouseDown` 中手动判断鼠标是否在 UI 上：

```csharp
using UnityEngine.EventSystems;

void OnMouseDown()
{
    if (EventSystem.current.IsPointerOverGameObject())
        return; // 点在 UI 上，不处理 3D 物体点击

    // 正常处理点击
}
```

`IsPointerOverGameObject()` 会检测当前鼠标位置是否有 UI 元素，有则返回 `true`。

### 问题二：物体遮挡

`OnMouseDown` 只会触发最前面物体的回调，不会穿透。这是正确的行为——如果你想点击被挡住的物体，需要绕过遮挡物体（比如将其设为 Ignore Raycast 层）。

### 问题三：性能

`OnMouseDown` 每帧对**所有带 Collider 的物体**发射射线做检测。场景中有大量 Collider 时会有性能开销。不过对于大多数中小型项目，影响可以忽略不计。

### 问题四：仅限主相机

`OnMouseDown` 默认只通过 `Camera.main` 检测。如果你的场景有多相机或有自定义渲染管线，需要注意。

## 总结

简单交互直接用 `OnMouseDown` 就够了，记得加一行 UI 屏蔽
