---
title: Unity 2D 常用技巧
tags:
  - 2d
id: '329'
categories:
  - - unity
date: 2019-11-01 21:52:42
---

# 1. 点击屏幕选中物体
```csharp
// 核心是坐标转化：屏幕坐标 -> 世界坐标 -> 射线碰撞
Vector3 mousePos = Camera.main.ScreenToWorldPoint(Input.mousePosition);
Vector2 mousePos2D = new Vector2(mousePos.x, mousePos.y);
RaycastHit2D hit = Physics2D.Raycast(mousePos2D, Vector2.zero);
```

# 2. 多个血条显示
```csharp
// 核心是坐标转换：世界坐标转为屏幕坐标
// GameObject所在的世界坐标转为屏幕坐标
Vector3 position = Camera.main.WorldToScreenPoint(transform.position);
// 设置血条的位置，并调整偏移量
bloodGameObject.gameObject.transform.position = new Vector3(position.x-35, position.y + 110, position.z);

// 设置血条长度
image.rectTransform.SetSizeWithCurrentAnchors(RectTransform.Axis.Horizontal, originalSize \* percent);
```

# 3. 移动
```csharp
// 刚体移动使用rig.MovePosition，防止物体抖动
// 使用MoveTowards计算移动的新位置， 防止速度过快时总是偏离目标位置
if( Vector3.Distance(rig.position, targetPosition) > Mathf.Epsilon)
{
   float distance = attackMoveSpeed \* Time.deltaTime;
   Vector3 newPosition = Vector3.MoveTowards(rig.position, targetPosition, distance);
   rig.MovePosition(newPosition);
}
```
# 4. 拖拽物品
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.EventSystems;
using UnityEngine.UI;

public class ItemDrag : MonoBehaviour, IPointerDownHandler, IPointerUpHandler, IDragHandler
{

    // 鼠标起点
    private Vector2 originalLocalPointerPosition;
    // 面板起点
    private Vector3 originalLocalPanelPosition;
    // 当前物体
    private RectTransform panelRectTransform;
    // 父节点
    private RectTransform parentRectTransform;

    // 格子列表, 在gridList上一层添加一个新的空对象作为父节点
    private GameObject gridManager;
    // 记录物品拖拽前的位置
    private GameObject originalGrid;

    private static int siblingIndex = 0;


    void Awake()
    {
        panelRectTransform = transform as RectTransform;
        parentRectTransform = GameObject.FindGameObjectWithTag("ScrollRect").GetComponent<RectTransform>() as RectTransform;
        gridManager = GameObject.Find("GridManager");

    }

    // 按下鼠标
    public void OnPointerDown(PointerEventData eventData)
    {
        // 记录物品当前所在格子信息
        originalGrid = panelRectTransform.parent.gameObject;
        // 将物品设置在gridManager下，并设置层级，保证物品显示在整个背包之上
        panelRectTransform.SetParent(parentRectTransform.transform);
        siblingIndex = 10;
        panelRectTransform.SetSiblingIndex(siblingIndex);
        // 记录当前物品的起点位置
        originalLocalPanelPosition = panelRectTransform.localPosition;

        // 通过屏幕中的鼠标点，获取在父节点中的鼠标点
        // parentRectTransform 父节点
        // data.position当前鼠标位置
        // data.pressEventCamera当前事件的摄像机
        // originalLocalPointerPosition获取当前鼠标起点
        RectTransformUtility.ScreenPointToLocalPointInRectangle(parentRectTransform, eventData.position, eventData.pressEventCamera,
            out originalLocalPointerPosition);

        Debug.Log(originalLocalPointerPosition.x + " : " + originalLocalPointerPosition.y);
    }

    // 拖动
    public void OnDrag(PointerEventData eventData)
    {
        if(panelRectTransform == null  parentRectTransform == null)
        {
            return;
        }

        Vector2 localPointerPosition;
        // 获取鼠标本地位置
        if( RectTransformUtility.ScreenPointToLocalPointInRectangle(parentRectTransform, eventData.position, eventData.pressEventCamera,
            out localPointerPosition))
        {
            // 移动位置 = 本地鼠标位置 - 鼠标起点位置
            Vector3 offsetToOriginal = localPointerPosition - originalLocalPointerPosition;
            // 当前物品位置 = 当前物品起点位置 + 移动位置
            panelRectTransform.transform.localPosition = originalLocalPanelPosition + offsetToOriginal;
        }
    }

    

    public void OnPointerUp(PointerEventData eventData)
    {
        RaycastHit2D hit = Physics2D.Raycast(Input.mousePosition, -Vector2.up);
        
        if( hit.collider != null)
        {

            GameObject g = hit.transform.gameObject;
            Debug.Log(" hit " + g.tag + " ");
            Debug.Log(g.transform.Find("Item(Clone)"));
            if ( hit.collider.gameObject.tag == "Grid" && hit.collider.gameObject.transform.Find("Item(Clone)")==null)
            {
                transform.SetParent(hit.transform);
            }
            else
            {
                transform.SetParent(originalGrid.transform);
            }
        }
        else
        {
            Debug.Log("not hit");
            transform.SetParent(originalGrid.transform);
        }
        // 重置物品位置
        transform.localPosition = Vector3.zero;
    }

}
```


