---
title: 以贝塞尔曲线渲染一个SVG椭圆弧
date: 2018-10-28 23:09:46
tags: [Graphics, Math]
mathjax: true
---

> 本文翻译自：https://mortoray.com/2017/02/16/rendering-an-svg-elliptical-arc-as-bezier-curves/

我需要绘制椭圆和圆弧，如果没有它们就不能完成一个向量API。但苹果的核心图形库除了轴对齐圆弧外没有其他的东西。安卓有椭圆，但似乎也限制它轴对齐。这意味着我要将椭圆转换成一组贝塞尔曲线，用于这些后端API。

> 在本文中，另请参阅[The smooth sexy curves of a bezier spline (平滑性感的贝塞尔曲线)](https://mortoray.com/2017/02/02/the-smooth-sexy-curves-of-a-bezier-spline/)和[Stuffing curves into boxes: calculating the bounds (让方框囊括曲线：计算边界)](https://mortoray.com/2017/02/23/stuffing-curves-into-boxes-calculating-the-bounds/)。

## SVG圆弧表示法
SVG的圆弧指令允许绘制任何一种想要的弧形。由于[Fuse](https://www.fusetools.com/)的旧`Path`已经允许SVG路径数据，所以使用这个弧线的定义似乎是合理的。

SVG通过通过它们的端点定义弧：在哪里开始和结束。这些再与椭圆的半径和旋转组合。基于以上参数，就有四个可能的弧。然后还要囊括两个奇怪的标志来选择对应弧。

```
a radius-x radius-y x-axis-rotation large-arc-flag sweep-flag x y
```

这是从绘图角度指定弧的一种好方法。你知道两点相连，你也知道你想要哪个扫掠角。但问题是，这不是一个实际绘制弧线的好办法。它需要转换为一个中心点和要绘制的角度范围。

## 端点转换到中心点
SVG附录“椭圆弧实现注释”中有一个“从端点到中心点的参数化转换”算法。这很有帮助，因为该圆弧标记法有些不常见，很难在别处找到这个算法。

然而，它也有一些问题。标准中的示例弧实际上错了！它需要修复一下。

### 平方根
首个问题就是F6.5.2(下图公式)中的平方根：

![svg_f-6-5-2](svg_f-6-5-2.png)

当试图开方一个负数时，会产生一个虚数，它会变成$NaN$。每当输入的半径不够大，两个端点与椭圆链接时，就会出现这种问题。即使原数是一致的，浮点精度误差也会导致这个值略负。这对开方来说是不可忽略的：`sqrt(0)`是$0$，但`sqrt(-0.000001)`是$NaN$。

我对它的修复如下。从规范中我们分离出$\sqrt{pq}$如下所示：

$$\begin{array}{rll} dq &= r_x^2 y_1'^2 + r_y^2 x_1'^2 \\ \\ pq &= \frac{r_x^2 r_y^2 - dq}{dq} \\ \end{array}$$

问题是当所给的$cr > 1$时，$pq < 0$：

$$\begin{array}{rll} cr &= dq : (r_x^2 r_y^2) \\ \\ cr &= \frac{x_1'^2}{r_x^2} + \frac{y_1'^2}{r_y^2} \end{array} $$

考虑到这个系数，很容易地扩大半径。我们将半径乘以$\sqrt{cr}$。这将准确地将$cr$值减少到1（尽管如此，仍需要注意浮点精度问题），导致$pq == 0$。

> 完全方程在本文的末尾。我将执行`sqrt(max(0, pq))`；由于精确性，在放大r之后，pq值仍可能为负（实际上在使用中，出现了轻微的负值）。

### 反余弦
该算法还包括计算两个矢量之间的夹角。虽然已经有了计算的函数，但我还是决定使用他们的方程来确保$±$部分能够按预期工作。

![svg_f-6-5-4](svg_f-6-5-4.png)

通常情况下，我们需要担心这里的除法，但这个算法中的预过滤确保我们有非零的长度。

但它不能保证`arccos`的参数是有效的。由于浮点精度（又是它），该值可能略大于1，或略小于1。这对于`arccos`是不可忽视的，这在那些情况下仅返回一个`NaN`（准确地说结果是一个虚数）。在我的代码中，我添加了一个`clamp`方法来防止它。

`clamp`是有效的，因为理论上，方程不能产生范围在-1~1之外的值。这是一个测定过的角度，它有固定的数值范围。我得到的超出范围的值也只是略微超出范围而已。

## 从弧到贝塞尔曲线
我们可以使用这个中心点表示法把圆弧转换为一组贝塞尔曲线。这涉及到很多东西：椭圆的参数方程，它的倒数，以及一些我没有导出的复杂公式。

值得庆幸的是，L. Maisonobe发表了一篇[Drawing an elliptical arc using polylines, quadratic or cubic Bézier curves（使用折线、二次或三次贝塞尔曲线绘制椭圆弧）](http://www.spaceroots.org/documents/ellipse/elliptical-arc.pdf)的论文。我所要做的就是读它并翻译成代码。

### 参数
首先是获得椭圆的参数方程。椭圆的标准方程如下：

$$\left(\frac{x}{a}\right)^2 + \left(\frac{y}{b}\right)^2 = 1$$

这告诉我们，给定的`x, y`点是否位于椭圆上。但这对绘制椭圆不是很有用。取而代之的是，我们想要得到一个参数化的形式。下面是一个函数，传递一个值`t`，这表示椭圆上的一个伪角度，然后返回`x, y`坐标。这其中包括椭圆半径与X轴的转角（SVG弧必须的）。

```cs
static public float2 EllipticArcPoint( float2 c, float2 r, float xAngle, float t )
{
    return float2(
        c.X + r.X * Math.Cos(xAngle) * Math.Cos(t) - r.Y * Math.Sin(xAngle) * Math.Sin(t),
        c.Y + r.X * Math.Sin(xAngle) * Math.Cos(t) + r.Y * Math.Cos(xAngle) * Math.Sin(t));
}
```

我提到`t`是一个伪夹角，它不是一个真正的“角度”。它的角度是基于“如果一个人认为椭圆是一个被拉伸然后旋转的圆”（这是SVG规范中的用词）。

这个方程式的目的是，我们可以在圆弧定义的起始角和终止角之间迭代，以找到椭圆上的点。第二个函数叫做椭圆弧导数。这两个函数让我们计算一个近似于任何弧线的贝塞尔曲线。下表是我们所需的。

![maisonobe_3-4-1](maisonobe_3-4-1.png)

其中$\mathit{E}$是椭圆弧点函数，$\mathit{E}'$是椭圆弧导数函数，$\eta_1$和$\eta_2$是我们正在近似计算的弧的起始角和结束角。

我所要做的就是把角度范围细分成小节以获得良好的近似值。我不太理解这篇论文的误差计算，但是我发现[Joe Cridge的另一篇论文](https://www.joecridge.me/content/pdf/bezier-arcs.pdf)指出$\pi/2$的分割方式在相当高分辨率的设备上有潜在的一个像素误差。因此，我选择了$\pi/4$来保证平滑，即使是在高像素密度的移动设备上。

## 一个椭圆
![arc_svg_example](arc_svg_example.png)

把之前的一切组合在一起，我们就能够从SVG渲染示例了。这是建立在向量API上的，我从上一篇文章开始，对[平滑性感的贝塞尔曲线](https://mortoray.com/2017/02/02/the-smooth-sexy-curves-of-a-bezier-spline/)进行了研究。我的工作后端是苹果核心图形，但这个代码也将运行在安卓Canvas和Windows的System.Drawing上。通过自己计算贝塞尔曲线，我们不需要限制后端绘制弧的能力了。

本系列还有一篇文章即将面世。我们仍然需要计算这些图形的边界。这是衍生的另一件奇遇。

> 我在[Fuse](https://www.fusetools.com/)上的作品充满了有趣的代码。关注我的[Twitter](http://twitter.com/edaqa)或者[Facebook](https://www.facebook.com/mortoray/)以获得更多的见解和轶事。如果有什么特别的跨平台工具激起你的好奇心，[请让我知道](https://mortoray.com/about/?src=fuse)。

## 附录：端点到中心弧转换
这是Uno代码（和本文发布时间一样）用于从SVG弧转换为中心点表示法。

```cs
/**
    执行SVG 1.1 规范中详细说明的点到弧中心点参数的转换。
    F.6.5 从端点到中心点参数化转换

    @param r 必须是一个ref，以防它需要按SVG规范放大。
*/
internal static void EndpointToCenterArcParams( float2 p1, float2 p2, ref float2 r_, float xAngle, 
    bool flagA, bool flagS, out float2 c, out float2 angles )
{
    double rX = Math.Abs(r_.X);
    double rY = Math.Abs(r_.Y);

    //(F.6.5.1)
    double dx2 = (p1.X - p2.X) / 2.0;
    double dy2 = (p1.Y - p2.Y) / 2.0;
    double x1p = Math.Cos(xAngle)*dx2 + Math.Sin(xAngle)*dy2;
    double y1p = -Math.Sin(xAngle)*dx2 + Math.Cos(xAngle)*dy2;

    //(F.6.5.2)
    double rxs = rX * rX;
    double rys = rY * rY;
    double x1ps = x1p * x1p;
    double y1ps = y1p * y1p;
    // 当 `dq > rxs * rys` (见下文)，检查半径是否太小 `pq < 0`
    // cr 是比率 (dq : rxs * rys) 
    double cr = x1ps/rxs + y1ps/rys;
    if (cr > 1) {
        //缩放 rX,rY 同时 cr == 1
        var s = Math.Sqrt(cr);
        rX = s * rX;
        rY = s * rY;
        rxs = rX * rX;
        rys = rY * rY;
    }
    double dq = (rxs * y1ps + rys * x1ps);
    double pq = (rxs*rys - dq) / dq;
    double q = Math.Sqrt( Math.Max(0,pq) ); //使用Max防止浮点精度问题
    if (flagA == flagS)
        q = -q;
    double cxp = q * rX * y1p / rY;
    double cyp = - q * rY * x1p / rX;

    //(F.6.5.3)
    double cx = Math.Cos(xAngle)*cxp - Math.Sin(xAngle)*cyp + (p1.X + p2.X)/2;
    double cy = Math.Sin(xAngle)*cxp + Math.Cos(xAngle)*cyp + (p1.Y + p2.Y)/2;

    //(F.6.5.5)
    double theta = svgAngle( 1,0, (x1p-cxp) / rX, (y1p - cyp)/rY );
    //(F.6.5.6)
    double delta = svgAngle(
        (x1p - cxp)/rX, (y1p - cyp)/rY,
        (-x1p - cxp)/rX, (-y1p-cyp)/rY);
    delta = Math.Mod(delta, Math.PIf * 2 );
    if (!flagS)
        delta -= 2 * Math.PIf;

    r_ = float2((float)rX,(float)rY);
    c = float2((float)cx,(float)cy);
    angles = float2((float)theta, (float)delta);
}

static float svgAngle( double ux, double uy, double vx, double vy )
{
    var u = float2((float)ux, (float)uy);
    var v = float2((float)vx, (float)vy);
    //(F.6.5.4)
    var dot = Vector.Dot(u,v);
    var len = Vector.Length(u) * Vector.Length(v);
    var ang = Math.Acos( Math.Clamp(dot / len,-1,1) ); //浮点精度误差，略微超过值
    if ( (u.X*v.Y - u.Y*v.X) < 0)
        ang = -ang;
    return ang;
}
```

## 附录：JavaScript完整实现
> 本代码来自：[StackOverflow](https://stackoverflow.com/questions/43946153/approximating-svg-elliptical-arc-in-canvas-with-javascript/43952974#43952974)

```js
var canvas = document.getElementById("canvas");
var ctx = canvas.getContext("2d");

// M100,350
// a45,35 -30 0,1 50,-25

canvas.width = document.body.clientWidth;
canvas.height = document.body.clientHeight;
ctx.strokeWidth = 2;
ctx.strokeStyle = "#000000";
function clamp(value, min, max) {
  return Math.min(Math.max(value, min), max)
}

function svgAngle(ux, uy, vx, vy ) {
  var dot = ux*vx + uy*vy;
  var len = Math.sqrt(ux*ux + uy*uy) * Math.sqrt(vx*vx + vy*vy);

  var ang = Math.acos( clamp(dot / len,-1,1) );
  if ( (ux*vy - uy*vx) < 0)
    ang = -ang;
  return ang;
}

function generateBezierPoints(rx, ry, phi, flagA, flagS, x1, y1, x2, y2) {
  var rX = Math.abs(rx);
  var rY = Math.abs(ry);

  var dx2 = (x1 - x2)/2;
  var dy2 = (y1 - y2)/2;

  var x1p =  Math.cos(phi)*dx2 + Math.sin(phi)*dy2;
  var y1p = -Math.sin(phi)*dx2 + Math.cos(phi)*dy2;

  var rxs = rX * rX;
  var rys = rY * rY;
  var x1ps = x1p * x1p;
  var y1ps = y1p * y1p;

  var cr = x1ps/rxs + y1ps/rys;
  if (cr > 1) {
    var s = Math.sqrt(cr);
    rX = s * rX;
    rY = s * rY;
    rxs = rX * rX;
    rys = rY * rY;
  }

  var dq = (rxs * y1ps + rys * x1ps);
  var pq = (rxs*rys - dq) / dq;
  var q = Math.sqrt( Math.max(0,pq) );
  if (flagA === flagS)
    q = -q;
  var cxp = q * rX * y1p / rY;
  var cyp = - q * rY * x1p / rX;

  var cx = Math.cos(phi)*cxp - Math.sin(phi)*cyp + (x1 + x2)/2;
  var cy = Math.sin(phi)*cxp + Math.cos(phi)*cyp + (y1 + y2)/2;

  var theta = svgAngle( 1,0, (x1p-cxp) / rX, (y1p - cyp)/rY );

  var delta = svgAngle(
    (x1p - cxp)/rX, (y1p - cyp)/rY,
    (-x1p - cxp)/rX, (-y1p-cyp)/rY);

  delta = delta - Math.PI * 2 * Math.floor(delta / (Math.PI * 2));

  if (!flagS)
    delta -= 2 * Math.PI;

  var n1 = theta, n2 = delta;


  // E(n)
  // cx +acosθcosη−bsinθsinη
  // cy +asinθcosη+bcosθsinη
  function E(n) {
    var enx = cx + rx * Math.cos(phi) * Math.cos(n) - ry * Math.sin(phi) * Math.sin(n);
    var eny = cy + rx * Math.sin(phi) * Math.cos(n) + ry * Math.cos(phi) * Math.sin(n);
    return {x: enx,y: eny};
  }

  // E'(n)
  // −acosθsinη−bsinθcosη
  // −asinθsinη+bcosθcosη
  function Ed(n) {
    var ednx = -1 * rx * Math.cos(phi) * Math.sin(n) - ry * Math.sin(phi) * Math.cos(n);
    var edny = -1 * rx * Math.sin(phi) * Math.sin(n) + ry * Math.cos(phi) * Math.cos(n);
    return {x: ednx, y: edny};
  }

  var n = [];
  n.push(n1);

  var interval = Math.PI/4;

  while(n[n.length - 1] + interval < n2)
    n.push(n[n.length - 1] + interval)

  n.push(n2);

  function getCP(n1, n2) {
    var en1 = E(n1);
    var en2 = E(n2);
    var edn1 = Ed(n1);
    var edn2 = Ed(n2);

    var alpha = Math.sin(n2 - n1) * (Math.sqrt(4 + 3 * Math.pow(Math.tan((n2 - n1)/2), 2)) - 1)/3;

    console.log(en1, en2);

    return {
      cpx1: en1.x + alpha*edn1.x,
      cpy1: en1.y + alpha*edn1.y,
      cpx2: en2.x - alpha*edn2.x,
      cpy2: en2.y - alpha*edn2.y,
      en1: en1,
      en2: en2
    };
  }

  var cps = []
  for(var i = 0; i < n.length - 1; i++) {
    cps.push(getCP(n[i],n[i+1]));
  }


  return cps;
}

// M100,100
ctx.moveTo(100,100)
// a45,35 -30 0,1 50,-25
var rx = 45, ry=35,phi =  -30 * Math.PI / 180, fa = 0, fs = 1, x = 100, y = 100, x1 = x + 50, y1 = y - 25;

  var cps = generateBezierPoints(rx, ry, phi, fa, fs, x, y, x1, y1);

  var limit = 2;

  for(var i = 0; i < limit && i < cps.length; i++) {
    ctx.bezierCurveTo(cps[i].cpx1, cps[i].cpy1,
                      cps[i].cpx2, cps[i].cpy2,
                      i < limit - 1 ? cps[i].en2.x : x1, i < limit - 1 ? cps[i].en2.y : y1);
  }
ctx.stroke()
```