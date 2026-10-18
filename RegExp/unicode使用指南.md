# UTS RegExp unicode 使用指南

> `unicode` 是 `RegExp` 的只读属性：当正则带有 **`u`（unicode）标志** 时为 `true`，否则为 `false`。
> 它让正则引擎以「Unicode 感知」的方式处理字符——能把非 ASCII 字符（中文、emoji、组合字符等）当作**一个完整的字符单元**来识别，而不是逐字节拆解。

## 一、什么是 unicode

`unicode`（Unicode 模式）描述的是「按完整 Unicode 码点 / 文字单元处理字符」这一行为：

- 带 `u` 标志时，引擎把非 BMP 字符（emoji、部分 CJK 扩展字符等）当作**单一字符单元**，而不是 UTF-16 代理对（两个 16 位码元）。
- 仅 `u` 模式下才支持 `\u{...}` 花括号码点语法和 `\p{...}` 通用属性类。
- `^` / `$` / `{n}` 数量界定符按 Unicode 文字单元计算。

```ts
console.log(/a/u.unicode)                  // true
console.log(/a/.unicode)                   // false
console.log(new RegExp('a', 'u').unicode)   // true
```

## 二、如何启用

- 字面量写法：斜杠后加 `u`，例如 `/汉字/u`。
- 构造函数写法：`new RegExp('汉字', 'u')`，也可与其他标志组合，如 `'gu'`。

```ts
const a = /汉字/u                          // 字面量带 u
const b = new RegExp('汉字', 'u')          // 构造函数带 u
const c = new RegExp('汉字', 'gu')        // 与其他标志组合
console.log(a.unicode, b.unicode, c.unicode) // true true true
```

## 三、核心机制：让 \u{...} 与 \p{...} 正确工作

开启 `u` 后，最直接的体现是：

1. **`\u{...}` 花括号形式支持任意码点**：不只是 BMP 内的 `\uXXXX`，还能写 `\u{1F600}` 这类超出 BMP 的 emoji 码点（必须带 `u`）。
2. **`\p{...}` 通用属性类可用**：`u` 模式是 `\p{L}`（字母）、`\p{N}`（数字）、`\p{Script=Han}`（汉字）、`\p{Extended_Pictographic}`（表情符号）等属性类可用的前提。
3. **`^` / `$` 按文字单元边界工作**：组合字符（如 `e` + 组合重音）能被当作整体边界。
4. **更严格的校验**：`u` 模式对不完整的代理对、非法转义序列、未闭合字符类会报错，而非宽松容忍。

```ts
// 匹配 emoji（码点 U+1F600，超出 BMP，必须用 \u{...} 且带 u 标志）
const smiley = /\u{1F600}/u
console.log(smiley.test('😀'))   // true

// 用属性类匹配表情符号（仅 u 模式支持 \p{}）
const astral = /\p{Extended_Pictographic}/u
console.log(astral.test('😀'))   // true
```

## 四、与「不带 u」的关键区别

| 特性 | 不带 `u` | 带 `u`（unicode） |
|------|---------|------------------|
| `\u{...}` 花括号码点 | 不支持，仅 `\uXXXX`（BMP） | 支持任意码点 `\u{1F600}` |
| 非 BMP 字符 | 按代理对（surrogate pair）拆成两个 16 位单位处理 | 当作完整单一字符 |
| `^` / `$` / `{n}` 边界 | 按 UTF-16 码元计算 | 按 Unicode 文字单元计算 |
| 非法序列 | 宽松容忍 | 更严格校验 |

```ts
// 非 ASCII 匹配示例
const smile = /😀/u
console.log(smile.test('hi 😀'))   // true，u 模式下 emoji 作为单一字符单元参与匹配
```

## 五、典型使用场景

1. **中文 / 国际化文本处理**：匹配汉字、日文、韩文等 CJK 字符，`u` 模式保证码点与字素边界正确。
2. **emoji 匹配与过滤**：用 `\u{1F600}` 或 `\p{Extended_Pictographic}` 精确识别表情符号，必须带 `u`。
3. **校验用户输入编码合法性**：`u` 模式对不完整的代理对、非法转义会更严格地报错，帮助尽早发现问题。
4. **配合 `\p{...}` 通用属性类做语言 / 字符类别判定**：如 `\p{Script=Han}` 判定汉字。

示例（匹配中文用户名，仅 CJK 汉字且长度 2–6）：

```ts
const name = /^\p{Script=Han}{2,6}$/u
console.log(name.test('张三'))    // true
console.log(name.test('王小明'))  // true
console.log(name.test('AB'))      // false
```

## 六、注意事项与兼容性

- 使用 `\u{...}` 与 `\p{...}` 必须带 `u` 标志，否则是语法错误。
- `u` 模式对非法的转义序列和未闭合的字符类更严格，某些在宽松模式下「碰巧能跑」的正则会报错，迁移时要留意。
- **HBuilderX 4.72 之前 Android 端使用 PCRE**，对 `u` 模式、`\p{...}` 属性类的支持有限且与 web 标准不一致；**4.72 起 Android 端完全遵循 ECMAScript 标准**，行为与 web 一致。涉及 emoji / `\p{Script=Han}` 等高级特性的跨端逻辑，建议按 ECMAScript 语义编写并做真机验证。

## 一句话总结

`unicode`（`u` 标志）让正则以「完整 Unicode 码点 / 文字单元」方式工作，是 `\u{...}`、`\p{...}` 属性类（汉字、emoji 等）可用的前提；跨端涉及 CJK / emoji 的高级匹配务必带 `u` 并做真机验证。
