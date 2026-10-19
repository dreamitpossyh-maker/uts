# UTS RegExp `lastIndex` 使用指南

> 适用环境：uni-app x / UTS（uni-app TypeScript）
> 说明：`lastIndex` 是正则表达式实例的**可读写数值属性**，表示下一次匹配的起始位置。它只在带 `g`（global）或 `y`（sticky）标志时才有意义。

---

## 一、什么是 `lastIndex`

`lastIndex` 是 `RegExp` **实例**上的一个属性（不是静态属性），类型为 `number`。

- 它表示**下一次匹配从字符串的哪个索引开始**。
- 只有在正则带有 `g`（全局）或 `y`（粘性）标志时，`lastIndex` 才会被引擎使用和更新。
- 不带 `g`/`y` 时，`lastIndex` 始终为 `0`，且不会被 `exec`/`test` 修改。

```uts
const re = /a/g
console.log(re.lastIndex) // 0（初始值）
```

---

## 二、如何读写 `lastIndex`

### 1. 读取

直接访问属性即可：

```uts
const re = /a/g
re.test('aaa')
console.log(re.lastIndex) // 1（匹配到索引 0 的 'a'，下次从 1 开始）
```

### 2. 手动赋值

`lastIndex` 是**可写**的，可以手动设置，用来跳过前缀或从指定位置继续匹配：

```uts
const re = /a/g
re.lastIndex = 3          // 强制从索引 3 开始匹配
const m = re.exec('aXaXa')
console.log(m?.index)     // 4
```

> 手动赋值常用于：跳过某段前缀、从上次中断处继续、或重置匹配状态。

---

## 三、核心机制

`exec()` 和 `test()` 在带 `g`/`y` 时会**自动更新** `lastIndex`：

| 情况 | `lastIndex` 变化 |
| --- | --- |
| 匹配成功（带 `g`/`y`） | 更新为 `匹配起始索引 + 匹配文本长度` |
| 匹配失败（带 `g`/`y`） | 重置为 `0` |
| 不带 `g`/`y` | 始终为 `0`，不变化 |
| 匹配到空字符串 | 可能"卡住"（`lastIndex` 不前进），需手动 `lastIndex++` |

### 示例：`/a/g` 遍历 `'aXaXa'`

```uts
const re = /a/g
const str = 'aXaXa'

console.log(re.exec(str)?.index, re.lastIndex) // 0, 1
console.log(re.exec(str)?.index, re.lastIndex) // 2, 3
console.log(re.exec(str)?.index, re.lastIndex) // 4, 5
console.log(re.exec(str), re.lastIndex)        // null, 0（失败后重置）
```

---

## 四、与不带 `g`/`y` 的区别

```uts
// 不带 g：每次都从 0 开始，lastIndex 恒为 0
const re1 = /a/
re1.test('aaa')
console.log(re1.lastIndex) // 0

// 带 g：从 lastIndex 开始向后搜索
const re2 = /a/g
re2.test('aaa')
console.log(re2.lastIndex) // 1

// 带 y（sticky）：必须正好在 lastIndex 处匹配，否则失败并重置
const re3 = /a/y
re3.lastIndex = 1
console.log(re3.test('aaa')) // true（索引 1 正好是 'a'）
console.log(re3.lastIndex)   // 2
```

- **`g`**：从 `lastIndex` 开始**向后搜索**，找到任意位置即可。
- **`y`**：必须**正好在 `lastIndex` 处**匹配，否则失败并把 `lastIndex` 重置为 `0`。

---

## 五、典型使用场景

### 1. 用 `exec` + `g` 循环遍历所有匹配

```uts
const re = /\d+/g
const str = 'a1b22c333'
let m: RegExpExecArray | null
while ((m = re.exec(str)) !== null) {
  console.log(m[0], m.index)
}
// 1 1 / 22 3 / 333 6
```

### 2. 从指定位置继续匹配

```uts
const re = /foo/g
re.lastIndex = 10   // 跳过前 10 个字符
const m = re.exec(longText)
```

### 3. 粘性解析 / 词法分析（配合 `y`）

```uts
const token = /\d+/y
let pos = 0
const input = '12ab34'
token.lastIndex = pos
while (token.test(input)) {
  pos = token.lastIndex
  // 处理匹配到的数字 token
}
```

### 4. 重置状态，避免残留 `lastIndex`

复用同一个带 `g` 的正则对象时，务必在重新开始前重置：

```uts
const re = /a/g
re.lastIndex = 0   // 重置，避免上次匹配残留影响本次结果
```

---

## 六、注意事项与兼容性

1. **复用带 `g` 的正则要小心**：`lastIndex` 会保留上次的状态，导致"时对时错"。跨多次调用时建议每次重置 `lastIndex = 0`，或使用新实例。
2. **空匹配死循环**：当正则能匹配空字符串（如 `/a*/g`）时，`lastIndex` 可能不前进，`while` 循环会卡死。需手动推进：
   ```uts
   if (m[0] === '') re.lastIndex++
   ```
3. **`test()` 也会修改 `lastIndex`**：不只是 `exec()`，`test()` 在带 `g`/`y` 时同样会更新 `lastIndex`。
4. **`match` / `matchAll` 的行为**：
   - `String.prototype.match` 带 `g` 时返回所有匹配，但**不**依赖/暴露 `lastIndex`。
   - `String.prototype.matchAll` 要求正则带 `g`，内部会复制正则，**不修改**原正则的 `lastIndex`。
5. **引擎兼容性**：HBuilderX 4.72 之前 Android 端使用 PCRE 引擎；4.72 及之后完全遵循 ECMAScript 标准（与 Web 一致），`lastIndex` 行为与浏览器一致。
6. **编译产物**：UTS 中 `RegExp` 编译为 Kotlin 的 `io.dcloud.uts.UTSRegExp`，`lastIndex` 语义遵循 ECMAScript 规范。

---

## 一句话总结

`lastIndex` 是带 `g`/`y` 正则的"**游标**"：匹配成功则前进到匹配末尾，失败则归零；不带 `g`/`y` 时恒为 `0`。手动读写它可实现"跳过前缀""从指定位置继续""重置状态"等控制。
