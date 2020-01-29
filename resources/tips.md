---
description: 本页收集汇总了一些很有用但不容易记忆或查找到出处的代码片段
---

# 技巧汇总

### 测试 <a id="testing"></a>

#### 如何在 test helper 中查找到一个指定的 service？

```javascript
import { getContext } from '@ember/test-helpers';

// when you want to retieve the current context:
let context = getContext();

// When you want to get access to the owner of:
let owner = context.owner;

// When you want to look up a service:
let fooService = owner.lookUp('service:foo');
```



