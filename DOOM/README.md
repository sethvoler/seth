# 使用 JS 和 Canvas 来实现屏幕融化效果

我们来实现一下屏幕切换时的屏幕融化效果。效果如下：
![屏幕融化效果](./1.gif)

## 实现逻辑

1. 在将屏幕分为几列，以使它们可以独立移动。
2. 为每列分配一个 -100px 到 0 的高度值。同时还要做一步限制，相邻列的高度差在 50px 范围内。<br/>例如：第一列的高度若取了 -80px，则首先第二列的高度可以取 -130px 到 -30px，但是同时还要满足 -100px 到 0 的范围，所以第二列可以取 -100px 到 -30px 内的值。<br/><mark>**注意：**</mark> 这些值不是一成不变的，但是列之间的偏差越大，效果将变得越随机。将列值保持在相邻列的某个范围内的原因是要创建起伏的丘陵效果。
3. 以增加列的高度的方式来呈现降低列的视觉效果，显示前景图后面的图像。这也是一开始给高度分配负值的原因。

## 代码实现

### HTML

这里我们只需要一个简单的 canvas 元素

```html
<canvas  id="canvas"></canvas>
```

### JavaScript

获取 canvas，获取图片并存入数组 images，使用 drawImage 绘制偏移量列

```JavaScript
var cas = document.getElementById("canvas"),
    ctx = cas.getContext("2d"),
    images = [document.getElementById('image1'), document.getElementById('image2')],
    bgImage = 1,
    fgImage = 0,
```

配置动画参数，colSize 控制列的宽度，maxDev 控制列的最大 height 负值，maxDiff 控制相邻列之间的最大差值，fallSpeed 控制列下降的速度，columns 控制列数，height 控制动画变化范围的高度。

```JavaScript
settings = {
  colSize: 2,
  maxDev: 100,
  maxDiff: 50,
  fallSpeed: 6,
  columns: 200,
  height: 600,
},
```

在 init 函数中，设置列的初始值，并将要融化的图像绘制到画布上。我们将第一个元素设置为介于 0 到 maxDev 之间的随机数，然后为每个相邻列选择一个在我们设置的 maxDiff 范围内的随机值。

```JavaScript
function init() {
  ctx.drawImage(images[fgImage], 0, 0);

  for (var x = 0; x < settings.columns; x++) {
    if (x === 0) {
      y[x] = -Math.floor(Math.random() * settings.maxDev);
    } else {
      y[x] = y[x - 1] + (Math.floor(Math.random() * settings.maxDiff) - settings.maxDiff / 2);
    }

    if (y[x] > 0) {
      y[x] = 0;
    } else if (y[x] < -settings.maxDev) {
      y[x] = -settings.maxDev;
    }
  }
}
```

创建 down 函数。首先，在融化的图像后面将图像绘制到画布上。接下来，我们遍历各列，使其值以下降速度 fallSpeed 递增。如果该值不大于 0，则表示用户尚未看到效果，因此 y 列（yPos）保持为 0。如果该列值大于 0，则将 y 列的位置设置为该列的值。然后，使用 drawImage 通过将 yPos 偏移 y 来将列绘制到画布。

如果列值大于高度，则完成标志 down 保持为 true，若为 false，执行 requestAnimationFrame。

```JavaScript
function down() {
  ctx.drawImage(images[bgImage], 0, 0);

  for (var col = 0; col < settings.columns; col++) {
    y[col] += settings.fallSpeed;

    if (y[col] < 0) {
      done = false;
      yPos = 0;
    } else if (y[col] < settings.height) {
      done = false;
      yPos = y[col];
    } else {
      done = true;
    }

    ctx.drawImage(images[fgImage], col * settings.colSize, 0, settings.colSize, settings.height, col * settings.colSize, yPos, settings.colSize, settings.height);
  }
  if (!done) {
    requestAnimationFrame(down);
  }
}
```

<center style="color: red;font-size:20px;margin-top:20px;">祝大家新年快乐！</center>