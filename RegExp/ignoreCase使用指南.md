# UTS RegExp `ignoreCase` 使用指南

> 适用环境：uni-app x / UTS（uni-app TypeScript）
> 对应文档：https://doc.dcloud.net.cn/uni-app-x/uts/buildin-object-api/regexp.html

---

## 一、什么是 `ignoreCase`

`ignoreCase` 是 `RegExp` 实例的一个**只读布尔属性**，用于表示当前正则对象是否启用了**忽略大小写**匹配。

- 当正则表达式带有 `i` 修饰符时，`ignoreCase` 返回 `true`；
- 未带 `i` 修饰符时，返回 `false`。

它本质上是对 `i` 修饰符的**运行时读取入口**，让你可以在代码中动态判断一个正则对象是否处于忽略大小写模式。

```uts
const re1 = /abc/i
console.log(re1.ignoreCase) // true

const re2 = /abc/
console.log(re2.ignoreCase) // false
```

---

## 二、如何启用

`ignoreCase` 本身**不能通过赋值修改**，它由创建正则时是否传入 `i` 修饰符决定。启用方式有两种：

### 1. 字面量方式

```uts
const re = /hello/i
console.log(re.ignoreCase) // true
```

### 2. 构造函数方式

```uts
const re = new RegExp('hello', 'i')
console.log(re.ignoreCase) // true
```

> 注意：`ignoreCase` 是只读属性，`re.ignoreCase = false` 不会生效（严格模式下会报错，非严格模式下静默失败）。

---

## 三、核心机制

启用 `i` 修饰符后，正则引擎在匹配时会进行**大小写折叠（case folding）**，即把待匹配字符与模式字符都按大小写不敏感的方式比较。

```uts
const re = /abc/i

console.log(re.test('ABC')) // true
console.log(re.test('AbC')) // true
console.log(re.test('abc')) // true
console.log(re.test('aBc')) // true
```

`ignoreCase` 属性只是反映这一状态，它不会改变匹配行为本身。

---

## 四、与不带 `i` 的区别

| 场景 | `/abc/`（`ignoreCase = false`） | `/abc/i`（`ignoreCase = true`） |
| --- | --- | --- |
| `test('abc')` | `true` | `true` |
| `test('ABC')` | `false` | `true` |
| `test('AbC')` | `false` | `true` |
| 匹配速度 | 略快（无需折叠） | 略慢（需大小写折叠） |
| 适用场景 | 严格区分大小写 | 用户输入、关键字搜索等 |

```uts
console.log(/abc/.test('ABC'))  // false
console.log(/abc/i.test('ABC')) // true
```

---

## 五、典型使用场景

### 1. 用户输入匹配

用户输入的大小写往往不可控，忽略大小写能提升容错性。

```uts
const keyword = 'UniApp'
const re = new RegExp(keyword, 'i')
console.log(re.test('uniapp')) // true
console.log(re.test('UNIAPP')) // true
```

### 2. 关键字搜索 / 高亮

在文本中查找关键字并高亮时，通常需要忽略大小写。

```uts
const text = 'Welcome to UniApp X'
const re = /uniapp/gi
const result = text.replace(re, '**$&**')
console.log(result) // Welcome to **UniApp** X
```

### 3. 大小写不敏感的格式校验

例如校验文件扩展名、协议头等。

```uts
const re = /\.(png|jpg|jpeg)$/i
console.log(re.test('photo.PNG')) // true
console.log(re.test('photo.Jpg')) // true
```

### 4. 配置 / 枚举解析

从配置中读取的字符串大小写可能不统一，忽略大小写可避免解析失败。

```uts
const mode = 'PRODUCTION'
const re = /^production$/i
console.log(re.test(mode)) // true
```

### 5. 通过 `ignoreCase` 属性动态分支

当需要根据正则是否忽略大小写做不同处理时，可直接读取该属性。

```uts
function describe(re: RegExp): string {
  return re.ignoreCase ? '忽略大小写' : '区分大小写'
}

console.log(describe(/abc/i)) // 忽略大小写
console.log(describe(/abc/))  // 区分大小写
```

---

## 六、注意事项与兼容性

1. **只读属性**：`ignoreCase` 无法通过赋值改变，只能通过创建正则时的 `i` 修饰符决定。

2. **引擎变更**：HBuilderX 4.72 之前，Android 端正则使用 PCRE 引擎；4.72 及之后完全遵循 ECMAScript 标准，行为与 Web 端一致。跨版本时需注意大小写折叠的细节差异。

3. **平台兼容性**：iOS / HarmonyOS 等平台的正则支持情况请参考 UTS 兼容性表，`ignoreCase` 作为标准属性在各平台均受支持。

4. **性能提示**：启用 `i` 会带来大小写折叠开销，在超高频匹配场景下，若确定数据大小写统一，可考虑不使用 `i` 以提升性能。

5. **与 `unicode` 组合**：在涉及 Unicode 字符（如某些特殊字母）时，`i` 的大小写折叠行为可能受 `u` 修饰符影响，建议同时评估 `u` 的使用。

```uts
// 组合使用示例
const re = /straße/iu
console.log(re.ignoreCase) // true
console.log(re.unicode)    // true
```

---

## 七、小结

- `ignoreCase` 是 `RegExp` 实例的**只读布尔属性**，反映是否启用 `i` 修饰符。
- 启用方式：`/pattern/i` 或 `new RegExp('pattern', 'i')`。
- 作用：让匹配忽略大小写，提升用户输入、搜索、校验等场景的容错性。
- 注意：只读、有性能开销、跨引擎版本需关注行为差异。
