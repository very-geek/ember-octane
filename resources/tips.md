---
description: 本页收集汇总了一些很有用但不容易记忆或查找到出处的代码片段
---

# 技巧汇总

## 调试 <a id="debugging"></a>

### 如何调试 babel 转译后的代码？ <a id="how-to-enable-babel-sourcemaps-support"></a>

默认情况下，你在 ember.js app 所写的代码都会进过 babel 处理转译之后再在浏览器中运行，这样一来你会发现调试的时候很痛苦，因为 babel 已经把你的代码搅得“面目全非“了。

可以通过开启 babel 的 sourceMaps 支持来解决这个问题，只需要在 `ember-cli-build.js` 文件中加入下列配置即可：

{% code title="ember-cli-build.js" %}
```javascript
const app = new EmberApp(defaults, {
  // ...
  
  babel: {
    sourceMaps: process.env.EMBER_ENV !== 'production' ? 'inline' : null,
  },
});
```
{% endcode %}

这一行代码仅在应用以 `production` 环境编译的时候关闭 sourceMaps，其他情况下都会开启内联 sourceMaps 的支持（包括 `development` 和 `test` 环境）。如果你不介意在生产环境的代码中也包含 sourceMaps 的话，也可以简单写成：`sourceMaps: 'inline'` 。

{% hint style="info" %}
#### 为什么要在生产环境中包含 sourceMaps？ <a id="why-need-sourcemaps-in-production-environment"></a>

sourceMaps 主要用于在源代码和编译后的源代码之间做一个匹配映射，以便于调试代码，或者代码运行出错之后可以准确定位到编译前的源代码位置。

尽管我们通常只在开发或测试环境下才需要它，但有些产品非常关注生产环境时的问题处理，它们会采用一些在线的错误收集和跟踪服务（比如 Rollbar、Sentry 等产品）来帮助它们达成这一目标。而这些服务要想良好工作，就需要有 sourceMaps 的支持。

当然，更加“完美“的流程应该是在开发/测试环境的时候采用内联 sourceMaps，而在生产环境时生成独立于源代码之外的 sourceMaps 文件，然后通过自动化流程将这些 sourceMaps 文件上传到在线错误收集和跟踪服务能够访问到的地方上。这样既可以保证生产环境上代码的“干净、轻量“，又可以完美利用好在线错误收集和跟踪服务。
{% endhint %}

## 测试 <a id="testing"></a>

### 如何在 test helper 中查找到一个指定的 service？ <a id="look-up-a-service-in-test-helpers"></a>

```javascript
import { getContext } from '@ember/test-helpers';

// when you want to retieve the current context:
let context = getContext();

// When you want to get access to the owner of:
let owner = context.owner;

// When you want to look up a service:
let fooService = owner.lookUp('service:foo');
```

## 工具 <a id="tooling"></a>

### 如何让 ember-cli-template-lint 不检查 local linked add-on 中的模板？ <a id="not-to-lint-templates-in-a-linked-addon"></a>

在本地开发时，为了方便调试有时候会使用诸如 `npm link` 或 `yarn link` 来创建某个 add-on 的本地链接，这样可以在应用程序里直接调试 add-on 在本地的代码而不需要每次修改都重新发布 add-on。

又因为这种本地链接的 add-on 需要开启

```javascript
isDevelopingAddon() { return true; }
```

才可以让 add-on 触发自动刷新，而这个 hook 会导致 ember-cli-template-lint 针对本地链接的 add-on 里的模板进行语法检查。

有时候我们不希望出现这种行为（比如说这个 add-on 不是你开发的，你调试它只是为了解决某个 bug，但你并不希望帮它处理模版上存在语法问题），一个简单的解决办法就是：

```bash
$ yarn remove ember-cli-template-lint
$ ember install ember-template-lint
```

