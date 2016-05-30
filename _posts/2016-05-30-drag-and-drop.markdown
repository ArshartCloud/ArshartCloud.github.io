---
layout: post
title: "Unity: Drag & Drop"
date: 2016-05-30
---

通过Unity自带的GUI的drag和drop的接口，实现GUI的图片拖拽变换。

#### Drag

先构造

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
