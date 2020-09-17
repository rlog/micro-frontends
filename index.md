使多个团队可以使用不同的技术栈来开发现代web应用

## 什么是微前端

`微前端`这个词最早出现在2016年底的 [ThoughtWorks Technology Radar](https://www.thoughtworks.com/radar/techniques/micro-frontends) 上。它把微服务的概念扩展到前端领域。当前的趋势是构建功能丰富、强大的浏览器应用，又称单页应用，它位于微服务架构之上。前端层通常由独立的团队开发，随着时间的推移，前端层会不断增长，并且越来越难以维护。这就是我们所说的前端巨石 [Frontend Monolith](https://www.youtube.com/watch?v=pU1gXA0rfwc)。

微前端背后的理念是，把一个网站或web应用看作是由不同独立团队所开发的功能组合。每个团队都有自己关心并专门从事的业务或领域。单一团队是跨职能的端到端开发，从数据库到用户界面。

这并不是一个全新的架构。它与 [Self-contained Systems](https://scs-architecture.org/) 有许多共同之处。过去这种方法被称为 垂直系统前端集成 [Frontend Integration for Verticalised Systems](https://dev.otto.de/2014/07/29/scaling-with-microservices-and-vertical-decomposition/)。但微前端显然是一个更好的名字。

**Monolithic Frontends**
![Monolithic Frontends](https://micro-frontends.org/ressources/diagrams/organisational/monolith-frontback-microservices.png)

**Organisation in Verticals**
![Organisation in Verticals](https://micro-frontends.org/ressources/diagrams/organisational/verticals-headline.png)

## 什么是 Modern Web App？

在介绍中，我使用了“构建一个现代web应用程序”这个说法。让我们定义与这一项相关的假设。

为了从更广泛的角度来看待这个问题，[Aral Balkan](https://ar.al/) 写了一篇博客文章，内容是关于他所谓的 [Documents‐to‐Applications Continuum](https://ar.al/notes/the-documents-to-applications-continuum/)。他提出了一个滑动比例的概念，即一个由静态文档构建、通过链接连接的站点位于左侧，而一个纯行为驱动、无内容的应用程序(如在线照片编辑器)位于右侧。

![The Documents‐to‐Applications Continuum](https://ar.al/notes/the-documents-to-applications-continuum/images/continuum.svg)

如果将你的的项目放在这个范围的左边，那么web服务器级别的集成是一个很好的选择。通过这个模型，服务器可以从组成用户请求的页面的所有组件中收集和连接HTML字符串。更新是通过从服务器重新加载页面或通过ajax替换部分页面来完成的。[Gustaf Nilsson Kotte](https://twitter.com/gustaf_nk/) 就这个话题写了一篇[综合性的文章](https://gustafnk.github.io/microservice-websites/)。

当你的UI必须提供即时反馈时，即使是在不可靠的连接上，纯服务端渲染的站点也不够用了。要实现乐观UI ([Optimistic User Interfaces](https://www.smashingmagazine.com/2016/11/true-lies-of-optimistic-user-interfaces/)) 或骨架屏 ([Skeleton Screens](http://www.lukew.com/ff/entry.asp?1797)) 这样的技术，你还需要能够更新设备本身的UI。谷歌的渐进Web应用程序([Progressive Web Apps](https://developers.google.com/web/progressive-web-apps/))正好描述了如何平衡此类应用的使用体验和性能。这种应用程序位于站点-应用程序-连续体的中间位置。在这里，一个单独的基于服务器的解决方案已经不够了。我们必须将集成移到浏览器中，这是本文的重点。

## 微前端的核心理念

* **技术无关性**
  每个团队应该能独立选择和升级他们的技术栈，而无需与其他团队协调。[Custom Elements](https://micro-frontends.org/#the-dom-is-the-api) 是隐藏实现细节的好方法，同时也为外部提供了一个中立的接口。

* **团队代码隔离**
  不共享运行时，即使所有团队使用相同的框架。构建独立的、自包含的应用程序。不要依赖于共享状态或全局变量。

* **建立团队前缀**
  在还不可能实现隔离的情况下，建立团队前缀就命名约定达成一致。给 CSS，事件，本地存储和cookie 提供独立的命名空间，以避免冲突和明确所有权。

* **尽量使用浏览器原生能力，而不是自定义API**
  使用浏览器事件进行通信，而不是构建一个全局的 PubSub 系统。如果你真的需要建立一个跨团队的 API，试着让它尽可能的简单。

* **构建一个有弹性的网站**
  即使 JS 失败或尚未执行，也要尽量保证应用可用。使用通用渲染 ([Universal Rendering](https://micro-frontends.org/#serverside-rendering--universal-rendering)) 和渐进增强来提高感知性能。

## DOM 即 API

自定义元素 ([Custom Elements](https://developers.google.com/web/fundamentals/getting-started/primers/customelements)) 是集成到浏览器中的一个很好的原语。每个团队使用自己选择的web技术来构建组件，并将其包装在一个定制元素中(例如 `<order-minicart></order-minicart>` )。这个特定元素(标签、属性和事件)的DOM规范充当其他团队的契约或公共API。其优点是他们可以使用组件及其功能，而不需要知道实现，只与DOM交互。

但是，定制元素本身并不能解决我们所有的需求。为了实现渐进式增强、通用渲染或路由，我们需要额外的软件。

本页主要分为两个部分。首先，我们将讨论页面组合——如何用不同团队开发的组件组装页面。之后，我们将展示实现客户端页面转场的示例。

## 页面组成

除去客户端和服务端集成的代码写在不同的框架本身，还有很多方面应该讨论：js 隔离机制,避免css冲突，按需加载，团队间资源共享，处理数据请求，加载状态。我们将一步一步地讨论这些主题。

### 基础原型

这个拖拉机模型商店的产品页面将作为下面示例的基础。

它的功能是点击三种不同的拖拉机模型缩略图时，会改变产品图片，名称，价格和建议。还有一个购买按钮，它将选中的商品添加到购物篮中，顶部还有一个相应更新的迷你购物车。

![The Base Prototype](https://micro-frontends.org/ressources/video/model-store-0.gif)

[在浏览器里试试](https://micro-frontends.org/0-model-store/) & [看看代码](https://github.com/neuland/micro-frontends/tree/master/0-model-store)

所有的HTML都是在客户端使用无依赖的普通JavaScript和ES6模板字符串生成的。代码使用简单的状态/标记分离，并在每次更改时重新呈现整个HTML客户端，目前没有花哨的 DOM diff 和服务端渲染。也没有团队分离-代码是写在一个js/css文件。

### 客户端集成

在这个例子中，页面被分割成由三个团队拥有的独立组件/片段。结算团队(蓝色)现在负责所有与购买过程有关的事情-即购买按钮和迷你购物车。推荐团队(绿色)管理这个页面上的产品推荐。页面本身属于产品团队(红色)。

![](https://micro-frontends.org/ressources/screen/three-teams.png)

[在浏览器里试试](https://micro-frontends.org/1-composition-client-only/) & [看看代码](https://github.com/neuland/micro-frontends/tree/master/1-composition-client-only)

产品团队决定包含哪些功能以及它在布局中的位置。该页面包含产品团队本身提供的信息，如产品名称、图像和可用的变量。但是它也包括来自其他团队的片段(定制元素)。

### 如何创建一个自定义元素?

让我们以购买按钮为例。产品团队只需添加 `<blue-buy sku="t_porsche"></blue-buy>` 到标记中的所需位置。要使其工作，结算团队必须在页面上注册 `blue-buy` 元素。

```js
class BlueBuy extends HTMLElement {
  connectedCallback() {
    this.innerHTML = `<button type="button">buy for 66,00 €</button>`;
  }

  disconnectedCallback() { ... }
}
window.customElements.define('blue-buy', BlueBuy);
```

现在，每当浏览器遇到新的 `blue-buy` 标记时，就会调用 `connectedCallback`。这是对自定义元素的根DOM节点的引用。标准DOM元素的所有属性和方法(如innerHTML或getAttribute())都可以使用。

![custom-element](https://micro-frontends.org/ressources/video/custom-element.gif)

在命名元素时，规范定义的唯一要求是名称必须包含破折号(-)，以保持与即将到来的新HTML标记的兼容性。在下面的示例中使用命名约定[team_color]-[feature]。team名称空间可以防止冲突，通过这种方式，只需查看DOM，特性的所有权就变得显而易见了。

### 父-子 通讯 / DOM 修改

当用户在变量选择器中选择另一个拖拉机时，buy按钮必须相应地更新。要实现这个功能，产品团队只需从DOM中删除现有元素并插入一个新元素。

```js
container.innerHTML;
// => <blue-buy sku="t_porsche">...</blue-buy>
container.innerHTML = '<blue-buy sku="t_fendt"></blue-buy>';
```

旧元素的 `disconnectedCallback` 被同步调用，为元素提供清理事件侦听器之类东西的机会。然后调用新创建的 t_fendt 元素的 `connectedCallback`。

另一个更具性能的选项是只更新现有元素上的sku属性。

```js
document.querySelector('blue-buy').setAttribute('sku', 't_fendt');
```

如果产品团队使用具有 DOM diffing 特性的模板引擎，比如 React，那么这将由算法自动完成。

![custom-element-attribute](https://micro-frontends.org/ressources/video/custom-element-attribute.gif)

为了支持这一点，自定义元素可以实现 `attributeChangedCallback`，并为该回调指定一个 `observedAttributes `列表:

```js
const prices = {
  t_porsche: '66,00 €',
  t_fendt: '54,00 €',
  t_eicher: '58,00 €',
};

class BlueBuy extends HTMLElement {
  static get observedAttributes() {
    return ['sku'];
  }
  connectedCallback() {
    this.render();
  }
  render() {
    const sku = this.getAttribute('sku');
    const price = prices[sku];
    this.innerHTML = `<button type="button">buy for ${price}</button>`;
  }
  attributeChangedCallback(attr, oldValue, newValue) {
    this.render();
  }
  disconnectedCallback() {...}
}
window.customElements.define('blue-buy', BlueBuy);
```

为了避免重复，引入了一个 `render()` 方法，该方法从 `connectedCallback` 和 `attributeChangedCallback` 调用。这个方法收集需要的数据，innerHTML的新标记。当决定在定制元素中使用更复杂的模板引擎或框架时，这就是初始化代码的位置。

### 浏览器支持
上面的例子使用了 Chrome、Safari 和 Opera 支持的自定义元素 V1 Spec。但是对于 document-register-element，可以使用 polyfill 兼容所有浏览器。它使用了广泛支持的Mutation Observer API，因此在后台没有进行任何复杂的 DOM 树监听。

### 框架的兼容性
因为自定义元素是一种web标准，所有主流 JavaScript 框架，如 Angular、React、Preact、Vue 或 Hyperapp 都支持它们。但是当您深入到细节时，在某些框架中仍然存在一些实现问题。[Custom Elements Everywhere](https://custom-elements-everywhere.com/), Rob Dodson 整理了一个兼容性测试报告，强调了未解决的问题。

### 子-父或兄弟通信 / DOM事件
但只是向下传递属性并不足以满足所有交互。在我们的示例中，当用户单击 buy 按钮时，迷你购物车应该刷新。

这两个片段都属于结算团队(蓝色)，所以他们可以构建某种内部 JavaScript API，让迷你购物车知道什么时候按钮被按下了。但是这将需要组件实例相互知晓，而这样也会违反隔离策略。

一种更干净的方法是使用 PubSub 机制，其中组件可以发布消息，而其他组件可以订阅特定的主题。所幸浏览器有这个内置的特性。这正是浏览器事件如 click, select 或 mouseover 的工作方式。除了原生事件外，还可以使用新的 `CustomEvent(...)` 创建更高级别的事件。事件总是与创建/分派它们的DOM节点绑定。大多数原生事件还可以冒泡。这使得侦听 DOM 特定子树上的所有事件成为可能。如果您想监听页面上的所有事件，可以将事件监听器附加到 window 元素上。下面是创建 `blue:basket:changed` 事件的示例:

```js
class BlueBuy extends HTMLElement {
  [...]
  connectedCallback() {
    [...]
    this.render();
    this.firstChild.addEventListener('click', this.addToCart);
  }
  addToCart() {
    // maybe talk to an api
    this.dispatchEvent(new CustomEvent('blue:basket:changed', {
      bubbles: true,
    }));
  }
  render() {
    this.innerHTML = `<button type="button">buy</button>`;
  }
  disconnectedCallback() {
    this.firstChild.removeEventListener('click', this.addToCart);
  }
}
```

迷你购物车现在可以订阅 `window` 上的这个事件，并在该刷新数据时得到通知。

```js
class BlueBasket extends HTMLElement {
  connectedCallback() {
    [...]
    window.addEventListener('blue:basket:changed', this.refresh);
  }
  refresh() {
    // fetch new data and render it
  }
  disconnectedCallback() {
    window.removeEventListener('blue:basket:changed', this.refresh);
  }
}
```

使用这种方法，迷你购物车片段向其作用域(`window`)之外的DOM元素添加了一个侦听器。这对于许多应用程序来说应该没问题，但是如果你不喜欢这样做，你还可以实现一种方法，即页面本身(产品团队)侦听事件，并通过调用DOM元素上的 `refresh()` 来通知迷你购物车。

```js
// page.js
const $ = document.getElementsByTagName;

$('blue-buy')[0].addEventListener('blue:basket:changed', function() {
  $('blue-basket')[0].refresh();
});
```

命令式调用DOM方法比较少见，可以在 (video element api)[http://webtrans.yodao.com/server/webtrans/tranUrl?url=https%3A%2F%2Fdeveloper.mozilla.org%2Fde%2Fdocs%2FWeb%2FHTML%2FUsing_HTML5_audio_and_video%23Controlling_media_playback&from=en&to=zh-CHS&type=2&product=mdictweb&salt=1599568319121&sign=3b85870769ece44a0171314edc53c532] 中找到。如果可能，尽量使用声明式方法(修改属性)。

## 服务器端渲染/通用渲染

自定义元素非常适合在浏览器中作为组件集成。但是，当构建一个web上可访问的站点时，首页加载性能很可能会影响到用户，在所有js框架下载并执行之前，用户会看到一个白屏。此外，最好也考虑一下如果JavaScript失败或被阻止，会发生什么。[Jeremy Keith](https://adactio.com/) 在他的电子书/播客中解释了[弹性网页设计](https://resilientwebdesign.com/)的重要性。因此，在服务器端能渲染核心内容是个关键。遗憾的是web组件规范根本没有提到服务端渲染。没有JavaScript就没法玩自定义元素 :(

### 自定义元素 + 服务端包含 = ❤️

要使服务端渲染工作，对前面的示例需要进行重构。每个团队都有自己的express服务器，定制元素的render()方法也可以通过url访问。

```html
$ curl http://127.0.0.1:3000/blue-buy?sku=t_porsche
<button type="button">buy for 66,00 €</button>
```

自定义元素标签名称用作路径名称—属性成为查询参数。现在有一 种方法可以呈现每个组件的内容。结合 -自定义元素，一些非常接近通用Web组件的东西被实现:

```html
<blue-buy sku="t_porsche">
  <!--#include virtual="/blue-buy?sku=t_porsche" -->
</blue-buy>
```

#include注释是服务器端include的一部分，这是一个在大多数web服务器中可用的特性。是的，这和以前在我们的网站上嵌入当前数据的技术是一样的。也有一些替代技术，如ESI, nodesi, compoxure和tailor，但对于我们的项目，SSI已经证明自己是一个简单和难以置信的稳定解决方案。

#include注释被/blue-buy?的响应取代。在web服务器将完整的页面发送到浏览器之前，sku=t_porsche。nginx的配置是这样的:

```js
upstream team_blue {
  server team_blue:3001;
}
upstream team_green {
  server team_green:3002;
}
upstream team_red {
  server team_red:3003;
}

server {
  listen 3000;
  ssi on;

  location /blue {
    proxy_pass  http://team_blue;
  }
  location /green {
    proxy_pass  http://team_green;
  }
  location /red {
    proxy_pass  http://team_red;
  }
  location / {
    proxy_pass  http://team_red;
  }
}
```

指令ssi: on;启用SSI特性，并为每个团队添加上游和位置块，以确保所有以/blue开头的url将被路由到正确的应用程序(team_blue:3001)。此外，/路线被映射到team red，它控制着主页/产品页面。

这个动画显示了禁用JavaScript的浏览器中的拖拉机存储。

![server-render](https://micro-frontends.org/ressources/video/server-render.gif)

[查看代码](https://github.com/neuland/micro-frontends/tree/master/2-composition-universal)

变体选择按钮现在是实际的链接，每次点击都会导致页面的重新加载。右边的终端说明了如何将一个页面的请求路由到team red的过程，后者控制产品页面，然后用来自team blue和green的片段补充标记。

当重新打开JavaScript时，只会看到第一个请求的服务器日志消息。所有后续的牵引器更改都在客户端处理，就像第一个示例一样。在后面的示例中，产品数据将从JavaScript中提取并根据需要通过REST api加载。

您可以在本地机器上使用此示例代码。只需要安装Docker Compose。

```shell
git clone https://github.com/neuland/micro-frontends.git
cd micro-frontends/2-composition-universal
docker-compose up --build
```

然后Docker在端口3000上启动nginx，并为每个团队构建node.js图像。在浏览器中打开http://127.0.0.1:3000/时，应该会看到一个红色拖拉机。docker-compose的组合日志使查看网络中正在发生的事情变得很容易。遗憾的是，没有办法控制输出的颜色，所以您必须忍受团队蓝色可能用绿色突出显示的事实:)

src文件被映射到各个容器中，当您进行代码更改时，节点应用程序将重新启动。更改nginx.conf需要重新启动docker-compose才能产生效果。所以请随意摆弄并给出反馈。

### 数据获取和加载状态

SSI/ESI方法的缺点是，最慢的片段决定整个页面的响应时间。因此，当片段的响应可以被缓存时，这是很好的。对于那些制作成本昂贵且难以缓存的片段，最好将它们从初始渲染中排除。它们可以在浏览器中异步加载。在我们的示例中，显示个性化推荐的green-recos片段是一个候选选项。

一个可能的解决方案是team red跳过SSI包含。

之前

```html
<green-recos sku="t_porsche">
  <!--#include virtual="/green-recos?sku=t_porsche" -->
</green-recos>
```

之后

```html
<green-recos sku="t_porsche"></green-recos>
```

**重要的附注:定制元素不能自动关闭,所以写 `<green-recos sku="t_porsche" />` 不能正常工作。**

![](https://micro-frontends.org/ressources/video/data-fetching-reflow.gif)

渲染只在浏览器中进行。但是，正如在动画中看到的，这个更改现在已经引入了页面的大量回流。推荐区域最初是空白的。载入和执行绿色团队的JavaScript。调用API来获取个性化推荐。将呈现推荐标记并请求相关的图像。片段现在需要更多的空间并推动页面的布局。

有不同的选择来避免烦人的回流像这样。Team red控制页面，它可以固定推荐容器的高度。在一个响应式网站上，通常很难确定高度，因为不同的屏幕尺寸可能会有不同的高度。但更重要的问题是，这种团队间的协议在团队红与绿之间形成了紧密的耦合。如果team green想在reco元素中引入额外的子标题，那么它必须与team red在新的高度上协调。两个团队必须同时展示他们的改变，以避免破坏布局。

更好的方法是使用一种叫做骨架屏幕的技术。红队留下了标记中包含的绿色recos SSI。此外，team green更改了其片段的服务器端呈现方法，以便生成内容的示意图版本。框架标记可以重用实际内容的部分布局样式。这样就保留了所需的空间，并且实际内容的填充不会导致跳转。

![](https://micro-frontends.org/ressources/video/data-fetching-skeleton.gif)

骨架屏幕对于客户机呈现也非常有用。当您的自定义元素由于用户操作被插入到DOM中时，它可以立即呈现骨架，直到它从服务器获得所需的数据为止。

即使在属性更改(比如变体选择)时，您也可以决定切换到骨架视图，直到新的数据到达。通过这种方式，用户可以得到片段中正在发生的事情的指示。但是，当端点快速响应时，旧数据和新数据之间的骨架闪烁也可能很烦人。保留旧数据或使用智能超时可能会有所帮助。所以要明智地使用这种技术，并努力获得用户的反馈。

## 页面之间导航
TODO