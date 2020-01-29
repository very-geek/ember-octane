# 基本用法

{% hint style="info" %}
#### 给初学者

除了这本册子相信大家也看过其他的文章，有很多介绍对象模型的例子（特别是在 Ember Octane 以前）都直接使用了 `EmberObject` 来讲述，这可能会让初学者有一些困惑，因为在实际使用 Ember.js 框架的时候我们并不经常使用 `EmberObject` 的。在这里我需要解释一下：

1. 实际上 `EmberObject` 无处不在：`Route`，`Controller`，`Service`，`Component` 等等都是扩展自 `EmberObject` 的。我们很少直接使用它是因为大部分情况下框架都为我们准备好了更具体的基于 `EmberObject` 的子系统（比如上面列举的这些），又或者是我们不知道什么时候是直接使用 `EmberObject` 的好时机（这个已经不重要了，因为从 Ember Octane 开始就真的不再需要直接用它了）
2. 我们看到的教程一般都是为了讲解最核心、最基本的概念而设计的，所以会直接使用 `EmberObject`，因为它是 Classic Ember 对象模型里的根基；也可以这么说：只要能用它讲明白的问题，那么相同的理论运用在其他一切基于它的上层（比如 1. 里介绍的那些）身上都是一样的

接下来我们将基于 Ember Octane 来学习对象模型的基本知识，好消息是新的概念要比过去简单得多，比方说你不需要纠结“什么时候该用 `EmberObject` 而什么时候不必“等问题。希望上面的解释能让大家彻底丢掉过去的包袱，重新开始轻装上阵。
{% endhint %}

### 再见！`EmberObject` <a id="farewell-emberobject"></a>

Ember Octane 带来的最根本的变化就是对于原生类的完全兼容，要理解这一点我们来看一个例子：

{% tabs %}
{% tab title="Step 1" %}
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
```markup
First Name: {{@model.firstName}}
Last Name: {{@model.lastName}}
```
{% endcode %}

如果我们要渲染一个全名呢？最单纯的做法无非是直接在模版上拼接：

{% code title="app/templates/hero.hbs" %}
```text
Full Name: {{@model.firstName}} {{@model.lastName}}
```
{% endcode %}

对于这个极简单的例子来说是没问题的，但现实情况下并不总是如此简单。看下一个例子——
{% endtab %}

{% tab title="Step 2" %}
比方说我们得到的数据变成了：

![](../../.gitbook/assets/image%20%281%29.png)

`dob` 是 Unix Timestamp 所表示的出生日期，如果我们要用对人类友好的方式来显示生日，那么就很难直接在模板上进行转换了。

在 Classic Ember 时代，上面的诉求通常都会经由 `Controller` 来帮我们实现，例如：

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

```text
Full Name: {{@model.firstName}} {{@model.lastName}}
Birthday: {{this.birthday}}
```

做到这一步似乎就已经大功告成了，但在这里我必须要指出：这样做是错误的！
{% endtab %}
{% endtabs %}

