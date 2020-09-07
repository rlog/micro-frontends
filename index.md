使多个团队可以使用不同的技术栈来开发现代web应用

## 什么是微前端

`微前端`这个词最早出现在2016年底的 [ThoughtWorks Technology Radar](https://www.thoughtworks.com/radar/techniques/micro-frontends) 上。它把微服务的概念扩展到前端领域。当前的趋势是构建功能丰富、强大的浏览器应用，又称单页应用，它位于微服务架构之上。前端层通常由独立的团队开发，随着时间的推移，前端层会不断增长，并且越来越难以维护。这就是我们所说的前端巨石 [Frontend Monolith](https://www.youtube.com/watch?v=pU1gXA0rfwc)。

微前端背后的理念是，把一个网站或web应用看作是由不同独立团队所开发的功能组合。每个团队都有自己关心并专门从事的业务或领域。单一的团队是跨职能的端到端开发，从数据库到用户界面。

这并不是一个全新的架构。它与 [Self-contained Systems](https://scs-architecture.org/) 有许多共同之处。过去这种方法被称为 垂直系统前端集成 [Frontend Integration for Verticalised Systems](https://dev.otto.de/2014/07/29/scaling-with-microservices-and-vertical-decomposition/)。但微前端显然是一个更好的名字。

**Monolithic Frontends**
![Monolithic Frontends](https://micro-frontends.org/ressources/diagrams/organisational/monolith-frontback-microservices.png)

**Organisation in Verticals**
![Organisation in Verticals](https://micro-frontends.org/ressources/diagrams/organisational/verticals-headline.png)

## 什么是 Modern Web App？

在介绍中，我使用了“构建一个现代web应用程序”这个短语。让我们定义与这一项相关的假设。

为了从更广泛的角度来看待这个问题，阿拉尔巴尔干写了一篇博客文章，内容是关于他所谓的“文件到申请的连续性”。他提出了一个滑动比例的概念，即一个由静态文档构建、通过链接连接的站点位于左侧，而一个纯行为驱动、无内容的应用程序(如在线照片编辑器)位于右侧。

如果您将您的项目放在这个范围的左边，那么web服务器级别的集成是一个很好的选择。通过这个模型，服务器可以从组成用户请求的页面的所有组件中收集和连接HTML字符串。更新是通过从服务器重新加载页面或通过ajax替换部分页面来完成的。古斯塔夫·尼尔森·科特就这个话题写了一篇综合性的文章。

当您的用户界面必须提供即时反馈时，即使是在不可靠的连接上，纯服务器呈现的站点也不再足够。要实现乐观UI或骨架屏幕这样的技术，您还需要能够更新设备本身的UI。谷歌的术语“渐进Web应用程序”恰当地描述了在提供类似应用程序的性能的同时，作为Web的好公民(渐进增强)的平衡行为。这种应用程序位于站点-应用程序-连续体的中间位置。在这里，一个单独的基于服务器的解决方案已经不够了。我们必须将集成移到浏览器中，这是本文的重点。

## 微前端的核心理念

* **技术无关性**
  每个团队应该能够选择和升级他们的堆栈，而不必与其他团队协调。定制元素是隐藏实现细节的好方法，同时为其他元素提供了一个中立的接口。

* **团队代码隔离**
  不共享运行时，即使所有团队使用相同的框架。构建独立的、自包含的应用程序。不要依赖于共享状态或全局变量。

* **建立团队前缀**
  在还不可能实现隔离的情况下，建立团队前缀就命名约定达成一致。命名空间CSS，事件，本地存储和cookie，以避免冲突和澄清所有权。

* **尽量使用浏览器原生能力，而不是自定义API**
  使用浏览器事件进行通信，而不是构建一个全局的PubSub系统。如果你真的需要建立一个跨团队的API，试着让它尽可能的简单。

* **构建一个有弹性的网站**
  你的特性应该是有用的，即使JavaScript失败或尚未执行。使用通用渲染和渐进增强来提高感知性能。

## DOM 就是 API

自定义元素(Web组件规范中的互操作性方面)是集成到浏览器中的一个很好的原语。每个团队使用他们选择的web技术构建他们的组件，并将其包装在一个定制元素中(例如 )。这个特定元素(标记名、属性和事件)的DOM规范充当其他团队的契约或公共API。其优点是他们可以使用组件及其功能，而不需要知道实现。它们只需要能够与DOM交互。

但是，定制元素本身并不能解决我们所有的需求。为了实现渐进式增强、通用渲染或路由，我们需要额外的软件。

本页主要分为两个部分。首先，我们将讨论页面组合——如何用不同团队拥有的组件组装页面。之后，我们将展示实现客户端页面转换的示例。

## 页面组成

在客户端和服务端集成的代码写在不同的框架本身,还有很多方面的话题,应该讨论:机制来隔离js,避免css冲突,根据需要加载资源,团队之间共享公共资源,处理数据抓取,想想好为用户加载状态。我们将一步一步地讨论这些主题。

### 基础原型

这个型号拖拉机商店的产品页面将作为下面示例的基础。

它的特点是变型选择开关之间的三种不同的拖拉机模型。在改变产品形象，名称，价格和建议更新。还有一个购买按钮，它将选中的变量添加到购物篮中，顶部还有一个相应更新的迷你购物篮。

![The Base Prototype](https://micro-frontends.org/ressources/video/model-store-0.gif)

[在浏览器里试试](https://micro-frontends.org/0-model-store/) & [看看代码](https://github.com/neuland/micro-frontends/tree/master/0-model-store)

所有的HTML都是在客户端使用无依赖的普通JavaScript和ES6模板字符串生成的。代码使用简单的状态/标记分离，并在每次更改时重新呈现整个HTML客户端——目前没有花哨的DOM差异和通用的呈现。也没有团队分离-代码是写在一个js/css文件。

### 客户端集成

在这个例子中，页面被分割成由三个团队拥有的独立组件/片段。团队结帐(蓝色)现在负责所有与购买过程有关的事情-即购买按钮和迷你篮子。Team Inspire(绿色)管理这个页面上的产品推荐。页面本身属于Team Product(红色)。

![](https://micro-frontends.org/ressources/screen/three-teams.png)

[在浏览器里试试](https://micro-frontends.org/1-composition-client-only/) & [看看代码](https://github.com/neuland/micro-frontends/tree/master/1-composition-client-only)

Team Product决定包含哪些功能以及它在布局中的位置。该页面包含团队产品本身提供的信息，如产品名称、图像和可用的变体。但是它也包括来自其他团队的片段(定制元素)。

### 如何创建一个自定义元素?

让我们以购买按钮为例。团队产品包含的按钮，只需添加 到标记中的所需位置。要使其工作，Team Checkout必须在页面上注册 `blue-buy` 元素。

```js
class BlueBuy extends HTMLElement {
  connectedCallback() {
    this.innerHTML = `<button type="button">buy for 66,00 €</button>`;
  }

  disconnectedCallback() { ... }
}
window.customElements.define('blue-buy', BlueBuy);
```