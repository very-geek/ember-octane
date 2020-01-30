# 概述

### 沉重的历史 <a id="historic-burden"></a>

在 Ember Octane 以前，官方文档里专门有一个讲述对象模型的板块，其内容大概是下列几点：

1. Ember Object（类、对象、重定义静态成员或原型属性、追加实例属性等）
2. 反应系统（Computed Properties、Observers）
3. 数据绑定
4. 可枚举对象（在框架中对 enumerable 的扩展支持）

这部分内容对于初学者来说是不友好的，一方面是难懂，另一方面是不明白为什么要懂，或者说懂了以后究竟有什么作用？这些问题是很难一下子在头脑中形成明确概念的。

但这部分内容又是十分重要的。JavaScript 本质上还是面向对象语言（尽管它具备很多函数式语言的重要特性），但其语言自身为对象提供的支持又不足以满足 UI 开发的所有需求，所以框架的一个核心任务就是增强和/或扩展 JavaScript 语言的对象模型来解决“不够用和不好用“的问题。

Ember Classic 的问题在于它的对象模型相较于其他前端框架而言过于复杂了，以至于在其他框架的文档里只需要寥寥几笔就能概述的问题（但也会有额外篇幅专门解释进阶的用法和原理）在这里却需要六七篇独立的文档才能讲完整，讲清楚。Ember Classic 会给初学者以“学习曲线陡峭“的第一印象很大程度上都来源于此。

### 崭新的开始 <a id="a-new-beginning"></a>

幸好在 Ember Octane 到来之后，这个问题得到了根本性的解决。对比最后一个 Ember Classic 版本（v3.14）的文档，我们可以看到以下改变：

#### ~~Ember Object（类、对象、重定义静态成员或原型属性、追加实例属性等）~~ <a id="optional-ember-object"></a>

由于 Ember Octane 已经不再推荐使用 EmberObject 了（但它依然存在，并且依然可用，短期内也没有废弃的计划），所以相关的内容亦不必赘述——只要你会用标准的 JavaScript Class，在 Ember Octane 里就还是一样的用。于是在新的文档里就只剩一篇单独的进阶文章讲解标准的 ES2015 Class 在 Ember Octane 里的基本使用方法。

{% hint style="info" %}
关于对象模型的演变，Core Team 成员 [Chris Garrett](https://www.pzuraq.com/author/pzuraq/) 撰写了一篇专文来介绍更多的细节知识。欲知详情，请移步：[Do You Need EmberObject](https://www.pzuraq.com/do-you-need-ember-object/)
{% endhint %}

#### ~~反应系统（Computed Properties、Observers）~~ =&gt; Auto-Tracking System <a id="auto-tracking-system"></a>

Computed Properties 和 Observers 依然存在，但是新的文档里已经看不到它们出现了。这是因为全新的自动跟踪（Auto-Tracking）反应系统大大缩减了过去在这方面需要付出的学习成本。一个简单的 `@tracked` 装饰器合理的出现在其他关键环节的文档里（比如 Components），不需要单独的篇幅就可以轻易理解它的用途。当然也有一篇单独介绍 Auto-Tracking System 的文章，适合进阶阅读。

{% hint style="info" %}
Chris Garrett 另有一篇单独介绍 Auto-Tracking System 的文章：[Thinking With Auto-tracking: What Is Reactivity?](https://www.pzuraq.com/thinking-with-autotracking-what-is-reactivity/) 这篇文章更具备深度和广度，适合学有余力者开拓视野和加深理解所用。
{% endhint %}

#### 数据绑定与可枚举对象 <a id="binding-and-enumerable"></a>

这两点内容也在新版文档中消失了，但并不是因为不重要或者不存在，而是因为一方面新的对象模型与反应系统将过去的很多问题消弭于无形之中，另一方面则是这两点也具备它们各自的特点，而新版文档选择不过度强调这些，也是为了尽可能不要给初学者带来过多的学习负担吧。

下面我们就以 Ember Octane 的视角来看看现在的对象模型以及反应系统的基本用法吧。
