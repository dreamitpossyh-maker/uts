# UTS RegExp `exec` 使用指南

> 文档来源：<https://doc.dcloud.net.cn/uni-app-x/uts/buildin-object-api/regexp.html>
> 适用环境：uni-app x / UTS（编译为 Kotlin `io.dcloud.uts.UTSRegExp`）

---

## 一、什么是 `exec`

`exec` 是 `RegExp` 实例方法，用于在**目标字符串中执行一次匹配搜索**，并返回匹配结果。

- 它是正则表达式最核心的匹配方法，`test`、`String.prototype.match` 等能力都建立在它之上。
- 与 `test` 只返回布尔值不同，`exec` 会返回**完整的匹配信息**（匹配文本、捕获组、位置等）。
- 当正则带有 `g`（global）或 `y`（sticky）标志时，`exec` 会**读写 `lastIndex`**，从而支持连续多次匹配。

---

## 二、签名与参数 / 返回值

### 签名

```uts
exec(string: string): RegExpExecArray | null
```

### 参数

| 参数     | 类型     | 必填 | 说明               |
| -------- | -------- | ---- | ------------------ |
| `string` | `string` | 是   | 要执行匹配的字符串 |

### 返回值

| 情况     | 返回值             | 说明                                                                 |
| -------- | ------------------ | -------------------------------------------------------------------- |
| 匹配失败 | `null`             | 同时将 `lastIndex` 重置为 `0`（当带 `g`/`y` 时）                     |
| 匹配成功 | `RegExpExecArray`  | 一个数组，并附带 `index`、`input` 等属性；带 `g`/`y` 时更新 `lastIndex` |

`RegExpExecArray` 的结构：

| 成员        | 说明                                             |
| ----------- | ------------------------------------------------ |
| `[0]`       | 完整匹配的字符串                                 |
| `[1]`…`[n]` | 各捕获组（括号）匹配到的内容，未匹配到为 `undefined` |
| `index`     | 匹配在原始字符串中的起始下标                     |
| `input`     | 被搜索的原始字符串                               |

### 官方示例

```uts
const pattern1 = new RegExp('hello')
const result1 = pattern1.exec('hello world')!
console.log(result1[0]) // 'hello'
console.log(result1.index) // 0
```

---

## 三、核心机制

1. **单次匹配**：每次调用只返回**第一个**匹配结果（除非配合 `g`/`y` 循环调用）。
2. **`lastIndex` 联动**：
   - 带 `g` 或 `y` 时，`exec` 从 `lastIndex` 处开始搜索，成功后把 `lastIndex` 更新为**匹配结束位置**。
   - 匹配失败时，`lastIndex` 被重置为 `0`。
   - 不带 `g`/`y` 时，`exec` 始终从 `0` 开始，且**不修改** `lastIndex`。
3. **捕获组**：返回数组除 `[0]` 外，依次是各捕获组内容；命名捕获组可通过 `groups` 访问。
4. **空匹配风险**：当正则能匹配空字符串（如 `/a*/g`）时，`lastIndex` 可能不前进，导致死循环，需手动 `lastIndex++`。

---

## 四、与 `test` 的区别

| 对比项         | `exec`                          | `test`            |
| -------------- | ------------------------------- | ----------------- |
| 返回值         | `RegExpExecArray \| null`       | `boolean`         |
| 是否含捕获组   | 是                              | 否                |
| 是否含位置信息 | 是（`index` / `input`）         | 否                |
| 性能           | 相对较重（构造结果数组）        | 相对较轻          |
| 是否修改 `lastIndex` | 带 `g`/`y` 时修改         | 带 `g`/`y` 时修改 |
| 典型用途       | 提取内容、解析、遍历所有匹配    | 判断是否存在、校验 |

> 关系：`regexp.test(str)` 等价于 `regexp.exec(str) !== null`（但注意两者都会修改 `lastIndex`）。

---

## 五、典型使用场景 + 使用案例

### 场景概览

- 提取字符串中的结构化信息（日期、数字、URL 等）。
- 遍历字符串中**所有**匹配项。
- 配合 `y` 做**逐段解析**（词法分析、协议解析）。
- 提取捕获组做数据加工。

---

### 案例 A1：提取单个匹配 + 捕获组

```uts
const re = /(\d{4})-(\d{2})-(\d{2})/
const str = '日期：2026-10-19'
const m = re.exec(str)

if (m != null) {
  console.log(m[0]) // '2026-10-19'  完整匹配
  console.log(m[1]) // '2026'        年
  console.log(m[2]) // '10'          月
  console.log(m[3]) // '19'          日
  console.log(m.index) // 3          起始下标
  console.log(m.input) // '日期：2026-10-19'
}
```

**要点**：不带 `g` 时每次从 0 开始，适合“只取第一个”的场景。

---

### 案例 A2：用 `g` 遍历所有匹配

```uts
const re = /\d+/g
const str = 'a1 b22 c333'
let m: RegExpExecArray | null
let total = 0

while ((m = re.exec(str)) != null) {
  console.log(m[0]) // '1' -> '22' -> '333'
  total += parseInt(m[0])
}
console.log(total) // 356
```

**要点**：`g` 标志让 `lastIndex` 自动前进，循环直到返回 `null`（此时 `lastIndex` 归零）。

---

### 案例 A3：用 `y`（sticky）做逐段解析

```uts
const re = /\d+/y
const str = '123abc'

console.log(re.exec(str)![0]) // '123'，lastIndex 变为 3
console.log(re.lastIndex)     // 3

// 从 lastIndex=3 处继续，'abc' 不是数字，匹配失败
console.log(re.exec(str))     // null
console.log(re.lastIndex)     // 0（失败后重置）
```

**要点**：`y` 要求匹配**必须从 `lastIndex` 处开始**，非常适合按顺序解析固定格式的输入。

---

### 案例 A4：空匹配防死循环

```uts
const re = /a*/g
const str = 'baa'
let m: RegExpExecArray | null

while ((m = re.exec(str)) != null) {
  console.log(`match='${m[0]}' index=${m.index}`)
  // 空匹配时 lastIndex 不前进，手动 +1 防止死循环
  if (m[0] == '') {
    re.lastIndex++
  }
}
// 输出：
// match='' index=0
// match='aa' index=1
// match='' index=3
```

**要点**：任何可能匹配空串的 `g` 正则，循环时都要处理空匹配。

---

## 六、注意事项与兼容性

1. **`lastIndex` 副作用**：带 `g`/`y` 的 `exec` 会修改 `lastIndex`，复用同一个正则对象时要注意状态残留；必要时手动 `re.lastIndex = 0` 重置。
2. **空匹配死循环**：见案例 A4，务必处理 `m[0] == ''` 的情况。
3. **非空断言**：UTS 中 `exec` 返回可空类型，使用 `!` 或判空后再访问，避免空指针。
4. **引擎变更**：HBuilderX **4.72 之前** Android 端使用 PCRE 引擎；**4.72 及之后**完全遵循 ECMAScript 标准，行为与 Web 一致。
5. **编译目标**：`RegExp` 在 UTS 中编译为 Kotlin 的 `io.dcloud.uts.UTSRegExp`。
