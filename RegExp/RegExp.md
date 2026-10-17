# UTS RegExp（正则表达式）使用总结

> 文档来源：https://doc.dcloud.net.cn/uni-app-x/uts/buildin-object-api/regexp.html
> 适用：uni-app x / UTS（uni-app TypeScript）

---

## 一、重要调整（引擎差异）

UTS 中 RegExp 的底层实现发生过一次**重大变更**，直接影响正则语法行为：

| HBuilderX 版本 | Android 端正则引擎 | 行为 |
| --- | --- | --- |
| **4.72 之前** | PCRE（Perl 兼容正则） | 支持 PCRE 语法，与 Web 标准存在差异 |
| **4.72 及之后** | **完全遵循 ECMAScript 标准** | 与 Web（浏览器）行为一致 |

> ⚠️ 升级到 4.72+ 后，原先依赖 PCRE 特性的正则可能失效，需要按 ECMAScript 标准改写。

**示例：**

```uts
const FLOAT_REGEX = /(\d+(\.\d+)?)px/g
// 4.72 之前（PCRE）与 4.72 之后（ECMAScript）行为不同
console.log('10px 20px'.replace(FLOAT_REGEX, '${0}'))
```

---

## 二、构造函数

RegExp 提供两种构造方式（重载）：

```uts
// 方式一：传入正则字面量或字符串
new(pattern: RegExp | string)

// 方式二：传入字符串模式 + 修饰符
new(pattern: string, flags?: string)
```

**示例：**

```uts
const re1 = new RegExp('abc')
const re2 = new RegExp('abc', 'gi')
const re3 = new RegExp(/abc/i)
```

### 修饰符（flags）说明

| 修饰符 | 名称 | 含义 |
| --- | --- | --- |
| `g` | global | 全局匹配，查找所有匹配项 |
| `i` | ignoreCase | 忽略大小写 |
| `m` | multiline | 多行模式，`^`/`$` 匹配每行首尾 |
| `s` | dotAll | `.` 可匹配换行符 |
| `u` | unicode | Unicode 模式 |
| `y` | sticky | 粘性匹配，从 `lastIndex` 处开始 |
| `d` | hasIndices | 生成匹配索引信息 |

---

## 三、实例属性

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| `flags` | string | 返回当前正则的修饰符字符串 |
| `dotAll` | boolean | 是否启用 `s` 修饰符 |
| `hasIndices` | boolean | 是否启用 `d` 修饰符 |
| `sticky` | boolean | 是否启用 `y` 修饰符 |
| `unicode` | boolean | 是否启用 `u` 修饰符 |
| `source` | string | 正则表达式的源文本（不含修饰符） |
| `global` | boolean | 是否启用 `g` 修饰符 |
| `ignoreCase` | boolean | 是否启用 `i` 修饰符 |
| `multiline` | boolean | 是否启用 `m` 修饰符 |
| `lastIndex` | number | 下一次匹配的起始位置（配合 `g`/`y` 使用） |

**示例：**

```uts
const re = /abc/gi
console.log(re.flags)      // "gi"
console.log(re.source)     // "abc"
console.log(re.global)     // true
console.log(re.ignoreCase) // true
console.log(re.lastIndex)  // 0
```

---

## 四、实例方法

### 1. `exec(string)`

在字符串中执行一次匹配，返回匹配结果数组或 `null`。

```uts
exec(string: string): RegExpExecArray | null
```

- 返回的 `RegExpExecArray` 包含匹配文本、捕获组、`index`、`input` 等信息。
- 若带 `g`/`y` 修饰符，会更新 `lastIndex`，可循环调用获取全部匹配。

**示例：**

```uts
const re = /(\d+)/g
let result = re.exec('a1b22c333')
console.log(result) // 匹配 "1"，index = 1
```

### 2. `test(string)`

测试字符串是否匹配，返回布尔值。

```uts
test(string: string): boolean
```

- 带 `g` 修饰符时，每次调用会从 `lastIndex` 继续，可能影响连续调用结果。

**示例：**

```uts
const re = /abc/
console.log(re.test('xxabcxx')) // true
console.log(re.test('xxxxx'))   // false
```

---

## 五、兼容性汇总

| 平台 | 支持情况 |
| --- | --- |
| Android | ✅ 支持（4.72+ 遵循 ECMAScript） |
| iOS | ✅ 支持（部分属性/修饰符不支持） |
| HarmonyOS | ✅ 支持（部分属性/修饰符不支持） |
| Web | ✅ 支持 |

> ⚠️ 注意：`dotAll`、`hasIndices`、`sticky`、`unicode` 等属性/修饰符在 **iOS 与 HarmonyOS** 上暂不支持，跨端使用时需谨慎。

---

## 六、Bug & Tips

- UTS 中 RegExp 会编译为 Kotlin 的 `io.dcloud.uts.UTSRegExp`。
- 已知问题反馈地址：https://issues.dcloud.net.cn/?mid=uts.buildInObject.RegExp
- 跨端开发时，优先使用 ECMAScript 标准语法，避免依赖平台特有行为。

---

## 七、方法速查表

| 成员 | 类型 | 说明 |
| --- | --- | --- |
| `new(pattern)` | 构造函数 | 由正则或字符串创建 |
| `new(pattern, flags)` | 构造函数 | 由字符串 + 修饰符创建 |
| `exec(string)` | 方法 | 执行匹配，返回结果数组或 null |
| `test(string)` | 方法 | 测试是否匹配，返回 boolean |
| `flags` | 属性 | 修饰符字符串 |
| `source` | 属性 | 源文本 |
| `global` / `ignoreCase` / `multiline` | 属性 | 对应修饰符状态 |
| `dotAll` / `hasIndices` / `sticky` / `unicode` | 属性 | 对应修饰符状态（部分平台不支持） |
| `lastIndex` | 属性 | 下次匹配起始位置 |
