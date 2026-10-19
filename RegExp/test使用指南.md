# UTS RegExp `test` 使用指南

> 文档来源：<https://doc.dcloud.net.cn/uni-app-x/uts/buildin-object-api/regexp.html>
> 适用环境：uni-app x / UTS（编译为 Kotlin `io.dcloud.uts.UTSRegExp`）

---

## 一、什么是 `test`

`test` 是 `RegExp` 实例方法，用于**判断目标字符串中是否存在匹配**，返回布尔值。

- 它是最轻量的匹配方法，只关心“有没有匹配”，不返回匹配内容。
- 当正则带有 `g`（global）或 `y`（sticky）标志时，`test` 会**读写 `lastIndex`**，因此连续调用会从上次位置继续。
- 常用于表单校验、存在性判断、条件分支等场景。

---

## 二、签名与参数 / 返回值

### 签名

```uts
test(string: string): boolean
```

### 参数

| 参数     | 类型     | 必填 | 说明               |
| -------- | -------- | ---- | ------------------ |
| `string` | `string` | 是   | 要执行匹配的字符串 |

### 返回值

| 情况     | 返回值  | 说明                                                       |
| -------- | ------- | ---------------------------------------------------------- |
| 存在匹配 | `true`  | 带 `g`/`y` 时更新 `lastIndex` 为匹配结束位置               |
| 无匹配   | `false` | 带 `g`/`y` 时把 `lastIndex` 重置为 `0`                     |

### 官方示例

```uts
const str = 'table football'
const regex = new RegExp('foo*')
const globalRegex = new RegExp('foo*', 'g')

console.log(regex.test(str))        // true
console.log(globalRegex.lastIndex)  // 0
console.log(globalRegex.test(str))  // true
console.log(globalRegex.lastIndex)  // 9
console.log(globalRegex.test(str))  // false
```

---

## 三、核心机制

1. **只返回布尔值**：不提供匹配文本、捕获组或位置信息。
2. **`lastIndex` 联动**：
   - 带 `g` 或 `y` 时，`test` 从 `lastIndex` 处开始搜索，成功后把 `lastIndex` 更新为**匹配结束位置**。
   - 匹配失败时，`lastIndex` 被重置为 `0`。
   - 不带 `g`/`y` 时，`test` 始终从 `0` 开始，且**不修改** `lastIndex`。
3. **连续调用语义**：带 `g`/`y` 时，多次 `test` 会依次向后推进，直到返回 `false` 并归零。
4. **等价关系**：`regexp.test(str)` 等价于 `regexp.exec(str) !== null`。

---

## 四、与 `exec` 的区别

| 对比项         | `test`                    | `exec`                    |
| -------------- | ------------------------- | ------------------------- |
| 返回值         | `boolean`                 | `RegExpExecArray \| null` |
| 是否含捕获组   | 否                        | 是                        |
| 是否含位置信息 | 否                        | 是（`index` / `input`）   |
| 性能           | 相对较轻                  | 相对较重                  |
| 是否修改 `lastIndex` | 带 `g`/`y` 时修改   | 带 `g`/`y` 时修改         |
| 典型用途       | 判断是否存在、校验        | 提取内容、解析、遍历      |

> 选择建议：**只判断真假用 `test`，需要内容用 `exec`**。

---

## 五、典型使用场景 + 使用案例

### 场景概览

- 表单输入校验（邮箱、手机号、密码强度等）。
- 判断字符串中是否包含某类字符。
- 条件分支中根据匹配结果走不同逻辑。
- 注意带 `g` 标志时的 `lastIndex` 副作用。

---

### 案例 B1：表单校验（邮箱）

```uts
const emailRe = /^[\w.-]+@[\w.-]+\.\w+$/

function isValidEmail(input: string): boolean {
  return emailRe.test(input)
}

console.log(isValidEmail('user@example.com')) // true
console.log(isValidEmail('user@@example'))    // false
```

**要点**：校验类正则**不要加 `g`**，避免 `lastIndex` 状态影响多次校验。

---

### 案例 B2：存在性判断

```uts
const hasDigit = /\d/
console.log(hasDigit.test('abc123')) // true
console.log(hasDigit.test('abcdef')) // false
```

**要点**：只关心“有没有数字”，用 `test` 最简洁高效。

---

### 案例 B3：条件分支

```uts
const str = 'Hello World'
if (/world/i.test(str)) {
  console.log('包含 world（忽略大小写）')
} else {
  console.log('不包含 world')
}
// 输出：包含 world（忽略大小写）
```

**要点**：`i` 标志实现忽略大小写匹配，适合关键词判断。

---

### 案例 B4：`g` 标志的 `lastIndex` 副作用

```uts
const re = /foo/g
const str = 'foo foo'

console.log(re.lastIndex) // 0
console.log(re.test(str)) // true
console.log(re.lastIndex) // 3
console.log(re.test(str)) // true
console.log(re.lastIndex) // 7
console.log(re.test(str)) // false
console.log(re.lastIndex) // 0（失败后重置）
```

**要点**：带 `g` 的 `test` 会“记住”上次位置。若只想判断一次，请去掉 `g` 或每次手动 `re.lastIndex = 0`。

---

## 六、注意事项与兼容性

1. **`lastIndex` 副作用**：带 `g`/`y` 的 `test` 会修改 `lastIndex`，复用同一正则对象做多次独立判断时，务必重置 `re.lastIndex = 0` 或去掉 `g`。
2. **校验场景慎用 `g`**：表单校验等“每次独立判断”的场景，通常不应带 `g`。
3. **与 `exec` 的等价性**：`test` 等价于 `exec(...) != null`，但两者都会修改 `lastIndex`。
4. **引擎变更**：HBuilderX **4.72 之前** Android 端使用 PCRE 引擎；**4.72 及之后**完全遵循 ECMAScript 标准，行为与 Web 一致。
5. **编译目标**：`RegExp` 在 UTS 中编译为 Kotlin 的 `io.dcloud.uts.UTSRegExp`。
