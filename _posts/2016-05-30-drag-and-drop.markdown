---
layout: post
title: "Unity: Drag & Drop"
date: 2016-05-30
---

通过Unity自带的GUI的drag和drop的接口，实现GUI的图片拖拽变换。

## Drag

先构造一个与simpleUI的demo极为相似的结构。

界面

![preview](/img/Blog/20160530163146.png)

架构

![struct](/img/Blog/20160530164647.png)

创建一个Drag函数。

{% highlight c++ %}

using System.Collections.Generic;
using UnityEngine;
using UnityEngine.EventSystems;
using UnityEngine.UI;
[RequireComponent(typeof(Image))]
public class Drag : MonoBehaviour, IBeginDragHandler, IDragHandler, IEndDragHandler
{
    public bool dragOnSurfaces = true;

    private Dictionary<int,GameObject> m_DraggingIcons = new Dictionary<int, GameObject>();
    private Dictionary<int, RectTransform> m_DraggingPlanes = new Dictionary<int, RectTransform>();

    public Sprite noneSprite;

}

{% endhighlight %}

这里继承了 IBeginDragHandler, IDragHandler, IEndDragHandler等Unity自带的Drag相关的接口，分别对应抓取开始，抓取时，抓取结束（放下）。

中间两个dictionary储存drag时创建的临时小图像。  
noneSprite指没有图像时显示的图像（避免一片空白）  

实现Drag的思路很简单，在开始时创建一个新的小图，并将原来的图用一个noneSprite的mask将被Drag的图像遮住（并不删除），在Drag时保持小图随鼠标移动，在放下后清除小图并去掉mask。相关的图像替换代码留在Drop中实现。  

{% highlight c++ %}

using System.Collections.Generic;
using UnityEngine;
using UnityEngine.EventSystems;
using UnityEngine.UI;

[RequireComponent(typeof(Image))]
public class Drag : MonoBehaviour, IBeginDragHandler, IDragHandler, IEndDragHandler
{
    public bool dragOnSurfaces = true;

    private Dictionary<int,GameObject> m_DraggingIcons = new Dictionary<int, GameObject>();
    private Dictionary<int, RectTransform> m_DraggingPlanes = new Dictionary<int, RectTransform>();

    public Sprite noneSprite;

    public void OnBeginDrag(PointerEventData eventData)
    {
        var canvas = FindInParents<Canvas>(gameObject);
        if (canvas == null)
            return;

        // We have clicked something that can be dragged.
        // What we want to do is create an icon for this.
        m_DraggingIcons[eventData.pointerId] = new GameObject("icon");

        m_DraggingIcons[eventData.pointerId].transform.SetParent(canvas.transform, false);
        m_DraggingIcons[eventData.pointerId].transform.SetAsLastSibling();

        var image = m_DraggingIcons[eventData.pointerId].AddComponent<Image>();
        // The icon will be under the cursor.
        // We want it to be ignored by the event system.
        var group = m_DraggingIcons[eventData.pointerId].AddComponent<CanvasGroup>();
        group.blocksRaycasts = false;

        image.sprite = GetComponent<Image>().sprite;
        image.SetNativeSize();

        GetComponent<Image>().overrideSprite = noneSprite;

        if (dragOnSurfaces)
            m_DraggingPlanes[eventData.pointerId] = transform as RectTransform;
        else
            m_DraggingPlanes[eventData.pointerId] = canvas.transform as RectTransform;

        SetDraggedPosition(eventData);
    }

    public void OnDrag(PointerEventData eventData)
    {
        if (m_DraggingIcons[eventData.pointerId] != null)
            SetDraggedPosition(eventData);
    }

    private void SetDraggedPosition(PointerEventData eventData)
    {
        if (dragOnSurfaces && eventData.pointerEnter != null && eventData.pointerEnter.transform as RectTransform != null)
            m_DraggingPlanes[eventData.pointerId] = eventData.pointerEnter.transform as RectTransform;

        var rt = m_DraggingIcons[eventData.pointerId].GetComponent<RectTransform>();
        Vector3 globalMousePos;
        if (RectTransformUtility.ScreenPointToWorldPointInRectangle(m_DraggingPlanes[eventData.pointerId], eventData.position, eventData.pressEventCamera, out globalMousePos))
        {
            rt.position = globalMousePos;
            rt.rotation = m_DraggingPlanes[eventData.pointerId].rotation;
        }
    }

    public void OnEndDrag(PointerEventData eventData)
    {
        if (m_DraggingIcons[eventData.pointerId] != null)
            Destroy(m_DraggingIcons[eventData.pointerId]);

        m_DraggingIcons[eventData.pointerId] = null;
        GetComponent<Image>().overrideSprite = null;
    }

    static public T FindInParents<T>(GameObject go) where T : Component
    {
        if (go == null)
            return null;
        var comp = go.GetComponent<T>();

        if (comp != null)
            return comp;

        var t = go.transform.parent;
        while (t != null && comp == null)
        {
            comp = t.gameObject.GetComponent<T>();
            t = t.parent;
        }
        return comp;
    }
}

{% endhighlight %}

## Drop

Drop函数和Drag其实可以合并为一个，都放在DropImage的对象上，同样，Drop需要继承一些接口。

{% highlight c++ %}

using System.Reflection;
using UnityEngine;
using UnityEngine.EventSystems;
using UnityEngine.UI;

[RequireComponent(typeof(Image))]
public class Drop : MonoBehaviour, IDropHandler, IPointerEnterHandler, IPointerExitHandler
{
    public Image containerImage;
    public Image receivingImage;
    public Sprite noneSprite;
    private Color normalColor;
    public Color highlightColor = Color.yellow;
}

{% endhighlight %}
