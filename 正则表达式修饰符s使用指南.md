# 正则表达式中的修饰符 s（dotAll 模式）

> 关联文档：[DataView 使用指南](./DataView使用指南.md) · [DataView 与 ArrayBuffer 区别及实战案例](./DataView与ArrayBuffer区别及实战案例.md) · [Uint8ClampedArray 使用指南](./Uint8ClampedArray使用指南.md)

`s` 修饰符又称 **dotAll 模式**（点号全匹配模式），作用是让元字符 `.` 能够匹配**包括换行符在内的任意字符**。

---

## 一、核心作用

| 模式 | `.` 的匹配范围 |
|------|---------------|
| **无 `s`**（默认） | 匹配任意字符，**但排除行终止符** |
| **有 `s`** | 匹配**任意字符，包括行终止符** |

**行终止符**包括 4 种：

| 字符 | 说明 |
|------|------|
| `\n` | 换行符（LF） |
| `\r` | 回车符（CR） |
| `\u2028` | 行分隔符（Line Separator） |
| `\u2029` | 段分隔符（Paragraph Separator） |

---

## 二、对比示例

```js
const text = "第一行\n第二行"

// 无 s：. 不匹配 \n，匹配失败
console.log(/第一行.第二行/.test(text))    // false

// 有 s：. 匹配 \n，匹配成功
console.log(/第一行.第二行/s.test(text))   // true
```

```js
// 提取跨行内容
const html = "<div>\n  hello\n</div>"

// 无 s：无法跨行
console.log(html.match(/<div>.*<\/div>/))    // null

// 有 s：跨行匹配
console.log(html.match(/<div>.*<\/div>/s))   // ["<div>\n  hello\n</div>"]
```

---

## 三、等价写法

`s` 修饰符本质上是把 `.` 变成 `[\s\S]`（或 `[\d\D]`、`[\w\W]`）的效果：

```js
// 以下三种写法等价
/第一行.第二行/s
/第一行[\s\S]第二行/
/第一行[\d\D]第二行/
```

> 在 `s` 出现之前（ES2018 之前），开发者普遍用 `[\s\S]` 来模拟"匹配任意字符含换行"。

---

## 四、重要边界：`s` 不影响什么

这是最容易混淆的地方：

| 元素 | `s` 是否影响 | 说明 |
|------|:-----------:|------|
| `.` | ✅ 影响 | 从"不含换行"变为"含换行" |
| `[...]` 字符类内的 `.` | ❌ 不影响 | 字符类内 `.` 本就是字面量点号 |
| `^` / `$` | ❌ 不影响 | 行首行尾由 `m` 修饰符控制 |
| `\n` 等转义 | ❌ 不影响 | 转义序列行为不变 |

```js
// 字符类内的 . 永远是字面量点号，与 s 无关
/./s.test(".")        // true（匹配任意字符）
/[.]/s.test(".")      // true（只匹配点号本身）
/[.]/s.test("a")      // false
```

---

## 五、`s` 与 `m` 的区别（常被混淆）

| 修饰符 | 名称 | 作用对象 | 效果 |
|--------|------|---------|------|
| `s` | dotAll | `.` | 让 `.` 匹配换行 |
| `m` | multiline | `^` `$` | 让 `^` `$` 匹配每行首尾 |

```js
const text = "a\nb\nc"

// m：^ 匹配每行开头
console.log(text.match(/^./gm))    // ["a", "b", "c"]

// s：. 匹配换行
console.log(text.match(/a.b/s))    // ["a\nb"]

// 可组合使用
console.log(text.match(/^.*$/gms)) // 整段匹配
```

---

## 六、典型使用场景

### 1. 跨行提取 HTML/XML 标签内容 ⭐

```js
const html = `<div class="box">
  <p>内容</p>
</div>`

// 提取 div 整体（含换行）
const m = html.match(/<div[^>]*>.*?<\/div>/s)
console.log(m?.[0])
```

### 2. 多行文本块匹配

```js
const log = `[ERROR] 2026-10-13
  错误详情：连接超时
  堆栈：...`

// 匹配从 ERROR 到下一个空行的整块
const block = log.match(/\[ERROR\].*?(?=\n\n|$)/s)
```

### 3. 解析配置文件 / Markdown

```js
const md = `# 标题
正文第一段
正文第二段`

// 提取标题后的所有正文（跨行）
const body = md.match(/# .+\n(.*)/s)?.[1]
```

### 4. 替换跨行内容

```js
const code = "function f() {\n  return 1\n}"

// 删除函数体（跨行）
const cleaned = code.replace(/function.*?\}/s, "/* removed */")
```

### 5. 提取代码块

```js
const text = "```js\nconst a = 1\n```"
const code = text.match(/```\w*\n(.*?)```/s)?.[1]
console.log(code)   // "const a = 1\n"
```

---

## 七、在 uni-app x / UTS 中的使用

UTS 的 `RegExp` 支持 `s` 修饰符，用法与 JS 一致：

```uts
const text = "第一行\n第二行"

// 字面量写法
const re1 = /第一行.第二行/s
console.log(re1.test(text))   // true

// 构造函数写法
const re2 = new RegExp("第一行.第二行", "s")
console.log(re2.test(text))   // true

// 组合多个修饰符
const re3 = new RegExp("^.*$", "gms")
```

> **兼容性**：`s` 修饰符属于 ES2018 特性，uni-app x 全平台（Android / iOS / Web / 鸿蒙）均支持。若需兼容极老环境，可用 `[\s\S]` 替代。

---

## 八、注意事项

1. **`s` 只影响 `.`**：不要误以为它能改变 `^`/`$` 行为（那是 `m`）。
2. **字符类内无效**：`[.]` 中的 `.` 永远是字面量点号。
3. **性能**：`.*` 配合 `s` 做贪婪跨行匹配时，大文本可能回溯严重，建议用**非贪婪** `.*?` 或加锚点限定范围。
4. **`s` 与 `m` 可共存**：如 `/^.*$/gms`，两者作用不同、互不冲突。
5. **替代方案**：不支持 `s` 时用 `[\s\S]`、`[\d\D]`、`[\w\W]` 均可。

---

## 九、速查总结

| 项目 | 内容 |
|------|------|
| **名称** | dotAll 模式 |
| **作用** | 让 `.` 匹配包括换行符在内的任意字符 |
| **影响对象** | 仅 `.`（不影响 `[...]`、`^`、`$`） |
| **标准** | ES2018 引入 |
| **等价写法** | `[\s\S]` / `[\d\D]` / `[\w\W]` |
| **典型场景** | 跨行 HTML 提取、多行文本块匹配、代码块解析 |
| **易混淆** | `m` 管 `^`/`$`，`s` 管 `.` |

> **一句话记忆**：**`s` = dotAll，让点号 `.` 也能"吃掉"换行符；`m` = multiline，让 `^`/`$` 认每一行。**
