# UTS RegExp `hasIndices` 使用指南

> 适用环境：uni-app x / UTS（uni-app TypeScript）
> 对应标准：ECMAScript 2022 新增的 `d` 修饰符（Indices / Match Indices）

---

## 一、什么是 `hasIndices`

`hasIndices` 是 `RegExp` 实例的一个**只读布尔属性**，用于判断当前正则对象是否启用了 `d` 修饰符（Indices 标志）。

- 当正则使用了 `d` 修饰符时，`hasIndices` 为 `true`；
- 否则为 `false`。

它本身不参与匹配逻辑，只是一个**状态查询属性**，用来告诉调用方：本次匹配结果中是否会附带 `indices`（匹配位置索引）信息。

```ts
const re1 = /foo/d
console.log(re1.hasIndices) // true

const re2 = /foo/
console.log(re2.hasIndices) // false
```

---

## 二、如何启用 `hasIndices`

`hasIndices` 由 `d` 修饰符决定，启用方式有两种：

### 1. 字面量方式

```ts
const re = /foo(\d+)/d
console.log(re.hasIndices) // true
```

### 2. 构造函数方式

```ts
const re = new RegExp('foo(\\d+)', 'd')
console.log(re.hasIndices) // true
```

> 注意：`d` 修饰符可以与其他修饰符组合使用，例如 `g`、`i`、`m`、`s`、`u`、`y`。
> 例如：`/foo/gi` 与 `/foo/gid` 都是合法的。

---

## 三、核心：`indices` 结果结构

启用 `d` 修饰符后，`exec()` / `match()` 返回的结果对象会**额外包含一个 `indices` 属性**。

`indices` 是一个数组，结构与匹配结果数组一一对应：

- `indices[0]`：整个匹配的 `[起始索引, 结束索引)`（左闭右开）；
- `indices[1]`、`indices[2]`……：对应第 1、2……个捕获组的位置；
- 命名捕获组可通过 `indices.groups` 访问。

```ts
const re = /foo(\d+)/d
const m = re.exec('xxfoo123yy')

console.log(m![0])          // 'foo123'
console.log(m!.indices)     // [[2, 8], [5, 8]]
// 整个匹配：索引 2 到 8（不含 8）
// 捕获组 \d+：索引 5 到 8（不含 8）
```

### 命名捕获组

```ts
const re = /(?<year>\d{4})-(?<month>\d{2})/d
const m = re.exec('2026-10')

console.log(m!.indices.groups!.year)  // [0, 4]
console.log(m!.indices.groups!.month) // [5, 7]
```

### 未匹配的捕获组

如果某个捕获组没有参与匹配，则它在 `indices` 中对应位置为 `undefined`。

```ts
const re = /(a)?(b)/d
const m = re.exec('b')

console.log(m!.indices) // [ [0, 1], undefined, [0, 1] ]
// 第 1 个捕获组 (a)? 未匹配 → undefined
```

---

## 四、典型使用场景

### 1. 语法高亮 / 编辑器着色

需要知道每个 token 在原文中的**精确起止位置**，才能对指定区间上色。

```ts
const re = /(\w+)(\s*=\s*)(\d+)/d
const code = 'count = 42'
const m = re.exec(code)

if (m) {
  const [nameStart, nameEnd] = m.indices[1]
  const [valStart, valEnd] = m.indices[3]
  // 可分别对变量名区间 [nameStart, nameEnd) 和数值区间 [valStart, valEnd) 着色
}
```

### 2. 精确提取子串（无需再计算偏移）

传统做法需要用 `m.index + 前缀长度` 手动推算，`indices` 直接给出答案。

```ts
const re = /@(\w+)/d
const text = 'hello @alice and @bob'
let m: RegExpExecArray | null
const reG = /@(\w+)/dg

while ((m = reG.exec(text)) !== null) {
  const [s, e] = m.indices[1]
  console.log(text.substring(s, e)) // 'alice'、'bob'
}
```

### 3. 位置感知的替换

在替换时保留原始位置信息，用于日志、审计或增量更新。

```ts
const re = /(\d+)/d
const m = re.exec('id=1001')
if (m) {
  const [s, e] = m.indices[1]
  console.log(`数字位于 [${s}, ${e})`) // 数字位于 [3, 7)
}
```

### 4. 简易词法分析器（Tokenizer）

逐个 token 记录类型与位置，便于后续构建 AST 或做错误定位。

```ts
const tokenRe = /(?<num>\d+)|(?<id>[a-zA-Z_]\w*)|(?<op>[+\-*/])/dg
const src = 'a + 12 * b'
let t: RegExpExecArray | null

while ((t = tokenRe.exec(src)) !== null) {
  const [s, e] = t.indices[0]
  console.log(`token "${src.substring(s, e)}" @ [${s}, ${e})`)
}
```

### 5. 配合 `matchAll` 批量获取位置

```ts
const re = /(\w+)/dg
const text = 'foo bar baz'

for (const m of text.matchAll(re)) {
  const [s, e] = m.indices[1]
  console.log(`${m[1]} -> [${s}, ${e})`)
}
```

---

## 五、注意事项与兼容性

| 项目 | 说明 |
| --- | --- |
| 属性类型 | 只读布尔值，不可手动赋值 |
| 依赖修饰符 | 由 `d` 修饰符决定，无 `d` 则为 `false` |
| 索引区间 | 左闭右开 `[start, end)`，`end` 不包含在匹配内 |
| 未匹配组 | 对应 `indices` 项为 `undefined` |
| 命名组 | 通过 `indices.groups.<name>` 访问 |
| 平台兼容 | 文档标注：`hasIndices`（及 `dotAll`、`sticky`、`unicode`）在 **iOS 与 HarmonyOS 不支持**，仅 **Android / Web** 支持 |
| 引擎差异 | HBuilderX 4.72 之前 Android 端使用 PCRE；4.72 及之后完全遵循 ECMAScript 标准（与 Web 一致） |
| 编译产物 | RegExp 编译为 Kotlin `io.dcloud.uts.UTSRegExp` |

> ⚠️ 跨端开发时，若目标平台包含 iOS / HarmonyOS，请勿依赖 `hasIndices` / `indices`，应改用 `m.index` 与捕获组长度手动计算位置。

---

## 六、一句话总结

`hasIndices` 是 `d` 修饰符的开关状态查询属性；启用后，匹配结果会附带 `indices`，以 `[start, end)` 形式精确给出**整个匹配及每个捕获组在原文中的位置**，非常适合语法高亮、精确提取、位置感知替换与词法分析等场景——但需注意 iOS / HarmonyOS 平台暂不支持。
