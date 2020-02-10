---
description: 以一个全新创建的 Ember Octane App 为基础介绍如何整合 Typescript
---

# 整合 TypeScript

{% hint style="info" %}
这一部分内容是我录制的 Ember Octane 官方教程系列视频的补充内容，也是我极力推荐给初学者的知识点，即：使用 Typescript 来编写 Ember Octane 应用程序。

关于这个主题我有很多可以谈的，但这一篇的主题聚焦于如何整合 Typescript 到一个新创建的 Ember Octane App 中去。这部分内容以操作为主，且随着时间的推移会有内容不断更新，所以不便于占用视频的宝贵时间。如果你想知道如何安装和配置 Typescript，那这篇文章非常适合。
{% endhint %}

## 安装 ember-cli-typescript

[ember-cli-typescript](https://ember-cli-typescript.com/) 是我们为 Ember Octane App 添加 TypeScript 支持的最佳选择。安装它只需要在项目根路径下执行命令：

```bash
$ ember install ember-cli-typescript
```

这条命令同时还会帮我们安装下列几个依赖包：

1. ember-cli-typescript-blueprints
2. typescript
3. @types/ember
4. @types/rsvp
5. @types/ember\_\_test-helpers
6. @types/ember-data
7. @types/ember-qunit
8. @types/qunit

其中以 `@types` 开头的六个是为了给 Ember.js 框架（包括 Ember Data 在内）提供配套的类型声明文件。

### 删除不兼容的 ember-cli-typescript-blueprints（暂时）

Blueprints 是 ember-cli 框架中的一个机制，它的作用给 `ember generate xxx` 命令提供默认的文件模板。当我们使用 TypeScript 之后，原则上我们通过 `ember generate` 创建出的文件也应该是 `.ts` 文件，它们的默认内容也应该和 `.js` 有所区别。

但因为 Ember Octane 刚刚发布不久，目前最新版本的 ember-cli-typescript（v3.1.3）还没有完全做好兼容性处理，因此默认安装的 ember-cli-typescript-blueprints 应该删除掉，以确保 `ember generate` 命令可以生成正确的文件内容（文件的类型则暂时需要我们自行修改成 `.ts`）。可以执行以下命令：

```bash
$ yarn remove ember-cli-typescript-blueprints
# 或
$ npm uninstall ember-cli-typescript-blueprints
```

### 将缺省创建的文件转换成 `.ts` 格式（可选）

TypeScript 本质上是 JavaScript 的超集，所以一个 TypeScript 项目中同样可以使用 `.js` 文件。新创建的 Ember Octane App 里有几个默认的 `.js` 文件，你可以把它们转成 `.ts` 格式——当然这一步不是必须的！

以下是转换后的文件内容及一些说明：

{% tabs %}
{% tab title="app/app.ts" %}
```typescript
import Application from '@ember/application';
import Resolver from 'ember-resolver';
import loadInitializers from 'ember-load-initializers';
import config from 'super-rentals/config/environment';

export default class App extends Application {
  modulePrefix = config.modulePrefix;
  podModulePrefix = config.podModulePrefix;
  Resolver = Resolver;
}

loadInitializers(App, config.modulePrefix);
```

#### 添加缺失的类型声明

此文件在更改为 `.ts` 类型之后，会报如下图的错误：

![](../.gitbook/assets/image%20%284%29.png)

这是因为 `ember-resolver` 没有匹配的类型声明，ember-cli-typescript 里也没提供。

解决办法，添加 `types/ember-resolver/index.d.ts` 文件，内容如下：

{% code title="types/ember-resolver/index.d.ts" %}
```typescript
declare module 'ember-resolver';
```
{% endcode %}

#### 使用绝对路径引用模块

另外在第 4 行，我将以前的：`import config from './config/environment';` 改成了 `import config from 'super-rentals/config/environment';` 

这一改动也不是必须的，但至少在 config 这个例子中我推荐使用绝对路径。关于这一点所产生的影响，请继续往后看——
{% endtab %}

{% tab title="app/router.ts" %}
```typescript
import EmberRouter from '@ember/routing/router';
import config from 'super-rentals/config/environment';

export default class Router extends EmberRouter {
  location = config.locationType;
  rootURL = config.rootURL;
}

Router.map(function() {});
```

这个文件的改动就只有第 2 行的引用路径。
{% endtab %}

{% tab title="tests/test-helper.ts" %}
```typescript
import { setApplication } from '@ember/test-helpers';
import Application from 'super-rentals/app';
import config from 'super-rentals/config/environment';
import { start } from 'ember-qunit';

setApplication(Application.create(config.APP));

start();
```

#### 解决“找不到模块../app“的错误

报错如图：

![](../.gitbook/assets/image%20%285%29.png)

这个问题一开始会让人很疑惑，因为即使不做修改直接运行也是可以的，那为什么换成 TypeScript 就会报错了呢？

关于 TypeScript 有一个很重要的概念：**TypeScript 只作用于代码的编译时，而完全不会影响代码的运行时。**

也就是说，只有在编译的时候 TypeScript 才会起作用（同理 Babel 也是如此），而一旦编译好的代码开始运行（比如在浏览器中加载之后）就再也不会有 TypeScript 什么事儿了。

因此，上面的代码运行不会出错是因为 Ember CLI 在构建时重新映射了模块的组织关系（类似于 Webpack 所做的工作）。新的关系在绝大多数情况下和你在硬盘上看到的文件/目录关系是一致的，但有个别之处是不同的，`tests/test-helper.ts` 就是一个不同的例子。

这是物理文件结构，也就是你在硬盘上、编辑器里看到的文件结构（省略了无关文件/目录）：

```text
├── app
│   ├── app.ts
│   ├── ...
├── tests
│   ├── test-helper.ts
│   └── ...
└── ...
```

而这是加载到浏览器之后重新映射的模块结构（省略了无关模块）：

```text
├── app
│   ├── app.ts
│   ├── tests
│   ├───── test-helper.ts
│   └───── ...
└── ...
```

因为 TypeScript 只会参与到编译时的代码，所以它只能“看见“物理的文件结构，而无法知道编译后由 Ember CLI 重新映射后的相对关系——这就是问题的根本原因。

解决问题的办法很简单：使用绝对路径。即写为：`import Application from 'super-rentals/app';` 

对于编译时，TypeScript 通过 `tsconfig.json` 文件中的 `path` 配置来匹配正确的路径：

{% code title="tsconfig.json" %}
```javascript
{
  "compilerOptions": {
    // ...
    "paths": {
      "super-rentals/tests/*": ["tests/*"],
      "super-rentals/*": ["app/*"],
      "*": ["types/*"]
    }
  }
}
```
{% endcode %}

而对于运行时，Ember CLI 编译后也赋予了 `app/app.ts` 同样的模块名称，所以两者完美匹配。

#### 适配 `config/environment.js` =&gt; `app/config/environment.d.ts`

还有一个错误如下图所示：

![](../.gitbook/assets/image%20%286%29.png)

这个问题就更加“扑朔迷离“了。乍一看 `config/environment.js` 是一个 Node.js  Module，它怎么能在运行于浏览器的 JS Module 里直接 import 呢？其次，就算能 import，那么 `config/environment.js` 里面明明有 `APP` 对象，为什么 TypeScript 会认为它不存在呢？

没错，`config/environment.js` 的确是一个 Node.js Module，它的存在是用于_在 Ember CLI 启动后动态输出应用程序的配置对象，然后把这个配置对象注入到将在浏览器中执行的 JS 代码中去，以便让开发者可以在运行时获取这些配置信息的_。

因为要动态输出（其动态性主要基于 `process.env.EMBER_ENV` 环境变量，这个变量是在执行 `ember xxx` 命令时决定其值的），所以它不能直接导出一个对象，而是导出了一个函数，这个函数将由 Ember CLI 在执行后调用。那么前面讲过 TypeScript 只参与编译时的工作，所以对它来说 `config/environment.js` 导入后就是一个 `Function`，而这个 `Function` 的确没有 `APP` 属性呀！

然而我们知道我们 import 过来的 config 不是那个函数，而是那个函数返回的对象，可是我们该如何让 TypeScript 知道这一点呢？

请注意我们的项目里多了一个文件：`app/config/environment.d.ts`，这个文件恰好就是运行时注入到 JS 代码里的配置对象的模块位置，所以我们就可以改成用绝对路径来导入配置对象，那么 TypeScript 就“以为“我们导入的是一个对象，而非之前的那个函数了。

但是这个新的文件里缺少了对于 `APP` 属性的定义，所以我们要修改其内容为：

{% code title="app/config/environment.d.ts" %}
```typescript
declare const config: {
  environment: any;
  modulePrefix: string;
  podModulePrefix: string;
  locationType: string;
  rootURL: string;
  APP: any;
};

export default config;
```
{% endcode %}

这样，所有的问题就全都解决了。
{% endtab %}
{% endtabs %}

## 配置开发环境

### VS Code

