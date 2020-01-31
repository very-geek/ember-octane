---
description: 本页收集汇总了一些很有用但不容易记忆或查找到出处的代码片段
---

# 技巧汇总

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

## 工具

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

