# 基本用法

{% hint style="info" %}
#### 给初学者

除了这本册子相信大家也看过其他的文章，有很多介绍对象模型的例子（特别是在 Ember Octane 以前）都直接使用了 `EmberObject` 来讲述，这可能会让初学者有一些困惑，因为在实际使用 Ember.js 框架的时候我们并不经常使用 `EmberObject` 的。在这里我需要解释一下：

1. 实际上 `EmberObject` 无处不在：`Route`，`Controller`，`Service`，`Component` 等等都是扩展自 `EmberObject` 的。我们很少直接使用它是因为大部分情况下框架都为我们准备好了更具体的基于 `EmberObject` 的子系统（比如上面列举的这些），又或者是我们不知道什么时候是直接使用 `EmberObject` 的好时机（这个已经不重要了，因为从 Ember Octane 开始就真的不再需要直接用它了）
2. 我们看到的教程一般都是为了讲解最核心、最基本的概念而设计的，所以会直接使用 `EmberObject`，因为它是 Ember Classic 对象模型里的根基；也可以这么说：只要能用它讲明白的问题，那么相同的理论运用在其他一切基于它的上层（比如 1. 里介绍的那些）身上都是一样的

接下来我们将基于 Ember Octane 来学习对象模型的基本知识，好消息是新的概念要比过去简单得多，比方说你不需要纠结“什么时候该用 `EmberObject` 而什么时候不必“等问题。希望上面的解释能让大家彻底丢掉过去的包袱，重新开始轻装上阵。
{% endhint %}

### 再见！`EmberObject` <a id="farewell-emberobject"></a>

Ember Octane 带来的最根本的变化就是对于原生类的完全兼容，要理解这一点我们来看一个例子：

{% tabs %}
{% tab title="Demo 1" %}
比方说，我们可以通过后端 API 获得某位漫威英雄的基本信息：

{% code title="app/routes/hero.js" %}
```javascript
import Route from '@ember/routing/route';

export default class HeroRoute extends Route {
    async model() {
        let hero = await this.store.findRecord('hero', 'ironman');
        return hero;
    }
}
```
{% endcode %}

假定返回的数据结构如下图所示：

![](../../.gitbook/assets/image.png)

那么我们可以像这样来渲染模板：

{% code title="app/templates/hero.hbs" %}
```handlebars
First Name: {{@model.firstName}}
Last Name: {{@model.lastName}}
```
{% endcode %}

如果我们要渲染一个全名呢？最单纯的做法无非是直接在模版上拼接：

{% code title="app/templates/hero.hbs" %}
```handlebars
Full Name: {{@model.firstName}} {{@model.lastName}}
```
{% endcode %}

对于这个极简单的例子来说是没问题的，但现实情况下并不总是如此简单。请看 Demo 2：
{% endtab %}

{% tab title="Demo 2" %}
比方说我们得到的数据变成了：

![](../../.gitbook/assets/image%20%281%29.png)

`dob` 是 Unix Timestamp 所表示的出生日期，如果我们要用对人类友好的方式来显示生日，那么就很难直接在模板上进行转换了。

在 Ember Classic 时代，上面的诉求通常都会经由 Controller 来帮我们实现，例如：

{% code title="app/controllers/hero.js" %}
```javascript
import Controller from '@ember/controller';

export default class HeroController extends Controller {
    @computed('model.dob')
    get birthday() {
        let date = new Date(this.model.dob);
        return date.toDateString(); // => Sun Apr 4, 1965
    }
}
```
{% endcode %}

然后：

```handlebars
Full Name: {{@model.firstName}} {{@model.lastName}}
Birthday: {{this.birthday}}
```

做到这一步似乎就已经大功告成了，但在这里我必须要指出：这样做是错误的！
{% endtab %}
{% endtabs %}

我有一千个理由来陈述为什么不应该使用 Controller，在这里最合适的理由就是“职责混乱“。再看一眼最终的模板，我们发现：渲染姓名使用的是 `{{@model.xxx}}` 但渲染生日却用的是 `{{this.birthday}}`。这是因为 `birthday` 是我们在 Controller 里创造的衍生属性，可是直觉上又会觉得 `birthday` 不应该属于 Controller。

![Ember.js &#x5E94;&#x7528;&#x7A0B;&#x5E8F;&#x7684;&#x7ED3;&#x6784;&#x56FE;&#xFF08;&#x90E8;&#x5206;&#x6807;&#x6CE8;&#xFF09;](../../.gitbook/assets/jie-ping-20200130-shang-wu-10.45.50.png)

上图是官网文档中介绍框架结构的示意图，我们的演示集中在图中蓝色矩形里面的部分，红色圆圈里的 Model 就是通过路由 `model() {}` 返回的可供模板直接渲染的对象。在这里其实隐含了一个很关键的问题：**如果 `@model` 不够用怎么办？**

假如我们无视 Controller 的存在，那么惯例上我们可以通过创建 Component，然后在 Component 的内部完成对 `@model` 数据的进一步修饰（就好像 Demo 2 里创造的衍生属性 `birthday` 一样）。然而这么做有时候很合适，有时候则不一定，两者之间的分界在于我们要衍生的东西，究竟是属于哪边的？业务？还是 UI？

在我们这个例子里，很显然 `birthday` 和 UI 没有直接关系，它是 `@model.dob` 的以另外一种格式存在的数据。如果有一个 Component 需要渲染一位漫威英雄的基本资料，那么这个组件应该只需要接收到数据然后直接渲染就够了，数据要怎么“装饰“则超出了组件了职责范畴。

Controller 的问题就在于此：它不是组件，也不单纯是业务逻辑的领域；它介乎于二者之间的同时又有着独特的性质（比如说 Controller 都是单例对象且一旦生成就不会销毁）。所以它在 Ember.js 应用程序之中的职责总是模糊而暧昧的，那些非常依赖 Controller 的 Ember.js 应用程序总是看起来十分臃肿且晦涩难懂。

OK，Controller 不能用，Component 不见得合适，那么这个问题究竟应该如何处理较好呢？我们继续看 Demo 3：

{% tabs %}
{% tab title="Demo 3" %}
这一次我们把生成衍生的 `birthday` 属性的职责转移到正确的地方——专门表示漫威英雄的 `Hero` Class：

{% code title="app/routes/hero.js" %}
```javascript
import EmberObject, { computed } from '@ember/object';
import Route from '@ember/routing/route';

class Hero extends EmberObject {
    @computed('firstName', 'lastName')
    get fullName() {
        return `${this.firstName} ${this.lastName}`;
    }

    @computed('dob')
    get birthday() {
        let date = new Date(this.model.dob);
        return date.toDateString(); // => Sun Apr 4, 1965
    }
}

export default class HeroRoute extends Route {
    async model() {
        let hero = await this.store.findRecord('hero', 'ironman');
        return Hero.create(hero);
    }
}
```
{% endcode %}

这也是 Ember Classic 时代我们做这样的事情需要依赖 `EmberObject` 的主要原因：我们的 `Hero` Class 需要 Computed Property 特性的支持。

于是我们的模版就可以写成：

{% code title="app/templates/hero.hbs" %}
```handlebars
Full Name: {{@model.fullName}}
Birthday: {{@model.birthday}}
```
{% endcode %}

漂亮！你还可以看到我顺便也写了一个 `fullName` 属性，这当然不是必须的，但这么做会让模版变得更易读和易于维护。

然而本节内容的标题是：再见！`EmberObject`，所以这还没完，继续看 Demo 4：
{% endtab %}

{% tab title="Demo 4" %}
之前说到，Ember Octane 是可以完全使用原生的 ES2015 Class 的，所以这个例子还可以进一步改进：

{% code title="app/routes/hero.js" %}
```javascript
import { computed } from '@ember/object';
import Route from '@ember/routing/route';

class Hero {
    constructor(attrs) {
        this.firstName = attrs.firstName || 'John';
        this.lastName = attrs.lastName || 'Doe';
        this.dob = attrs.dob || -2209017943000; // => Mon Jan 1, 1900
    }

    @computed('firstName', 'lastName')
    get fullName() {
        return `${this.firstName} ${this.lastName}`;
    }

    @computed('dob')
    get birthday() {
        let date = new Date(this.model.dob);
        return date.toDateString(); // => Sun Apr 4, 1965
    }
}

export default class HeroRoute extends Route {
    async model() {
        let hero = await this.store.findRecord('hero', 'ironman');
        return new Hero(hero);
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}
