# d3-shape

可视化通常由离散图形标记组成, 比如 [symbols](#symbols), [arcs](#arcs), [lines](#lines) 和 [areas](#areas)。虽然条形的矩形可以很容易的使用 [SVG](http://www.w3.org/TR/SVG/paths.html#PathData) 或者 [Canvas](http://www.w3.org/TR/2dcontext/#canvaspathmethods) 来生成, 但是其他的比如圆形的扇形以及向心 `Catmull-Rom` 样条曲线就很复杂。这个模块提供了许多图形生成器以便使用。

与 `D3` 的其他特性一样，这些图形也是由数据驱动的: 每个图形生成器都暴露了一个如何将数据映射到可视化表现的访问器。例如你可以通过 [scaling](https://github.com/xswei/d3-scale) 定义一个时间序列的线条生成器以生成图表:

```js
var line = d3.line()
    .x(function(d) { return x(d.date); })
    .y(function(d) { return y(d.value); });
```

线条生成器可以计算 `SVG` `path` 元素的 `d` 属性：

```js
path.datum(data).attr("d", line);
```

或者也可以将其渲染到 `Canvas 2D` 上下文中:

```js
line.context(context)(data);
```

更多信息参考 [Introducing d3-shape](https://medium.com/@mbostock/introducing-d3-shape-73f8367e6d12).

## Installing

`NPM` 安装: `npm install d3-shape`. 也可以下载 [latest release](https://github.com/xswei/d3-shape/releases/latest). 此外还可以从 [d3js.org](https://d3js.org) 以 [standalone library](https://d3js.org/d3-shape.v1.min.js) 或作为 [D3 4.0](https://github.com/xswei/d3) 的一部分载入. 支持 `AMD`, `CommonJS` 以及基本的标签引入形式，如果使用标签引入则会暴露全局 `d3` 变量:

```html
<script src="https://d3js.org/d3-path.v1.min.js"></script>
<script src="https://d3js.org/d3-shape.v1.min.js"></script>
<script>

var line = d3.line();

</script>
```

[在浏览器中测试 `d3-shape`.](https://tonicdev.com/npm/d3-shape)

## API Reference

* [Arcs](#arcs)
* [Pies](#pies)
* [Lines](#lines)
* [Areas](#areas)
* [Curves](#curves)
* [Custom Curves](#custom-curves)
* [Links](#links)
* [Symbols](#symbols)
* [Custom Symbol Types](#custom-symbol-types)
* [Stacks](#stacks)

### Arcs

[<img alt="Pie Chart" src="https://raw.githubusercontent.com/d3/d3-shape/master/img/pie.png" width="295" height="295">](http://bl.ocks.org/mbostock/8878e7fd82034f1d63cf)[<img alt="Donut Chart" src="https://raw.githubusercontent.com/d3/d3-shape/master/img/donut.png" width="295" height="295">](http://bl.ocks.org/mbostock/2394b23da1994fc202e1)

`arc` 生成器用来在饼图或圆环图中生成 [circular(圆形)](https://en.wikipedia.org/wiki/Circular_sector) 或 [annular(环形)](https://en.wikipedia.org/wiki/Annulus_\(mathematics\)) 扇形。如果 [start](#arc_startAngle) 和 [end](#arc_endAngle) 之间的角度(*angular span*)差大于 [τ](https://en.wikipedia.org/wiki/Turn_\(geometry\)#Tau_proposal) 则 `arc` 生成器将会产生一个完整的圆或环。如果小于 `τ` 则生成的扇形可能有 [rounded corners(圆角)](#arc_cornerRadius) 和 [angular padding(角度间隙)](#arc_padAngle)。弧的中心总是在 ⟨0,0⟩; 可以使用 `transform` (参考 [SVG](http://www.w3.org/TR/SVG/coords.html#TransformAttribute), [Canvas](http://www.w3.org/TR/2dcontext/#transformations)) 来将其移动到指定的位置。

可以与 [pie generator](#pies) 对比，`pie` 生成器用来计算一组数据作为饼图或圆环图时所需要的角度信息；这些角度信息会被传递给 `arc` 生成器生成图形。

<a name="arc" href="#arc">#</a> d3.<b>arc</b>() [<源码>](https://github.com/d3/d3-shape/blob/master/src/arc.js "Source")

使用默认的设置创建一个新的 `arc` 生成器。

<a name="_arc" href="#_arc">#</a> <i>arc</i>(<i>arguments…</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/arc.js#L89 "Source")

根据指定的 *arguments* 生成 `arc`。*arguments* 是任意的; 它们只是简单地传递到 `arc` 生成器的访问器函数的对象。例如，根据默认的设置，传入的对象应该包含以下半径和角度信息:

```js
var arc = d3.arc();

arc({
  innerRadius: 0,
  outerRadius: 100,
  startAngle: 0,
  endAngle: Math.PI / 2
}); // "M0,-100A100,100,0,0,1,100,0L0,0Z"
```

如果半径和角度信息在构建生成器时已经被设置为常量，则不需要传入任何参数:

```js
var arc = d3.arc()
    .innerRadius(0)
    .outerRadius(100)
    .startAngle(0)
    .endAngle(Math.PI / 2);

arc(); // "M0,-100A100,100,0,0,1,100,0L0,0Z"
```

如果 `arc` 生成器拥有 [context](#arc_context) 则这个弧会被作为 [path method](http://www.w3.org/TR/2dcontext/#canvaspathmethods) 的调用序列渲染到对应的上下文中并返回空。否则，返回一个 [path data](http://www.w3.org/TR/SVG/paths.html#PathData) 字符串。

<a name="arc_centroid" href="#arc_centroid">#</a> <i>arc</i>.<b>centroid</b>(<i>arguments…</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/arc.js#L224 "Source")

计算由给定 *arguments* 生成的 [generated](#_arc )的中间点 [*x*, *y*]. *arguments* 是任意的，它们会被传递给 `arc` 生成器的访问器。为了与生成的弧保持一致，访问器必须是确定的。例如，相同的参数返回相同的值。中间点被定义为 ([startAngle](#arc_startAngle) + [endAngle](#arc_endAngle)) / 2 和 ([innerRadius](#arc_innerRadius) + [outerRadius](#arc_outerRadius)) / 2。例如:

[<img alt="Circular Sector Centroids" src="https://raw.githubusercontent.com/d3/d3-shape/master/img/centroid-circular-sector.png" width="250" height="250">](http://bl.ocks.org/mbostock/9b5a2fd1ce1a146f27e4)[<img alt="Annular Sector Centroids" src="https://raw.githubusercontent.com/d3/d3-shape/master/img/centroid-annular-sector.png" width="250" height="250">](http://bl.ocks.org/mbostock/c274877f647361f3df7d)

注意，中间点 **并不是几何中心**，因为几何中心点可能位于弧之外; 这个方法可以用来方便的对 `labels` 进行定位。

<a name="arc_innerRadius" href="#arc_innerRadius">#</a> <i>arc</i>.<b>innerRadius</b>([<i>radius</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/arc.js#L230 "Source")

如果指定了 *radius* 则将内半径设置为指定的函数或数值并返回当前 `arc` 生成器。如果没有指定 *radius* 则返回当前的内半径访问器，默认为:

```js
function innerRadius(d) {
  return d.innerRadius;
}
```

将内半径设置为函数在生成堆叠的极坐标条形图时非常有用，通常与 [sqrt scale](https://github.com/xswei/d3-scale#sqrt) 组合。更常见的是将内半径设置为常量用来生成 `donut` 或者 `pie` 图。如果外半径小于内半径则内外半径将会被互换。负值被看做 `0`。

<a name="arc_outerRadius" href="#arc_outerRadius">#</a> <i>arc</i>.<b>outerRadius</b>([<i>radius</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/arc.js#L234 "Source")

如果指定了 *radius* 则将外半径设置为指定的函数或数值并返回当前 `arc` 生成器。如果没有指定 *radius* 则返回当前的外半径访问器，默认为:

```js
function outerRadius(d) {
  return d.outerRadius;
}
```

将内半径设置为函数在生成 `coxcomb` 图或极坐标条形图时非常有用，通常与 [sqrt scale](https://github.com/xswei/d3-scale#sqrt) 组合。更常见的是将外半径设置为常量用来生成 `donut` 或者 `pie` 图。如果外半径小于内半径则内外半径将会被互换。负值被看做 `0`。

<a name="arc_cornerRadius" href="#arc_cornerRadius">#</a> <i>arc</i>.<b>cornerRadius</b>([<i>radius</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/arc.js#L238 "Source")

如果指定了 *radius* 则将拐角半径设置为指定的函数或数值并返回当前 `arc` 生成器。如果没有指定 *radius* 则返回当前的拐角半径访问器，默认为:

```js
function cornerRadius() {
  return 0;
}
```

如果拐角半径大于 `0` 则弧度的拐角将会适用指定半径大小的圆进行圆滑。对于扇形，会有两个拐角被圆滑处理，对于环形，所有的四个拐角都会被圆滑处理。拐角处理示意图如下:

[<img alt="Rounded Circular Sectors" src="https://raw.githubusercontent.com/d3/d3-shape/master/img/rounded-circular-sector.png" width="250" height="250">](http://bl.ocks.org/mbostock/e5e3680f3079cf5c3437)[<img alt="Rounded Annular Sectors" src="https://raw.githubusercontent.com/d3/d3-shape/master/img/rounded-annular-sector.png" width="250" height="250">](http://bl.ocks.org/mbostock/f41f50e06a6c04828b6e)

拐角半径不应该大于 ([outerRadius](#arc_outerRadius) - [innerRadius](#arc_innerRadius)) / 2。此外，对于弧长小于等于 `π` 的弧, 当两个相邻的拐角角相交时，拐角半径可以减小。这种情况更经常发生在内角。参考 [arc corners animation](http://bl.ocks.org/mbostock/b7671cb38efdfa5da3af) 中的插图。

<a name="arc_startAngle" href="#arc_startAngle">#</a> <i>arc</i>.<b>startAngle</b>([<i>angle</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/arc.js#L246 "Source")

如果指定了 *angle* 则将起始角度设置为指定的函数或数值并返回当前 `arc` 生成器。如果没有指定 *angle* 则返回当前的起始角度访问器，默认为:

```js
function startAngle(d) {
  return d.startAngle;
}
```

*angle* 以弧度的形式指定，`0` 表示 `12` 点钟方向并且顺时针方向为正。如果 |`endAngle` - `startAngle`| ≥ `τ` 则会绘制一个完整的扇形或圆环。

<a name="arc_endAngle" href="#arc_endAngle">#</a> <i>arc</i>.<b>endAngle</b>([<i>angle</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/arc.js#L250 "Source")

如果指定了 *angle* 则将终止角度设置为指定的函数或数值并返回当前 `arc` 生成器。如果没有指定 *angle* 则返回当前的终止角度访问器，默认为:

```js
function endAngle(d) {
  return d.endAngle;
}
```

*angle* 以弧度的形式指定，`0` 表示 `12` 点钟方向并且顺时针方向为正。如果 |`endAngle` - `startAngle`| ≥ `τ` 则会绘制一个完整的扇形或圆环。

<a name="arc_padAngle" href="#arc_padAngle">#</a> <i>arc</i>.<b>padAngle</b>([<i>angle</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/arc.js#L254 "Source")

如果指定了 *angle* 则将间隙角度设置为指定的函数或数值，并返回当前 `arc` 生成器。如果 *angle* 没有指定则返回当前间隙角度访问器，默认为:

```js
function padAngle() {
  return d && d.padAngle;
}
```

间隔角度会转换为一个在两个相邻的弧之间的确定的线性距离，定义为 [padRadius](#arc_padRadius) * padAngle，这个距离在弧的开始和结束处都是相等的。如果弧形成一个完整的圆或环，也就是 |endAngle - startAngle| ≥ τ, 则间隔角度会被忽略。

如果 [inner radius](#arc_innerRadius) 或扇(环)形状的角跨度相对于间隔角度很小，则在相邻的弧之间保持平行的边缘是不可能的。在这种情况下，弧的内边缘可能会靠近成一个点，类似于一个圆形的扇形区域。由于这个原因，间隔角度通常只应用于环形扇区（即当内半径大于 `0`），如图所示：

[<img alt="Padded Circular Sectors" src="https://raw.githubusercontent.com/d3/d3-shape/master/img/padded-circular-sector.png" width="250" height="250">](http://bl.ocks.org/mbostock/f37b07b92633781a46f7)[<img alt="Padded Annular Sectors" src="https://raw.githubusercontent.com/d3/d3-shape/master/img/padded-annular-sector.png" width="250" height="250">](http://bl.ocks.org/mbostock/99f0a6533f7c949cf8b8)

使用角度间隔时，推荐的角度间隔为 `outerRadius \* padAngle / sin(θ)`，其中 `θ` 是所有扇(环)形中角度跨度最小的角度值。例如，如果外半径为 `200` 像素，间隔角度为 `0.02` 弧度，则合理的 `θ` 为 `0.04`，合理的内半径为 `100` 像素。参考[arc padding animation](http://bl.ocks.org/mbostock/053fcc2295a445afab07).

通常情况下，间隔角度不会在 `arc` 生成器中直接设置，而是通过 [pie generator](#pies) 来计算出一个与具体数据相比合理的间隔角度；参考 [*pie*.padAngle](#pie_padAngle) 和 [pie padding animation](http://bl.ocks.org/mbostock/3e961b4c97a1b543fff2). 如果你使用了一个常量表示间隔角度，则它会倾向于从较小的弧跨度中减去不成比例的，可能会失真.

<a name="arc_padRadius" href="#arc_padRadius">#</a> <i>arc</i>.<b>padRadius</b>([<i>radius</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/arc.js#L242 "Source")

如果指定了 *radius* 则将间隔半径设置为指定的函数或数值并返回 `arc` 生成器。如果没有指定 `radius` 则返回当前的间隔半径访问器，默认为 `null`，此时间隔半径通过 `sqrt([innerRadius](#arc_innerRadius) * innerRadius + [outerRadius](#arc_outerRadius) * outerRadius)` 自动计算。间隔半径决定将两个相邻的扇(环)形分开的固定距离，定义为 `padRadius * [padAngle](#arc_padAngle)`.

<a name="arc_context" href="#arc_context">#</a> <i>arc</i>.<b>context</b>([<i>context</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/arc.js#L258 "Source")

如果指定了 *context* 则设置渲染上下文并返回 `arc` 生成器。如果 *context* 没有指定则返回当前的上下文，默认为 `null`。如果上下文非空，则 [generated arc](#_arc) 会被渲染到指定的上下文。否则会返回一个表示 `arc` 的 [path data](http://www.w3.org/TR/SVG/paths.html#PathData) 字符串。

### Pies

`pie` 生成器不会直接生成图形，但是会计算生成饼图或环形图所需要的角度信息，这些角度信息可以被传递给 [arc generator](#arcs)。

<a name="pie" href="#pie">#</a> d3.<b>pie</b>() [<源码>](https://github.com/d3/d3-shape/blob/master/src/pie.js "Source")

构建一个新的使用默认配置的 `pie` 生成器。

<a name="_pie" href="#_pie">#</a> <i>pie</i>(<i>data</i>[, <i>arguments…</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/pie.js#L14 "Source")

根据指定的 *data* 数组生成一组对象数组，其中每个对象包含每个传入的数据经过计算后的角度信息。可以包含其他的额外 *argements*，这些额外的参数会直接被传递给当前数据计算后生成的对象或饼图生成器的访问器。返回数组的长度与 *data* 长度一致，其中第 *i* 个元素与输入数据中的第 *i* 个元素对应。返回数组中的每个对象包含以下属性:

* `data` - 输入数据; 对应输入数组中的数据元素.
* `value` - `arc` 对应的 [value](#pie_value).
* `index` - `arc` 基于 `0` 的 [sorted index(排序后的索引)](#pie_sort).
* `startAngle` - `arc` 的 [start angle](#pie_startAngle).
* `endAngle` - `arc` 的 [end angle](#pie_endAngle).
* `padAngle` - `arc` 的 [pad angle](#pie_padAngle).

这种形式的设计可以兼容 `arc` 生成器的默认 [startAngle](#arc_startAngle), [endAngle](#arc_endAngle) 和 [padAngle](#arc_padAngle) 访问器。角度单位是任意的，但是如果你想将饼图生成器和 `arc` 生成器结合使用，则应该以弧度的形式指定角度值，其中 `12` 点钟方向为 `0` 度并且顺时针方向为正。

给定一个小数据集，下面为如何计算其每个数据的角度信息:

```js
var data = [1, 1, 2, 3, 5, 8, 13, 21];
var arcs = d3.pie()(data);
```

`pie()` [constructs(构造)](#pie) 一个默认的 `pie` 生成器。`pie()(data)` 为指定的数据集 [invokes(调用)](#_pie) 饼图生成器，返回一组对象数组:

```json
[
  {"data":  1, "value":  1, "index": 6, "startAngle": 6.050474740247008, "endAngle": 6.166830023713296, "padAngle": 0},
  {"data":  1, "value":  1, "index": 7, "startAngle": 6.166830023713296, "endAngle": 6.283185307179584, "padAngle": 0},
  {"data":  2, "value":  2, "index": 5, "startAngle": 5.817764173314431, "endAngle": 6.050474740247008, "padAngle": 0},
  {"data":  3, "value":  3, "index": 4, "startAngle": 5.468698322915565, "endAngle": 5.817764173314431, "padAngle": 0},
  {"data":  5, "value":  5, "index": 3, "startAngle": 4.886921905584122, "endAngle": 5.468698322915565, "padAngle": 0},
  {"data":  8, "value":  8, "index": 2, "startAngle": 3.956079637853813, "endAngle": 4.886921905584122, "padAngle": 0},
  {"data": 13, "value": 13, "index": 1, "startAngle": 2.443460952792061, "endAngle": 3.956079637853813, "padAngle": 0},
  {"data": 21, "value": 21, "index": 0, "startAngle": 0.000000000000000, "endAngle": 2.443460952792061, "padAngle": 0}
]
```

需要注意的是，返回的数组与传入的数据集的次序是一致的，无论数据元素的值大小。

<a name="pie_value" href="#pie_value">#</a> <i>pie</i>.<b>value</b>([<i>value</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/pie.js#L54 "Source")

如果指定了 *value* 则设置当前饼图生成器的值访问器为指定的函数或数值，并返回当前饼图生成器。如果没有指定 *value* 则返回当前的值访问器默认为:

```js
function value(d) {
  return d;
}
```

当生成饼图时，值访问器会为传入的数据的每个元素调用并传递当前数据元素 *d*, 索引 `i` 以及当前数组 `data` 三个参数。默认的值访问器假设传入的数据每个元素为数值类型，或者可以使用 [valueOf](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/valueOf) 转为数值类型的值。如果你的数据不是简单的数值，你应该指定一个返回数值类型的值访问器。例如:

```js
var data = [
  {"number":  4, "name": "Locke"},
  {"number":  8, "name": "Reyes"},
  {"number": 15, "name": "Ford"},
  {"number": 16, "name": "Jarrah"},
  {"number": 23, "name": "Shephard"},
  {"number": 42, "name": "Kwon"}
];

var arcs = d3.pie()
    .value(function(d) { return d.number; })
    (data);
```

这与 [mapping](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) 类似，在调用饼图生成器之前，对数据进行预处理:

```js
var arcs = d3.pie()(data.map(function(d) { return d.number; }));
```

访问器的好处是输入数据仍然与返回的对象相关联，从而使访问数据的其他字段变得更容易，例如设置颜色或添加文本标签。

<a name="pie_sort" href="#pie_sort">#</a> <i>pie</i>.<b>sort</b>([<i>compare</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/pie.js#L62 "Source")

如果指定了 *compare* 则将数据比较函数设置为指定的函数并返回饼图生成器。如果没有指定 *compare* 则返回当前的数据对比函数，默认为 `null`。如果数据比较函数和值比较函数都为 `null` 则返回的 `arc` 会保持数据的次序。否则，返回的结果会安装相应的比较函数进行排序。设置数据对比函数默认会将 [value comparator(值比较函数)](#pie_sortValues) 设置为 `null`。

*compare* 函数会传递两个参数 *a* 和 *b*, 每个元素都来自输入数据。如果数据 *a* 对应的扇形在 *b* 前面，则比较函数应该返回小于 `0` 的值; 如果 *a* 对应的扇形在 *b* 的后面则比较函数应返回大于 `0` 的值;返回 `0` 表示 *a* 和 *b* 的相对位置不做任何调整。例如根据 `name` 对生成的扇形数组进行排序:

```js
pie.sort(function(a, b) { return a.name.localeCompare(b.name); });
```

排序操作不会影响 [generated arc array(生成的数组次序)](#_pie), 生成的数据次序与传入的数组次序保持一致。排序操作是通过修改每个生成的元素的起始角度值来实现排序的。

<a name="pie_sortValues" href="#pie_sortValues">#</a> <i>pie</i>.<b>sortValues</b>([<i>compare</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/pie.js#L58 "Source")

如果指定了 *compare* 则将 `value` 比较函数设置为指定的函数并返回当前的饼图生成器。如果没有指定 *compare* 则返回当前的值比较函数，默认为降序。默认的值比较函数实现形式为:

```js
function compare(a, b) {
  return b - a;
}
```

如果数据比较函数和值比较函数都为 `null` 则生成的数组次序与输入数据的次序保持一致。否则，数据会按照数据比较函数进行排序。设置值比较函数默认将 [data comparator](#pie_sort) 设置为 `null`.

值比较函数与 [data comparator](#pie_sort) 类似，只不过两个参数 *a* 和 *b* 是经过 [value accessor](#pie_value) 计算之后的值，而不是原始的数据元素。如果 *a* 应该在 *b* 前则返回小于 `0` 的值，如果 *a* 应该在 *b* 后面则返回大于 `0` 的值。返回 `0` 表示 *a* 和 *b* 的相对位置不改变。例如根据值进行排序:

```js
pie.sortValues(function(a, b) { return a - b; });
```

排序操作不会影响 [generated arc array(生成的数组次序)](#_pie), 生成的数据次序与传入的数组次序保持一致。排序操作是通过修改每个生成的元素的起始角度值来实现排序的。

<a name="pie_startAngle" href="#pie_startAngle">#</a> <i>pie</i>.<b>startAngle</b>([<i>angle</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/pie.js#L66 "Source")

如果指定了 *angle* 则将饼图的布局起始角度设置为指定的函数或数值并返回饼图生成器。如果没有指定则返回当前起始角度访问器默认为:

```js
function startAngle() {
  return 0;
}
```

起始角度是整个饼图的开始角度，也就是第一个扇区的开始角度。起始角度访问器只会调用一次，并传递当前数据为参数，其中 `this` 指向 [pie generator](#_pie)。*angle* 的单位是任意的，但是如果要将饼图生成器与弧生成器结合使用则应该以弧度指定，`12点钟` 方向为 `0` 度方向，并且顺时针为正。

<a name="pie_endAngle" href="#pie_endAngle">#</a> <i>pie</i>.<b>endAngle</b>([<i>angle</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/pie.js#L70 "Source")

如果指定了 *angle* 则将整个饼图的终止角度设置为指定的函数或数值并返回当前饼图生成器。如果没有指定 *angle* 则返回当前的终止角度访问器。默认为:

```js
function endAngle() {
  return 2 * Math.PI;
}
```

终止角度也就是整个饼图的结束角度，即最后一个扇区的终止角度。终止角度访问器会被调用一次病传递当前数据，其中 `this` 指向 
[pie generator](#_pie)，角度的单位是任意的，但是如果要将饼图生成器与弧生成器结合使用则应该以弧度指定，`12点钟` 方向为 `0` 度方向，并且顺时针为正。

终止角度可以被设置为 [startAngle](#pie_startAngle) ± τ，这样就能保证 |endAngle - startAngle| ≤ τ.

<a name="pie_padAngle" href="#pie_padAngle">#</a> <i>pie</i>.<b>padAngle</b>([<i>angle</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/pie.js#L74 "Source")

如果指定了 *angle* 则将饼图扇形之间的间隔设置为指定的函数或数值，并返回当前饼图生成器。如果没有指定 *angle* 则返回当前默认的间隔角度访问器，默认为:

```js
function padAngle() {
  return 0;
}
```

这里的间隔角度也就是两个相邻的扇形之间的间隔。间隔角度的总和等于指定的角度乘以输入数据数组中的元素数量，最大为 |endAngle - startAngle|；然后，剩余的间隔按比例按比例分配，这样每个弧的相对面积就会被保留下来。参考 [pie padding animation](http://bl.ocks.org/mbostock/3e961b4c97a1b543fff2) 获取更详细的说明。间隔访问器只会被调用一次，并传递当前数据集，其中 `this` 上下文指向 [pie generator](#_pie)。角度的单位是任意的，但是如果要将饼图生成器与弧生成器结合使用则应该以弧度指定。

### Lines

[<img width="295" height="154" alt="Line Chart" src="https://raw.githubusercontent.com/d3/d3-shape/master/img/line.png">](http://bl.ocks.org/mbostock/1550e57e12e73b86ad9e)

`line` 生成器可以用来生成线条图需要的 [spline](https://en.wikipedia.org/wiki/Spline_\(mathematics\)) 或 [polyline](https://en.wikipedia.org/wiki/Polygonal_chain)。线条也可以被用在其他的可视化类型中，比如 [hierarchical edge bundling](http://bl.ocks.org/mbostock/7607999)。

<a name="line" href="#line">#</a> d3.<b>line</b>() [<源码>](https://github.com/d3/d3-shape/blob/master/src/line.js "Source")

使用默认的设置构造一个 `line` 生成器。

Constructs a new line generator with the default settings.

<a name="_line" href="#_line">#</a> <i>line</i>(<i>data</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/line.js#L14 "Source")

根据指定的 *data* 数组生成一个线条。根据与线条生成器的关联的 [curve](#line_curve)，输入数据 *data* 可能需要根据 *x* 值进行排序。如果线条生成器有 [context](#line_context)，则线条会通过 [path method](http://www.w3.org/TR/2dcontext/#canvaspathmethods) 被渲染到指定的上下文中，否则返回一个 [path data](http://www.w3.org/TR/SVG/paths.html#PathData) 字符串。

<a name="line_x" href="#line_x">#</a> <i>line</i>.<b>x</b>([<i>x</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/line.js#L34 "Source")

如果指定了 *x* 则将 `x` 访问器设置为指定的函数或数值并返回当前 `line` 生成器。如果没有指定 *x* 则返回当前 `x` 访问器，默认为:

```js
function x(d) {
  return d[0];
}
```

在线条被 [generated](#_line) 时，`x` 访问器将会为输入数据中每一个 [defined](#line_defined) 的元素进行调用，并传入当前元素 `d`, 当前索引 `i` 以及当前所有元素 `data` 三个参数。默认的 `x` 访问器假设输入的数据为一个数值类型的二维数组。如果你的数据是其他的格式或者你在渲染前想进一步处理，则应该指定一个自定义的访问器。比如，如果你的 `x` 为 [time scale](https://github.com/xswei/d3-scale#time-scales) 并且 `y` 为 [linear scale](https://github.com/xswei/d3-scale#linear-scales):

```js
var data = [
  {date: new Date(2007, 3, 24), value: 93.24},
  {date: new Date(2007, 3, 25), value: 95.35},
  {date: new Date(2007, 3, 26), value: 98.84},
  {date: new Date(2007, 3, 27), value: 99.92},
  {date: new Date(2007, 3, 30), value: 99.80},
  {date: new Date(2007, 4,  1), value: 99.47},
  …
];

var line = d3.line()
    .x(function(d) { return x(d.date); })
    .y(function(d) { return y(d.value); });
```

<a name="line_y" href="#line_y">#</a> <i>line</i>.<b>y</b>([<i>y</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/line.js#L38 "Source")

如果指定了 *y* 则将 `y` 访问器设置为指定的函数或数值并返回当前 `line` 生成器。如果没有指定 *y* 则返回当前 `y` 访问器，默认为:

```js
function y(d) {
  return d[1];
}
```

在线条被 [generated](#_line) 时，`x` 访问器将会为输入数据中每一个 [defined](#line_defined) 的元素进行调用，并传入当前元素 `d`, 当前索引 `i` 以及当前所有元素 `data` 三个参数。默认的 `x` 访问器假设输入的数据为一个数值类型的二维数组。参考 [*line*.x](#line_x) 获取更多信息。

<a name="line_defined" href="#line_defined">#</a> <i>line</i>.<b>defined</b>([<i>defined</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/line.js#L42 "Source")

如果指定了 *defined* 则将已定义的访问器设置为指定的函数或布尔值。如果没有指定 *defined* 则返回当前默认的已定义的访问器，默认为:

```js
function defined() {
  return true;
}
```

默认的访问器假设输入数据总是被定义的。当线条被 [generated](#_line) 时，定义的访问器会为输入数据的每个元素定义，并传递当前数据元素 `d`, 索引 `i` 以及数组 `data` 作为三个元素。如果给定的元素有定义的(*i.e.* 已定义的访问器为当前元素返回真值), 则 [x](#line_x) 和 [y](#line_y) 访问器会对其进行评估并将当前点添加到线段中生成一个新的线段。否则会跳过当前数据元素，并结束当前线段，在下一个点重新开始一个新的线段。此时的结果会如下所示:

[<img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/line-defined.png" width="480" height="250" alt="Line with Missing Data">](http://bl.ocks.org/mbostock/0533f44f2cfabecc5e3a)

注意，如果一个线段仅由一个点组成，那么它可能看起来是不可见的，除非用圆形或方形的 [line caps(线帽)](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/stroke-linecap) 呈现。此外一些曲线比如 [curveCardinalOpen](#curveCardinalOpen) 如果包含多个点会被渲染成为一个可见的线段。

<a name="line_curve" href="#line_curve">#</a> <i>line</i>.<b>curve</b>([<i>curve</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/line.js#L46 "Source")

如果指定了 *curve* 则表示设置当前的 [curve factory(曲线插值方法)](#curves) 并返回线条生成器。如果没有指定 *curve* 则返回当前的线条插值方式，默认为 [curveLinear](#curveLinear).

<a name="line_context" href="#line_context">#</a> <i>line</i>.<b>context</b>([<i>context</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/line.js#L50 "Source")

如果指定了 *context* 则设置上下文并返回当前线条生成器。如果没有指定 *context* 则返回当前的上下文，默认为 `null`。如果上下文不为空，则线条生成器会调用 [path method](http://www.w3.org/TR/2dcontext/#canvaspathmethods) 被渲染到当前上下文中。否则会返回一个 [path data](http://www.w3.org/TR/SVG/paths.html#PathData) 字符串用来表示生成的线条。

<a name="lineRadial" href="#lineRadial">#</a> d3.<b>lineRadial</b>() [<源码>](https://github.com/d3/d3-shape/blob/master/src/lineRadial.js "Source")

<img alt="Radial Line" width="250" height="250" src="https://raw.githubusercontent.com/d3/d3-shape/master/img/line-radial.png">

使用默认的设置构造一个新的 `radial line(径向线条)` 生成器。径向线条生成器类似于笛卡尔坐标系下的 [line generator](#line)，只不过 [x](#line_x) 和 [y](#line_y) 访问器被替换成了 [angle](#lineRadial_angle) 和 [radius](#lineRadial_radius) 访问器。径向线条的生成总是相对于 ⟨0,0⟩，但是你可以使用坐标变换调整其位置。参考 [SVG](http://www.w3.org/TR/SVG/coords.html#TransformAttribute), [Canvas](http://www.w3.org/TR/2dcontext/#transformations) 的坐标变换。

<a name="_lineRadial" href="#_lineRadial">#</a> <i>lineRadial</i>(<i>data</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/lineRadial.js#L4 "Source")

等价于 [*line*](#_line).

<a name="lineRadial_angle" href="#lineRadial_angle">#</a> <i>lineRadial</i>.<b>angle</b>([<i>angle</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/lineRadial.js#L7 "Source")

等价于 [*line*.x](#line_x), 只不过此访问器返回的是弧度，`0` 度在 - *y* (12 点钟方向).

<a name="lineRadial_radius" href="#lineRadial_radius">#</a> <i>lineRadial</i>.<b>radius</b>([<i>radius</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/lineRadial.js#L8 "Source")

等价于 [*line*.y](#line_y),只不过此访问器返回一个半径值，也就是距离 ⟨0,0⟩ 的距离.

<a name="lineRadial_defined" href="#lineRadial_defined">#</a> <i>lineRadial</i>.<b>defined</b>([<i>defined</i>])

等价于 [*line*.defined](#line_defined).

<a name="lineRadial_curve" href="#lineRadial_curve">#</a> <i>lineRadial</i>.<b>curve</b>([<i>curve</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/lineRadial.js#L10 "Source")

等价于 [*line*.curve](#line_curve). 注意 [curveMonotoneX](#curveMonotoneX) 或 [curveMonotoneY](#curveMonotoneY) 插值方式不被推荐用在径向线条布局中，因为这两种插值方式假设 *x* 或 *y* 是单调的。

<a name="lineRadial_context" href="#lineRadial_context">#</a> <i>lineRadial</i>.<b>context</b>([<i>context</i>])

等价于 [*line*.context](#line_context).

### Areas

[<img alt="Area Chart" width="295" height="154" src="https://raw.githubusercontent.com/d3/d3-shape/master/img/area.png">](http://bl.ocks.org/mbostock/3883195)[<img alt="Stacked Area Chart" width="295" height="154" src="https://raw.githubusercontent.com/d3/d3-shape/master/img/area-stacked.png">](http://bl.ocks.org/mbostock/3885211)[<img alt="Difference Chart" width="295" height="154" src="https://raw.githubusercontent.com/d3/d3-shape/master/img/area-difference.png">](http://bl.ocks.org/mbostock/3894205)

`area generator(区域生成器)` 用来在 `area` 图中生成区域图。一个区域图由两条边界 [lines](#lines) 定义，可以是曲线或折线。通常情况下两条边界线共享一个 [*x*-values](#area_x) ([x0](#area_x0) = [x1](#area_x1))，仅仅是 *y*-value ([y0](#area_y0) 和 [y1](#area_y1)) 不一样。大多数情况下，`y0` 会被定义为一个常量 [zero](http://www.vox.com/2015/11/19/9758062/y-axis-zero-chart). 第一条线 (也就是 <i>上侧线</i>) 由 `x1` 和 `y1` 定义渲染。而第二条线 (也就是 <i>基线</i>) 由 `x0` 和 `y0` 定义渲染。有了 [curveLinear](#curveLinear) [curve](#area_curve)，就可以生成一个顺时针方向的多边形。

<a name="area" href="#area">#</a> d3.<b>area</b>() [<源码>](https://github.com/d3/d3-shape/blob/master/src/area.js "Source")

使用默认的设置构建一个区域生成器。

<a name="_area" href="#_area">#</a> <i>area</i>(<i>data</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/area.js#L17 "Source")

根据指定的一组数据 *data*。根据这个区域生成器的相关 [curve](#area_curve) ，给定的输入数据可能需要在传递给区域生成器之前按 `x`- 值排序。如果区域生成器被指定了 [context](#line_context) 则区域图会调用 [path method](http://www.w3.org/TR/2dcontext/#canvaspathmethods) 被渲染到指定的上下文上，并且这个方法返回 `void`。否则会返回一个表示 [path data](http://www.w3.org/TR/SVG/paths.html#PathData) 的字符串。

<a name="area_x" href="#area_x">#</a> <i>area</i>.<b>x</b>([<i>x</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/area.js#L59 "Source")

如果指定了 *x* 则设置 [x0](#area_x0) 为 *x* 并且设置 [x1](#area_x1) 为 `null`，返回当前区域生成器。如果没有指定 *x* 则返回当前 `x0` 访问器。

<a name="area_x0" href="#area_x0">#</a> <i>area</i>.<b>x0</b>([<i>x</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/area.js#L63 "Source")

如果指定看 *x* 则将 `x0` 访问器设置为指定的函数或数值并返回当前区域生成器。如果没有指定 *x* 则返回当前 `x0` 生成器, 默认为:

```js
function x(d) {
  return d[0];
}
```

在区域被 [generated](#_area)，`x0` 访问器会依次为输入的数据元素调用。并传递当前数据 `d`, 索引 `i` 以及数据数组 `data` 三个参数。默认的 `x0` 访问器假设输入的数据为一个二元数值数组。如果你的数据是其他的不同的格式，或者需要在渲染前进行转换则你需要设置一个自定义的访问器。比如 `x` 为 [time scale](https://github.com/xswei/d3-scale#time-scales) 并且 `y` 为 [linear scale](https://github.com/xswei/d3-scale#linear-scales):

```js
var data = [
  {date: new Date(2007, 3, 24), value: 93.24},
  {date: new Date(2007, 3, 25), value: 95.35},
  {date: new Date(2007, 3, 26), value: 98.84},
  {date: new Date(2007, 3, 27), value: 99.92},
  {date: new Date(2007, 3, 30), value: 99.80},
  {date: new Date(2007, 4,  1), value: 99.47},
  …
];

var area = d3.area()
    .x(function(d) { return x(d.date); })
    .y1(function(d) { return y(d.value); })
    .y0(y(0));
```

<a name="area_x1" href="#area_x1">#</a> <i>area</i>.<b>x1</b>([<i>x</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/area.js#L67 "Source")

如果制定了 *x* 则设置 `x1` 访问器为指定的函数或数值并返回区域生成器。如果 `x` 没有指定则返回当前的 `x1` 访问器，默认为 `null` 表示先前计算的 `x0` 值应该为 `x1` 值重用。

当一个区域图被 [generated](#_area) 时，`x1` 访问器将会为每个定义的元素调用。并传递当前元素 `d`, 索引 `i` 以及数据数组三个参数。参考 [*area*.x0](#area_x0) 获取更多信息。

<a name="area_y" href="#area_y">#</a> <i>area</i>.<b>y</b>([<i>y</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/area.js#L71 "Source")

如果指定了 *y* 则设置 [y0](#area_y0) 为 *y* 并设置 [y1](#area_y1) 为 `null`, 返回区域生成器。如果 *y* 没有被指定则返回当前的 `y0` 访问器。

<a name="area_y0" href="#area_y0">#</a> <i>area</i>.<b>y0</b>([<i>y</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/area.js#L75 "Source")

如果指定了 *y* 则设置 *y0* 访问器为指定的函数或数值并返回当前区域生成器。如果没有指定 *y* 则返回当前 *y0* 访问器默认为:

```js
function y() {
  return 0;
}
```

在区域被 [generated](#_area) 时，*y0* 访问器会为每个输入的元素调用并传递当前数据元素 `d`, 当前索引 `i` 以及数据数组 `data`。参考 [*area*.x0](#area_x0) 获取更多信息。

<a name="area_y1" href="#area_y1">#</a> <i>area</i>.<b>y1</b>([<i>y</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/area.js#L79 "Source")

如果指定了 *y* 则将 `y1` 访问器设置为指定的函数或数值并返回当前区域生成器。如果没有指定 `y` 则返回当前 `y1` 访问器默认为：

```js
function y(d) {
  return d[1];
}
```

可以使用空的访问器以表明 [y0](#area_y0) 会被 `y1` 复用，`y1` 访问器会为每个定义的元素调用并传递当前数据元素 `d`, 索引 `i` 以及当前数组 `data` 三个参数。参考 [*area*.x0](#area_x0) 获取更多信息。

<a name="area_defined" href="#area_defined">#</a> <i>area</i>.<b>defined</b>([<i>defined</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/area.js#L96 "Source")

如果指定了 *defined* 则将定义访问器设置为指定的函数或布尔值并返回区域生成器。如果没有指定 *defined* 则返回当前定义访问器默认为：

```js
function defined() {
  return true;
}
```

默认的访问器假设输入数据都是定义的。当区域图被生成时，已经定义的访问器会为每个数据元素调用，并传递当前数据元素 `d`, 索引 `i` 以及数组 `data` 三个参数。如果给定的元素时定义的(*i.e.* 访问器返回真值)，[x0](#area_x0), [x1](#area_x1), [y0](#area_y0) 和 [y1](#area_y1) 访问器会为当前元素评估然后将坐标加入到区域分段中。否则会跳过当前元素并结束当前区域分段，并为下一个定义的数据新开一个分段。此时的结果会表现为分段的区域图:

[<img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/area-defined.png" width="480" height="250" alt="Area with Missing Data">](http://bl.ocks.org/mbostock/3035090)

需要注意的是，如果只包含一个点则可能看起来是不可见的除非用圆形或方形的 [line caps](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/stroke-linecap) 呈现。此外，一些曲线，如 [curveCardinalOpen](#curveCardinalOpen) ，如果它包含多个点，则只呈现一个可见的段。

<a name="area_curve" href="#area_curve">#</a> <i>area</i>.<b>curve</b>([<i>curve</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/area.js#L100 "Source")

如果指定了 *curve* 则将 [curve factory](#curves) 设置为指定的 *curve* 并返回区域生成器。如果没有指定 *curve* 则返回默认的插值方式，默认为 [curveLinear](#curveLinear)。

<a name="area_context" href="#area_context">#</a> <i>area</i>.<b>context</b>([<i>context</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/area.js#L104 "Source")

如果指定了 *context* 则将区域生成器的上下文设置为指定的 *context*。如果没有指定 *context* 则返回当前上下文，默认为 `null`。如果上下文非 `null`, 在 [generated area](#_area) 时会调用 [path method](http://www.w3.org/TR/2dcontext/#canvaspathmethods) 将区域渲染到指定的上下文中，否则返回一个表示轮廓的 [path data](http://www.w3.org/TR/SVG/paths.html#PathData) 字符串。

<a name="area_lineX0" href="#area_lineX0">#</a> <i>area</i>.<b>lineX0</b>() [<源码>](https://github.com/d3/d3-shape/blob/master/src/area.js#L83 "Source")
<br><a name="area_lineY0" href="#area_lineY0">#</a> <i>area</i>.<b>lineY0</b>() [<源码>](https://github.com/d3/d3-shape/blob/master/src/area.js#L84 "Source")

返回一个新的 [line generator](#lines), 内置了当前区域生成器定义的 [defined accessor](#area_defined), [curve](#area_curve) 和 [context](#area_context)。线条的 [*x*-accessor](#line_x) 为区域生成器的 [*x0*-accessor](#area_x0), 线条的 [*y*-accessor](#line_y) 为区域生成器的 [*y0*-accessor](#area_y0).

<a name="area_lineX1" href="#area_lineX1">#</a> <i>area</i>.<b>lineX1</b>() [<源码>](https://github.com/d3/d3-shape/blob/master/src/area.js#L92 "Source")

返回一个新的 [line generator](#lines) 内置了当前区域生成器定义的 [defined accessor](#area_defined), [curve](#area_curve) 和 [context](#area_context).线条的 [*x*-accessor](#line_x) 为区域生成器的 [*x1*-accessor](#area_x1), 线条的 [*y*-accessor](#line_y) 为区域生成器的 [*y0*-accessor](#area_y0).

<a name="area_lineY1" href="#area_lineY1">#</a> <i>area</i>.<b>lineY1</b>() [<源码>](https://github.com/d3/d3-shape/blob/master/src/area.js#L88 "Source")

返回一个新的 [line generator](#lines)，内置了区域生成器的 [defined accessor](#area_defined), [curve](#area_curve) and [context](#area_context). 线条的 [*x*-accessor](#line_x) 为区域生成器的 [*x0*-accessor](#area_x0), 线条的 [*y*-accessor](#line_y) 为区域生成器的 [*y1*-accessor](#area_y1).

<a name="areaRadial" href="#areaRadial">#</a> d3.<b>areaRadial</b>() [<源码>](https://github.com/d3/d3-shape/blob/master/src/areaRadial.js "Source")

<img alt="Radial Area" width="250" height="250" src="https://raw.githubusercontent.com/d3/d3-shape/master/img/area-radial.png">

使用默认的设置构造一个新的径向区域生成器。径向区域生成器类似于标准的笛卡尔坐标系下的 [area generator](#area) 只不过 [x](#area_x) 和 [y](#area_y) 访问器被替换为 [angle](#areaRadial_angle) 和 [radius](#areaRadial_radius) 访问器。径向区域图总是相对于 ⟨0,0⟩。你可以使用坐标变换将其平移到指定的位置。参考 [SVG](http://www.w3.org/TR/SVG/coords.html#TransformAttribute), [Canvas](http://www.w3.org/TR/2dcontext/#transformations) 的坐标变换。

<a name="_areaRadial" href="#_areaRadial">#</a> <i>areaRadial</i>(<i>data</i>)

等价于 [*area*](#_area).

<a name="areaRadial_angle" href="#areaRadial_angle">#</a> <i>areaRadial</i>.<b>angle</b>([<i>angle</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/areaRadial.js#L13 "Source")

等价于 [*area*.x](#area_x), 只不过返回值为弧度，其中 `0` 度位于 -*y* (12 点钟方向).

<a name="areaRadial_startAngle" href="#areaRadial_startAngle">#</a> <i>areaRadial</i>.<b>startAngle</b>([<i>angle</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/areaRadial.js#L14 "Source")

等价于 [*area*.x0](#area_x0), 只不过访问器返回弧度值，其中`0` 度位于 -*y* (12 点钟方向). 注意：通常使用角度，而不是设置单独的起始和结束角。

<a name="areaRadial_endAngle" href="#areaRadial_endAngle">#</a> <i>areaRadial</i>.<b>endAngle</b>([<i>angle</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/areaRadial.js#L15 "Source")

等价于 [*area*.x1](#area_x1), 只不过访问器返回弧度值，其中`0` 度位于 -*y* (12 点钟方向). 注意：通常使用角度，而不是设置单独的起始和结束角。

<a name="areaRadial_radius" href="#areaRadial_radius">#</a> <i>areaRadial</i>.<b>radius</b>([<i>radius</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/areaRadial.js#L16 "Source")

等价于 [*area*.y](#area_y), 只不过访问器返回的是距离 ⟨0,0⟩ 的距离。

<a name="areaRadial_innerRadius" href="#areaRadial_innerRadius">#</a> <i>areaRadial</i>.<b>innerRadius</b>([<i>radius</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/areaRadial.js#L17 "Source")

等价于 [*area*.y0](#area_y0), 只不过访问器返回的是距离 ⟨0,0⟩ 的距离。

<a name="areaRadial_outerRadius" href="#areaRadial_outerRadius">#</a> <i>areaRadial</i>.<b>outerRadius</b>([<i>radius</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/areaRadial.js#L18 "Source")

等价于 [*area*.y1](#area_y1), 只不过访问器返回的是距离 ⟨0,0⟩ 的距离。

<a name="areaRadial_defined" href="#areaRadial_defined">#</a> <i>areaRadial</i>.<b>defined</b>([<i>defined</i>])

等价于 [*area*.defined](#area_defined).

<a name="areaRadial_curve" href="#areaRadial_curve">#</a> <i>areaRadial</i>.<b>curve</b>([<i>curve</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/areaRadial.js#L24 "Source")

等价于 [*area*.curve](#area_curve). 注意不推荐使用 [curveMonotoneX](#curveMonotoneX) 或 [curveMonotoneY](#curveMonotoneY) 因为这两种曲线假设 *x* 或 *y* 维度是单调的, 不适合用于径向区域图.

<a name="areaRadial_context" href="#areaRadial_context">#</a> <i>areaRadial</i>.<b>context</b>([<i>context</i>])

等价于 [*line*.context](#line_context).

<a name="areaRadial_lineStartAngle" href="#areaRadial_lineStartAngle">#</a> <i>areaRadial</i>.<b>lineStartAngle</b>() [<源码>](https://github.com/d3/d3-shape/blob/master/src/areaRadial.js#L19 "Source")
<br><a name="areaRadial_lineInnerRadius" href="#areaRadial_lineInnerRadius">#</a> <i>areaRadial</i>.<b>lineInnerRadius</b>() [<源码>](https://github.com/d3/d3-shape/blob/master/src/areaRadial.js#L21 "Source")

返回一个新的 [radial line generator](#lineRadial)，内置当前径向区域图的 [defined accessor](#areaRadial_defined), [curve](#areaRadial_curve) 和 [context](#areaRadial_context)。线条的 [angle accessor](#lineRadial_angle) 为区域图的 [start angle accessor](#areaRadial_startAngle), 线条的 [radius accessor](#lineRadial_radius) 为区域图的 [inner radius accessor](#areaRadial_innerRadius).

<a name="areaRadial_lineEndAngle" href="#areaRadial_lineEndAngle">#</a> <i>areaRadial</i>.<b>lineEndAngle</b>() [<源码>](https://github.com/d3/d3-shape/blob/master/src/areaRadial.js#L20 "Source")

返回一个新的 [radial line generator](#lineRadial)，内置区域图的 [defined accessor](#areaRadial_defined), [curve](#areaRadial_curve) 和 [context](#areaRadial_context)。线条的 [angle accessor](#lineRadial_angle) 为区域图的 [end angle accessor](#areaRadial_endAngle), 线条的 [radius accessor](#lineRadial_radius) 为区域图的 [inner radius accessor](#areaRadial_innerRadius).

<a name="areaRadial_lineOuterRadius" href="#areaRadial_lineOuterRadius">#</a> <i>areaRadial</i>.<b>lineOuterRadius</b>() [<源码>](https://github.com/d3/d3-shape/blob/master/src/areaRadial.js#L22 "Source")

返回一个新的 [radial line generator](#lineRadial)，内置区域图的 [defined accessor](#areaRadial_defined), [curve](#areaRadial_curve) 和 [context](#areaRadial_context). 线条的 [angle accessor](#lineRadial_angle) 为区域图的 [start angle accessor](#areaRadial_startAngle), 线条的 [radius accessor](#lineRadial_radius) 为区域图的 [outer radius accessor](#areaRadial_outerRadius).

### Curves

[lines](#lines) 被定义为一系列二维点 [*x*, *y*]，[areas](#areas) 类似的可以由顶线和基线定义，但是还有一个任务就是把这些离散的点转换为连续的线条: 例如如何在这些点之间进行插值，插值的方式有很多种。

插值曲线通常不会直接使用，而是传递给 [*line*.curve](#line_curve) 和 [*area*.curve](#area_curve)。例如:

```js
var line = d3.line()
    .x(function(d) { return x(d.date); })
    .y(function(d) { return y(d.value); })
    .curve(d3.curveCatmullRom.alpha(0.5));
```

<a name="curveBasis" href="#curveBasis">#</a> d3.<b>curveBasis</b>(<i>context</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/basis.js#L12 "Source")

<img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/basis.png" width="888" height="240" alt="basis">

使用指定的控制点生成一个三次 [basis spline(样条曲线)](https://en.wikipedia.org/wiki/B-spline)。第一个和最后一个点会被分成三个重复的点，这样就能保证线条经过第一个和最后一个点。并且曲线与第一个和第二个点之间的连线相切，同时与最后一个与倒数第二个点连线相切。

<a name="curveBasisClosed" href="#curveBasisClosed">#</a> d3.<b>curveBasisClosed</b>(<i>context</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/basisClosed.js "Source")

<img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/basisClosed.png" width="888" height="240" alt="basisClosed">

使用指定的控制点生成一个闭合的三次 [basis spline](https://en.wikipedia.org/wiki/B-spline)。当一个线段结束时，前三个控制点被重复，产生一个连续性的闭环。

<a name="curveBasisOpen" href="#curveBasisOpen">#</a> d3.<b>curveBasisOpen</b>(<i>context</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/basisOpen.js "Source")

<img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/basisOpen.png" width="888" height="240" alt="basisOpen">

使用指定的控制点生成一个三次 [basis spline](https://en.wikipedia.org/wiki/B-spline)。与 [basis](#basis) 不同，第一个和最后一个控制点不会被重复，这条曲线通常不会与这些点相交。

<a name="curveBundle" href="#curveBundle">#</a> d3.<b>curveBundle</b>(<i>context</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/bundle.js "Source")

<img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/bundle.png" width="888" height="240" alt="bundle">

使用指定的控制点产生一个可以校正调整的三次 [basis spline](https://en.wikipedia.org/wiki/B-spline)，校正系数根据曲线的 [*beta*](#curveBundle_beta) 系数确定，默认为 `0.85`。这种曲线通常用在 [hierarchical edge bundling](http://bl.ocks.org/mbostock/7607999) 中来消除视觉混淆。这个算法是 [Danny Holten](https://www.win.tue.nl/vis1/home/dholten/) 在 [Hierarchical Edge Bundles: Visualization of Adjacency Relations in Hierarchical Data](https://www.win.tue.nl/vis1/home/dholten/papers/bundles_infovis.pdf) 中提出的。这种曲线生成方式不会实现 [*curve*.areaStart](#curve_areaStart) 和 [*curve*.areaEnd](#curve_areaEnd)，它的设计是用来和 [d3.line](#lines) 结合的，而不是 [d3.area](#areas)。

<a name="curveBundle_beta" href="#curveBundle_beta">#</a> <i>bundle</i>.<b>beta</b>(<i>beta</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/bundle.js#L51 "Source")

根据指定的 *beta* 设置曲线的校正系数，系数范围为 `[0, 1]` 用来表示绑定强度。如果 *beta* 为 `0` 则会在第一个和最后一个点之间生成一个直线，如果 *beta* 为 `1`，则会生成一个标准的 [basis](#basis)。例如:

```js
var line = d3.line().curve(d3.curveBundle.beta(0.5));
```

<a name="curveCardinal" href="#curveCardinal">#</a> d3.<b>curveCardinal</b>(<i>context</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/cardinal.js "Source")

<img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/cardinal.png" width="888" height="240" alt="cardinal">

使用指定的控制点生成一条三次 [cardinal spline](https://en.wikipedia.org/wiki/Cubic_Hermite_spline#Cardinal_spline) 曲线，默认 [tension](#curveCardinal_tension) 为 `0`.

<a name="curveCardinalClosed" href="#curveCardinalClosed">#</a> d3.<b>curveCardinalClosed</b>(<i>context</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/cardinalClosed.js "Source")

<img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/cardinalClosed.png" width="888" height="240" alt="cardinalClosed">

使用指定的控制点生成一条闭合的三次 [cardinal spline](https://en.wikipedia.org/wiki/Cubic_Hermite_spline#Cardinal_spline) 曲线。默认 [tension](#curveCardinal_tension) 为 `0`.

<a name="curveCardinalOpen" href="#curveCardinalOpen">#</a> d3.<b>curveCardinalOpen</b>(<i>context</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/cardinalOpen.js "Source")

<img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/cardinalOpen.png" width="888" height="240" alt="cardinalOpen">

使用指定的控制点生成一条三次 [cardinal spline](https://en.wikipedia.org/wiki/Cubic_Hermite_spline#Cardinal_spline) 曲线，与 [curveCardinal](#curveCardinal) 不同，生成的曲线不利用第一个和最后一个点。默认 [tension](#curveCardinal_tension) 为 `0`.

<a name="curveCardinal_tension" href="#curveCardinal_tension">#</a> <i>cardinal</i>.<b>tension</b>(<i>tension</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/cardinalOpen.js#L44 "Source")

使用指定的处于 [0, 1] 的 *tension* 系数设置曲线的张力，张力确定了切线的长度：张力为 `1` 等价于 [curveLinear](#curveLinear)，张力为 `0` 等价于 [Catmull–Rom](#curveCatmullRom)。例如：

```js
var line = d3.line().curve(d3.curveCardinal.tension(0.5));
```

<a name="curveCatmullRom" href="#curveCatmullRom">#</a> d3.<b>curveCatmullRom</b>(<i>context</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/catmullRom.js "Source")

<img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/catmullRom.png" width="888" height="240" alt="catmullRom">

使用指定的控制点和默认值为 `0.5` 的 [*alpha*](#catmullRom_alpha) 值生成一条 `Catmull–Rom` 曲线。曲线的详细介绍参考 [On the Parameterization of Catmull–Rom Curves](http://www.cemyuksel.com/research/catmullrom_param/)。

<a name="curveCatmullRomClosed" href="#curveCatmullRomClosed">#</a> d3.<b>curveCatmullRomClosed</b>(<i>context</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/catmullRomClosed.js "Source")

<img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/catmullRomClosed.png" width="888" height="330" alt="catmullRomClosed">

使用指定的控制点和默认值为 `0.5` 的 [*alpha*](#catmullRom_alpha) 值生成一条闭合的 `Catmull–Rom` 曲线。

<a name="curveCatmullRomOpen" href="#curveCatmullRomOpen">#</a> d3.<b>curveCatmullRomOpen</b>(<i>context</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/catmullRomOpen.js "Source")

<img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/catmullRomOpen.png" width="888" height="240" alt="catmullRomOpen">

使用指定的控制点和默认值为 `0.5` 的 [*alpha*](#catmullRom_alpha) 值生成一条 `Catmull–Rom` 曲线。与 [curveCatmullRom](#curveCatmullRom) 不同的是所生成的曲线不经过第一个和最后一个控制点。

<a name="curveCatmullRom_alpha" href="#curveCatmullRom_alpha">#</a> <i>catmullRom</i>.<b>alpha</b>(<i>alpha</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/catmullRom.js#L83 "Source")

使用指定的 *alpha* 值(`[0, 1]`) 返回一条 `Catmull–Rom` 生成器。如果 *alpha* 为 `0` 则等价于 [curveCardinal](#curveCardinal)，如果 *alpha* 为 `1` 则会生成 `chordal` 曲线，如果 *alpha* 为 `0.5` 则会生成 [centripetal spline](https://en.wikipedia.org/wiki/Centripetal_Catmull–Rom_spline)。例如:

```js
var line = d3.line().curve(d3.curveCatmullRom.alpha(0.5));
```

<a name="curveLinear" href="#curveLinear">#</a> d3.<b>curveLinear</b>(<i>context</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/linear.js "Source")

<img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/linear.png" width="888" height="240" alt="linear">

通过指定的点产生折线。

<a name="curveLinearClosed" href="#curveLinearClosed">#</a> d3.<b>curveLinearClosed</b>(<i>context</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/linearClosed.js "Source")

<img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/linearClosed.png" width="888" height="240" alt="linearClosed">

产生闭合折线

<a name="curveMonotoneX" href="#curveMonotoneX">#</a> d3.<b>curveMonotoneX</b>(<i>context</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/monotone.js#L98 "Source")

<img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/monotoneX.png" width="888" height="240" alt="monotoneX">

产生一条在 `y` 方向保持单调性的曲线，假设在 `x` 方向是单调的。曲线具体描述：[A simple method for monotonic interpolation in one dimension](http://adsabs.harvard.edu/full/1990A%26A...239..443S)。

<a name="curveMonotoneY" href="#curveMonotoneY">#</a> d3.<b>curveMonotoneY</b>(<i>context</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/monotone.js#L102 "Source")

<img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/monotoneY.png" width="888" height="240" alt="monotoneY">

产生一条在 `x` 方向保持单调性的曲线，假设在 `y` 方向是单调的。曲线具体描述：[A simple method for monotonic interpolation in one dimension](http://adsabs.harvard.edu/full/1990A%26A...239..443S)。

<a name="curveNatural" href="#curveNatural">#</a> d3.<b>curveNatural</b>(<i>context</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/natural.js "Source")

<img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/natural.png" width="888" height="240" alt="natural">

产生一条 [自然的](https://en.wikipedia.org/wiki/Spline_interpolation) 的 [三次样条曲线](http://mathworld.wolfram.com/CubicSpline.html)，其二阶导数在端点设为零。

<a name="curveStep" href="#curveStep">#</a> d3.<b>curveStep</b>(<i>context</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/step.js "Source")

<img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/step.png" width="888" height="240" alt="step">

产生一个分段常数函数 ([阶梯函数](https://en.wikipedia.org/wiki/Step_function))，由水平和垂直的交替线组成。`y` 值在每一对相邻 `x` 值的中点处发生变化。

<a name="curveStepAfter" href="#curveStepAfter">#</a> d3.<b>curveStepAfter</b>(<i>context</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/step.js#L51 "Source")

<img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/stepAfter.png" width="888" height="240" alt="stepAfter">

产生一个分段常数函数 ([阶梯函数](https://en.wikipedia.org/wiki/Step_function))，由水平和垂直的交替线组成。`y` 值在 `x` 值之后发生变化。

<a name="curveStepBefore" href="#curveStepBefore">#</a> d3.<b>curveStepBefore</b>(<i>context</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/step.js#L47 "Source")

<img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/stepBefore.png" width="888" height="240" alt="stepBefore">

产生一个分段常数函数 ([阶梯函数](https://en.wikipedia.org/wiki/Step_function))，由水平和垂直的交替线组成。`y` 值在 `x` 值之前发生变化。

### Custom Curves

`Curves` 通常不会直接使用，而是传递给 [*line*.curve](#line_curve) 和 [*area*.curve](#area_curve). 但是，如果你对内置的曲线不满意的话可以使用如下接口定义自己的曲线实现来替代内置的曲线。你也可以使用这个带有内置曲线类型的低级接口作为线和面积生成器的替代选择。

<a name="curve_areaStart" href="#curve_areaStart">#</a> <i>curve</i>.<b>areaStart</b>() [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/step.js#L7 "Source")

指明新区域段的开始。每个区域段恰好由两个 [line segments](#curve_lineStart) 组成 : 顶线，基线点以相反的顺序排列的基线。

<a name="curve_areaEnd" href="#curve_areaEnd">#</a> <i>curve</i>.<b>areaEnd</b>() [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/step.js#L10 "Source")

指明当前区域段的结束。

<a name="curve_lineStart" href="#curve_lineStart">#</a> <i>curve</i>.<b>lineStart</b>() [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/step.js#L13 "Source")

指明新线段的开始。后面可以跟着零个或多个 [points](#curve_point)。

<a name="curve_lineEnd" href="#curve_lineEnd">#</a> <i>curve</i>.<b>lineEnd</b>() [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/step.js#L17 "Source")

指明当前线段的结束。

<a name="curve_point" href="#curve_point">#</a> <i>curve</i>.<b>point</b>(<i>x</i>, <i>y</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/curve/step.js#L22 "Source")

指明当前线段中具有给定 `x` 和 `y` 值的新点。

### Links

[<img alt="Tidy Tree" src="https://raw.githubusercontent.com/d3/d3-hierarchy/master/img/tree.png">](http://bl.ocks.org/mbostock/9d0899acb5d3b8d839d9d613a9e1fe04)

`link` 用来生成从一个源点到目标点的光滑的三次贝塞尔曲线。曲线在起点和终点的切线要么是 [vertical](#linkVertical)，要么是 [horizontal](#linkHorizontal)，要么是 [radial](#linkRadial)的。

<a name="linkVertical" href="#linkVertical">#</a> d3.<b>linkVertical</b>() [<源码>](https://github.com/d3/d3-shape/blob/master/src/link/index.js#L74 "Source")

返回一个新的 [link 生成器](#_link)，生成的曲线在曲线的终点和起点处的切线是垂直方向的。例如在 [tree diagram](https://github.com/xswei/d3-hierarchy/blob/master/README.md#tree) 中对 
[links](https://github.com/xswei/d3-hierarchy/blob/master/README.md#node_links) 进行可视化时，可以定义为：

```js
var link = d3.linkVertical()
    .x(function(d) { return d.x; })
    .y(function(d) { return d.y; });
```

<a name="linkHorizontal" href="#linkHorizontal">#</a> d3.<b>linkHorizontal</b>() [<源码>](https://github.com/d3/d3-shape/blob/master/src/link/index.js#L70 "Source")

返回一个新的 [link 生成器](#_link)，生成的曲线在曲线的终点和起点处的切线是水平方向的。例如在 [tree diagram](https://github.com/xswei/d3-hierarchy/blob/master/README.md#tree) 中对 
[links](https://github.com/xswei/d3-hierarchy/blob/master/README.md#node_links) 进行可视化时，可以定义为：

```js
var link = d3.linkHorizontal()
    .x(function(d) { return d.y; })
    .y(function(d) { return d.x; });
```

<a href="#_link" name="_link">#</a> <i>link</i>(<i>arguments…</i>) [<源码>](https://github.com/d3/d3-shape/blob/master/src/link/index.js#L21 "Source")

根据指定的 *arguments* 生成 `links`。*arguments* 是任意的。它们会被直接传递给 `link` 生成器的访问函数。例如，使用的默认设置时，期望的参数为:

```js
link({
  source: [100, 100],
  target: [300, 300]
});
```

<a name="link_source" href="#link_source">#</a> <i>link</i>.<b>source</b>([<i>source</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/link/index.js#L28 "Source")

如果指定了 *source* 则将 `source` 访问器设置为指定的函数并返回当前 `link` 生成器。如果没有指定 *source* 则返回当前的 `source` 访问器，默认为:

```js
function source(d) {
  return d.source;
}
```

<a name="link_target" href="#link_target">#</a> <i>link</i>.<b>target</b>([<i>target</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/link/index.js#L32 "Source")

如果指定了 *target* 则将 `target` 访问器设置为指定的函数并返回当前 `link` 生成器。如果没有指定 *target* 则返回当前的 `target` 访问器，默认为:

```js
function target(d) {
  return d.target;
}
```

<a name="link_x" href="#link_x">#</a> <i>link</i>.<b>x</b>([<i>x</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/link/index.js#L36 "Source")

如果指定了 *x* 则将 *x* 访问器设置为指定的函数或数值，并返回当前 `link` 生成器。如果 *x* 没有指定则返回当前 *x* 访问器，默认为: 

```js
function x(d) {
  return d[0];
}
```

<a name="link_y" href="#link_y">#</a> <i>link</i>.<b>y</b>([<i>y</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/link/index.js#L40 "Source")

如果指定了 *y* 则将 *y* 访问器设置为指定的函数或数值，并返回当前 `link` 生成器。如果 *y* 没有指定则返回当前 *y* 访问器，默认为: 

```js
function y(d) {
  return d[1];
}
```

<a name="link_context" href="#link_context">#</a> <i>link</i>.<b>context</b>([<i>context</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/link/index.js#L44 "Source")

如果指定了 *context*，则设置上下文并返回当前 `link` 生成器。如果没有指定 *context* 则返回当前的上下文，默认为 `null`。如果上下文非空，则 [生成的 link](#_link) 会被渲染到指定的上下文中。否则会返回 [path data](http://www.w3.org/TR/SVG/paths.html#PathData)字符串。参考 [d3-path](https://github.com/xswei/d3-path).

<a name="linkRadial" href="#linkRadial">#</a> d3.<b>linkRadial</b>() [<源码>](https://github.com/d3/d3-shape/blob/master/src/link/index.js#L78 "Source")

返回一个新的径向 [link 生成器](#_link)。例如在 [tree diagram](https://github.com/xswei/d3-hierarchy/blob/master/README.md#tree) 中对 
[links](https://github.com/xswei/d3-hierarchy/blob/master/README.md#node_links) 进行可视化时，可以定义为：

```js
var link = d3.linkRadial()
    .angle(function(d) { return d.x; })
    .radius(function(d) { return d.y; });
```

<a name="linkRadial_angle" href="#linkRadial_angle">#</a> <i>linkRadial</i>.<b>angle</b>([<i>angle</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/link/index.js#L80 "Source")

等价于 [*link*.x](#link_x), 只不过访问器返回的是弧度值，其中 `0` 度为 `12` 点钟方向。

<a name="linkRadial_radius" href="#linkRadial_radius">#</a> <i>linkRadial</i>.<b>radius</b>([<i>radius</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/link/index.js#L81 "Source")

等价于 [*link*.y](#link_y), 只不过访问器返回的是半径: 到 `⟨0,0⟩` 的距离。

### Symbols

<a href="#symbolCircle"><img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/circle.png" width="100" height="100"></a><a href="#symbolCross"><img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/cross.png" width="100" height="100"></a><a href="#symbolDiamond"><img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/diamond.png" width="100" height="100"></a><a href="#symbolSquare"><img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/square.png" width="100" height="100"></a><a href="#symbolStar"><img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/star.png" width="100" height="100"></a><a href="#symbolTriangle"><img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/triangle.png" width="100" height="100"></a><a href="#symbolWye"><img src="https://raw.githubusercontent.com/d3/d3-shape/master/img/wye.png" width="100" height="100"></a>

符号提供了几种用来表示分类的形状。符号的坐标总是位于 `⟨0,0⟩`, 需要使用 `transform` 将其移动到指定的位置(参考: [SVG](http://www.w3.org/TR/SVG/coords.html#TransformAttribute), [Canvas](http://www.w3.org/TR/2dcontext/#transformations))。

<a name="symbol" href="#symbol">#</a> d3.<b>symbol</b>() [<源码>](https://github.com/d3/d3-shape/blob/master/src/symbol.js "Source")

使用默认的设置构造一个新的符号生成器。

<a name="_symbol" href="#_symbol">#</a> <i>symbol</i>(<i>arguments</i>…) [<源码>](https://github.com/d3/d3-shape/blob/master/src/symbol.js#L11 "Source")

使用指定的 *arguments* 生成一个符号。*arguments* 是任意的。它们会被直接传递给符号生成器的访问器。例如使用默认的设置，没有任何参数的情况下生成的是一个 `64` 平方像素的区域。如果符号生成器拥有 [context](#symbol_context) 则符号会被渲染到此上下文中。否则会返回一个 [path data](http://www.w3.org/TR/SVG/paths.html#PathData) 字符串。

<a name="symbol_type" href="#symbol_type">#</a> <i>symbol</i>.<b>type</b>([<i>type</i>]) [<源码>](https://github.com/d3/d3-shape/blob/master/src/symbol.js#L33 "Source")

如果指定了 *type* 则将符号的类型设置为指定的函数或符号类型并返回符号生成器。如果没有指定 *type* 则返回当前的类型访问器，默认为：

```js
function type() {
  return circle;
}
```
