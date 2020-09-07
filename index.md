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

当你的UI必须提供即时反馈时，即使是在不可靠的连接上，纯服务器呈现的站点也不够用了。要实现乐观UI ([Optimistic User Interfaces](https://www.smashingmagazine.com/2016/11/true-lies-of-optimistic-user-interfaces/)) 或骨架屏 ([Skeleton Screens](http://www.lukew.com/ff/entry.asp?1797)) 这样的技术，你还需要能够更新设备本身的UI。谷歌的渐进Web应用程序([Progressive Web Apps](https://developers.google.com/web/progressive-web-apps/))正好描述了如何平衡此类应用的使用体验和性能。这种应用程序位于站点-应用程序-连续体的中间位置。在这里，一个单独的基于服务器的解决方案已经不够了。我们必须将集成移到浏览器中，这是本文的重点。

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

它的功能是点击三种不同的拖拉机模型缩略图时，会改变产品图片，名称，价格和建议。还有一个购买按钮，它将选中的商品添加到购物篮中，顶部还有一个相应更新的迷你购物篮。

![The Base Prototype](https://micro-frontends.org/ressources/video/model-store-0.gif)

[在浏览器里试试](https://micro-frontends.org/0-model-store/) & [看看代码](https://github.com/neuland/micro-frontends/tree/master/0-model-store)

所有的HTML都是在客户端使用无依赖的普通JavaScript和ES6模板字符串生成的。代码使用简单的状态/标记分离，并在每次更改时重新呈现整个HTML客户端，目前没有花哨的 DOM diff 和服务端渲染。也没有团队分离-代码是写在一个js/css文件。

### 客户端集成

在这个例子中，页面被分割成由三个团队拥有的独立组件/片段。结算团队(蓝色)现在负责所有与购买过程有关的事情-即购买按钮和迷你篮子。推荐团队(绿色)管理这个页面上的产品推荐。页面本身属于产品团队(红色)。

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

为了避免重复，引入了一个 `render()` 方法，该方法从 `connectedCallback` 和`attributeChangedCallback` 调用。这个方法收集需要的数据，innerHTML的新标记。当决定在定制元素中使用更复杂的模板引擎或框架时，这就是初始化代码的位置。

- TODO: 

### 浏览器支持
上面的例子使用了 Chrome、Safari 和 Opera 支持的定制元素V1 Spec。但是对于 document-register-element，可以使用经过战斗测试的轻量级填充在所有浏览器中工作。在幕后，它使用了广泛支持的Mutation Observer API，因此在后台没有进行任何复杂的DOM树监视。

### 框架的兼容性
因为定制元素是一种web标准，所有主要的JavaScript框架，如Angular、React、Preact、Vue或Hyperapp都支持它们。但是当您深入到细节时，在某些框架中仍然存在一些实现问题。在Custom Elements Everywhere, Rob Dodson整理了一个兼容性测试套件，强调了未解决的问题。

### 子-父或兄弟通信 / DOM事件
但是向下传递属性并不足以满足所有交互。在我们的示例中，当用户单击buy按钮时，迷你购物篮应该刷新。

这两个片段都属于Team Checkout(蓝色)，所以他们可以构建某种内部JavaScript API，让迷你篮子知道什么时候按钮被按下了。但是这将需要组件实例相互了解，而且也会违反隔离。

一种更干净的方法是使用PubSub机制，其中组件可以发布消息，而其他组件可以订阅特定的主题。幸运的是，浏览器有这个内置的特性。这正是浏览器事件如单击、选择或鼠标悬停的工作方式。除了本机事件之外，还可以使用新的CustomEvent(…)创建更高级别的事件。事件总是与创建/分派它们的DOM节点绑定。大多数本机事件还具有冒泡特性。这使得侦听DOM特定子树上的所有事件成为可能。如果您想监听页面上的所有事件，请将事件监听器附加到window元素。下面是创建蓝色的:basket:changed-event的示例:

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

迷你篮子现在可以订阅窗口上的这个事件，并在应该刷新数据时得到通知。

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

使用这种方法，迷你篮子片段向其作用域(窗口)之外的DOM元素添加了一个侦听器。这对于许多应用程序来说应该没问题，但是如果您不喜欢这样做，您还可以实现一种方法，即页面本身(Team Product)侦听事件，并通过调用DOM元素上的refresh()来通知迷你篮子。

```js
// page.js
const $ = document.getElementsByTagName;

$('blue-buy')[0].addEventListener('blue:basket:changed', function() {
  $('blue-basket')[0].refresh();
});
```

命令式调用DOM方法是很少见的，但是可以在视频元素api中找到。如果可能，最好使用声明式方法(属性更改)。

## 服务器端呈现/通用呈现

定制元素非常适合在浏览器中集成组件。但是，当构建一个web上可访问的站点时，初始加载性能很可能会影响到用户，在所有js框架下载并执行之前，用户会看到一个白屏。此外，考虑一下如果JavaScript失败或被阻止，站点会发生什么情况是很好的。Jeremy Keith在他的电子书/播客中解释了弹性网页设计的重要性。因此，在服务器上呈现核心内容的能力是关键。遗憾的是，web组件规范根本没有谈到服务器呈现。没有JavaScript，没有自定义元素:(