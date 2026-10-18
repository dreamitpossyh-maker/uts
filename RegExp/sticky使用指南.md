# UTS RegExp sticky 使用指南

> `sticky` 是 `RegExp` 的只读属性：当正则带有 **`y`（sticky）标志** 时为 `true`，否则为 `false`。
> 它表示该正则在进行 `exec` / `test` 时，会**固定从 `lastIndex` 指定位点开始匹配**，而不是在整个字符串里自由寻找。

## 一、什么是 sticky

`sticky`（粘性匹配）描述的是「匹配锚定到 `lastIndex`」这一行为：

- 带 `y` 标志的正则，`exec` / `test` **只从 `lastIndex` 尝试匹配**，不向前向后搜索。
- 匹配成功后 `lastIndex` 自动推进到本次匹配的结束位置。
- 若 `lastIndex` 处匹配失败，`lastIndex` 会**重置为 0**。

```ts
console.log(/\d+/y.sticky)                 // true
console.log(/\d+/.sticky)                 // false
console.log(new RegExp('foo', 'y').sticky) // true
```

## 二、如何启用 sticky

- 字面量写法：在斜杠后加 `y`，例如 `/foo/y`。
- 构造函数写法：`new RegExp('foo', 'y')`，也可与其他标志组合，如 `'gy'`。

```ts
const a = /abc/y            // 字面量带 y
const b = new RegExp('abc', 'y') // 构造函数带 y
const c = new RegExp('abc', 'giy') // 与其他标志组合
console.log(a.sticky, b.sticky, c.sticky) // true true true
```

## 三、核心机制：与 lastIndex 的配合

`sticky` 本身不干活，它依赖 `lastIndex` 决定「从哪个位置开始」：

```ts
const re = /\d+/y

re.lastIndex = 0
console.log(re.exec('abc123def'))  // ['123']，从 0 起、在 3 处匹配成功

re.lastIndex = 3
console.log(re.exec('abc123def'))  // ['123']，恰好在 3 处开始匹配
console.log(re.lastIndex)         // 6（匹配 '123' 的结束位置）

re.lastIndex = 4                 // 4 处是 '2'，但要求「匹配点必须等于 lastIndex」
console.log(re.exec('abc123def')) // ['23']，从 4 处开始匹配剩余数字
console.log(re.lastIndex)         // 6
```

关键规则：

1. `exec` / `test` **只从 `lastIndex` 尝试匹配**，不向前向后「搜索」。
2. 匹配成功后，`lastIndex` 自动推进到本次匹配的**结束位置**。
3. 若 `lastIndex` 处匹配失败，`lastIndex` **重置为 0**。

## 四、与 global(g) 的关键区别

这是最容易混淆的点：

| 特性 | `g`（global） | `y`（sticky） |
|------|--------------|--------------|
| 匹配位置 | 从 `lastIndex` 向后**任意位置**搜索 | 必须**恰好从 `lastIndex` 开始**匹配 |
| 失败时 lastIndex | 保持原值（`exec` 返回 null 但不重置） | 重置为 0 |
| 适用场景 | 遍历字符串里所有匹配 | 顺序扫描 / 词法解析 |

```ts
// g：允许向后找
const g = /\d+/g
g.lastIndex = 4
console.log(g.exec('abc123def')) // ['23']（从 4 向后找到数字）

// y：必须恰好落在 4 处
const s = /\d+/y
s.lastIndex = 4
console.log(s.exec('abc123def')) // ['23']（4 处正好是数字起点，匹配成功）
s.lastIndex = 5
console.log(s.exec('abc123def')) // ['3']（5 处是 '3'，匹配成功）
// 若某位置起不是数字，y 直接返回 null 且 lastIndex 归 0
```

## 五、典型使用场景

1. **手写解析器 / 词法分析器（tokenizer）**：按顺序逐段消费输入，`lastIndex` 作为「读取指针」，`sticky` 保证每次只在当前指针处匹配下一段。
2. **状态机式顺序匹配**：先匹配 A，再在其后匹配 B、C……避免 `g` 可能「跳过头」的问题。
3. **流式 / 分段处理长文本**：每次只处理当前窗口，`lastIndex` 手动或自动推进。

示例（顺序解析 `用户:123`）：

```ts
const colon = /:/y
const digits = /\d+/y

const input = '用户:123'
colon.lastIndex = 2
console.log(colon.exec(input))   // ':'，匹配到冒号
digits.lastIndex = 3
console.log(digits.exec(input))  // '123'，紧接着匹配数字段
```

## 六、注意事项与兼容性

- `sticky` 必须配合 `y` 使用；不带 `y` 时即使有 `lastIndex` 也无效。
- **HBuilderX 4.72 之前 Android 端使用 PCRE**，对 `y` / `lastIndex` 的支持可能与 web 标准有差异；**4.72 起 Android 端完全遵循 ECMAScript 标准**，行为与 web 一致。跨端开发建议统一按 ECMAScript 语义编写，并做真机验证。
- 手动改动 `lastIndex` 时要清楚「匹配点必须等于 lastIndex」这一约束，否则容易拿到 `null` 而忘记 `lastIndex` 已被重置为 0。

## 一句话总结

`sticky`（`y` 标志）让正则「钉」在 `lastIndex` 上逐段顺序匹配，配合 `lastIndex` 推进/归零，是手写词法分析器与顺序解析器的理想选择；注意与 `g` 的区别（g 可向后搜索、失败不重置 lastIndex，y 必须精确匹配、失败重置为 0）。
