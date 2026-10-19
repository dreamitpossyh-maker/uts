# UTS RegExp 实例方法 `exec` 使用指南

> 参考文档：<https://doc.dcloud.net.cn/uni-app-x/uts/buildin-object-api/regexp.html>
> 适用环境：uni-app x / UTS（编译为 Kotlin `io.dcloud.uts.UTSRegExp`）

---

## 一、什么是 `exec`

`exec` 是 `RegExp` 实例上用于**执行一次匹配**的方法。它会在目标字符串中查找符合正则的**第一个匹配项**，并把匹配结果以数组形式返回；如果没有匹配到，则返回 `null`。

与 `test` 只返回布尔值不同，`exec` 会返回**完整的匹配信息**，包括：

- 匹配到的完整子串（`array[0]`）
- 各个捕获组的内容（`array[1]`、`array[2]` …）
- 匹配在原字符串中的起始下标（`array.index`）
- 被匹配的原始字符串（`array.input`）

因此，当你**不仅想知道"有没有匹配"，还想知道"匹配到了什么、在哪里"**时，就应该使用 `exec`。

---

## 二、签名与参数 / 返回值

### 签名

```ts
exec(string: string): RegExpExecArray | null
```

### 参数

| 参数名   | 类型     | 是否必填 | 说明               |
| -------- | -------- | -------- | ------------------ |
| `string` | `string` | 是       | 要执行匹配的字符串 |

### 返回值

| 情况     | 返回值              | 说明                                                                 |
| -------- | ------------------- | -------------------------------------------------------------------- |
| 匹配失败 | `null`              | 同时会把 `lastIndex` 重置为 `0`（当正则带 `g` 或 `y` 时）            |
| 匹配成功 | `RegExpExecArray`   | 一个数组，`array[0]` 为完整匹配，`array[1..]` 为捕获组，并附带 `index`、`input` 属性；带 `g`/`y` 时会更新 `lastIndex` |

### 官方示例

```ts
const pattern1 = new RegExp('hello')
const result1 = pattern1.exec('hello world')!
console.log(result1[0]) // 'hello'
console.log(result1.index) // 0
```

---

## 三、核心机制

1. **单次匹配**：每次调用 `exec` 只返回**一个**匹配结果（第一个匹配项），不会一次性返回全部。
2. **`lastIndex` 联动**：
   - 当正则**带 `g`（global）或 `y`（sticky）标志**时，`exec` 会从 `lastIndex` 处开始查找，匹配成功后把 `lastIndex` 更新为**本次匹配结束的位置**，从而支持"循环取出所有匹配"。
   - 当正则**不带 `g`/`y`** 时，`exec` 每次都从字符串开头查找，`lastIndex` 始终为 `0`。
3. **失败重置**：匹配失败返回 `null`，并把 `lastIndex` 重置为 `0`。
4. **捕获组**：若正则包含捕获组 `(...)`，则 `array[1]`、`array[2]` … 依次对应各捕获组；未匹配到的组为 `undefined`。
5. **命名捕获组**：若使用 `(?<name>...)`，可通过 `array.groups.name` 访问。

---

## 四、与 `test` 的区别

| 对比项         | `exec`                          | `test`            |
| -------------- | ------------------------------- | ----------------- |
| 返回值         | `RegExpExecArray \| null`       | `boolean`         |
| 是否返回捕获组 | ✅ 是                           | ❌ 否             |
| 是否返回下标   | ✅ 是（`index`）                | ❌ 否             |
| 是否返回原串   | ✅ 是（`input`）                | ❌ 否             |
| 性能           | 相对较重（构造数组）            | 相对较轻          |
| `lastIndex`    | 带 `g`/`y` 时会被更新           | 带 `g`/`y` 时会被更新 |
| 典型用途       | 提取内容、循环遍历所有匹配      | 判断是否存在、表单校验 |

> 结论：**只需要"是否匹配"用 `test`；需要"匹配内容"用 `exec`。**

---

## 五、典型使用场景 + 使用案例

### 场景 1：提取单个匹配及其捕获组

**案例 A1 —— 从文本中解析日期**

```ts
const re = /(\d{4})-(\d{2})-(\d{2})/
const result = re.exec('日期：2026-10-19')

if (result != null) {
  console.log(result[0]) // '2026-10-19'  完整匹配
  console.log(result[1]) // '2026'         年
  console.log(result[2]) // '10'           月
  console.log(result[3]) // '19'           日
  console.log(result.index) // 3           起始下标
  console.log(result.input) // '日期：2026-10-19'
} else {
  console.log('未匹配到日期')
}
```

---

### 场景 2：配合 `g` 标志循环取出所有匹配

**案例 A2 —— 提取字符串中所有数字**

```ts
const re = /\d+/g
const str = 'a1 b22 c333'
let m: RegExpExecArray | null

while ((m = re.exec(str)) != null) {
  console.log(m[0], m.index)
}
// 输出：
// 1 1
// 22 4
// 333 8
```

> 说明：因为带 `g`，每次 `exec` 都从上次结束位置继续，直到返回 `null` 结束循环。

---

### 场景 3：配合 `y`（sticky）做"从指定位置严格连续匹配"

**案例 A3 —— 从字符串开头连续解析数字**

```ts
const re = /\d+/y
const str = '123abc'

console.log(re.exec(str)![0]) // '123'
console.log(re.lastIndex)     // 3
console.log(re.exec(str))     // null（因为位置 3 处是 'a'，不满足 sticky 要求）
```

> 说明：`y` 要求匹配必须**紧接在 `lastIndex` 处**开始，适合做词法解析、逐段扫描。

---

### 场景 4：处理"空匹配"避免死循环

**案例 A4 —— 空匹配时手动推进 `lastIndex`**

```ts
const re = /a*/g
const str = 'baa'
let m: RegExpExecArray | null

while ((m = re.exec(str)) != null) {
  console.log(`匹配 "${m[0]}" @ ${m.index}`)
  // 若本次为空匹配（m[0] === ''），必须手动推进，否则会死循环
  if (m[0] == '') {
    re.lastIndex++
  }
}
```

> 说明：像 `a*`、`.*?` 这类可以匹配空串的正则，在 `g` 模式下可能返回空匹配且 `lastIndex` 不前进，必须手动 `lastIndex++` 防止无限循环。

---

## 六、注意事项与兼容性

1. **`lastIndex` 副作用**：带 `g`/`y` 的正则对象是**有状态**的，多次调用 `exec` 会相互影响。若想每次从头匹配，请重置 `re.lastIndex = 0`，或使用不带 `g`/`y` 的正则。
2. **空匹配死循环**：见案例 A4，务必在空匹配时手动推进 `lastIndex`。
3. **`exec` 与 `test` 等价关系**：`re.test(str)` 等价于 `re.exec(str) != null`（但 `test` 不返回匹配内容）。
4. **非空断言**：UTS 中 `exec` 返回可空类型，使用 `!` 或先判空再访问，避免空指针。
5. **引擎差异**：HBuilderX **4.72 之前** Android 端使用 PCRE 引擎；**4.72 及之后**完全遵循 ECMAScript 标准（与 Web 行为一致）。跨端行为请以 4.72+ 为准。
6. **编译目标**：`RegExp` 在 UTS 中编译为 Kotlin 的 `io.dcloud.uts.UTSRegExp`。
