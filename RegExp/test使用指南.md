# UTS RegExp 实例方法 `test` 使用指南

> 参考文档：<https://doc.dcloud.net.cn/uni-app-x/uts/buildin-object-api/regexp.html>
> 适用环境：uni-app x / UTS（编译为 Kotlin `io.dcloud.uts.UTSRegExp`）

---

## 一、什么是 `test`

`test` 是 `RegExp` 实例上用于**判断是否存在匹配**的方法。它会在目标字符串中查找是否符合正则，返回一个布尔值：

- 匹配到 → `true`
- 没匹配到 → `false`

它**不返回匹配内容**，只回答"有没有"。因此它非常适合**表单校验、条件判断、存在性检查**等场景。

---

## 二、签名与参数 / 返回值

### 签名

```ts
test(string: string): boolean
```

### 参数

| 参数名   | 类型     | 是否必填 | 说明               |
| -------- | -------- | -------- | ------------------ |
| `string` | `string` | 是       | 要执行匹配的字符串 |

### 返回值

| 情况     | 返回值  | 说明                                                       |
| -------- | ------- | ---------------------------------------------------------- |
| 匹配成功 | `true`  | 带 `g`/`y` 时会更新 `lastIndex`                            |
| 匹配失败 | `false` | 带 `g`/`y` 时会把 `lastIndex` 重置为 `0`                   |

### 官方示例

```ts
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

1. **只返回布尔值**：`test` 内部执行一次匹配，但只把结果转成 `true` / `false`，不暴露捕获组、下标等信息。
2. **`lastIndex` 联动**：
   - 当正则**带 `g`（global）或 `y`（sticky）标志**时，`test` 会从 `lastIndex` 处开始查找，匹配成功后把 `lastIndex` 更新为本次匹配结束的位置。
   - 因此**连续调用 `test` 会从上次结束位置继续**，可能得到交替的 `true` / `false`（见案例 B4）。
   - 当正则**不带 `g`/`y`** 时，每次都从字符串开头查找，`lastIndex` 始终为 `0`。
3. **失败重置**：匹配失败返回 `false`，并把 `lastIndex` 重置为 `0`。
4. **等价关系**：`re.test(str)` 等价于 `re.exec(str) != null`。

---

## 四、与 `exec` 的区别

| 对比项         | `test`            | `exec`                    |
| -------------- | ----------------- | ------------------------- |
| 返回值         | `boolean`         | `RegExpExecArray \| null` |
| 是否返回捕获组 | ❌ 否             | ✅ 是                     |
| 是否返回下标   | ❌ 否             | ✅ 是（`index`）          |
| 是否返回原串   | ❌ 否             | ✅ 是（`input`）          |
| 性能           | 相对较轻          | 相对较重（构造数组）      |
| `lastIndex`    | 带 `g`/`y` 时会被更新 | 带 `g`/`y` 时会被更新 |
| 典型用途       | 判断是否存在、表单校验 | 提取内容、循环遍历所有匹配 |

> 结论：**只需要"是否匹配"用 `test`；需要"匹配内容"用 `exec`。**

---

## 五、典型使用场景 + 使用案例

### 场景 1：表单校验

**案例 B1 —— 校验邮箱格式**

```ts
const emailRe = /^[\w.-]+@[\w.-]+\.\w+$/

console.log(emailRe.test('user@example.com')) // true
console.log(emailRe.test('not-an-email'))     // false

if (!emailRe.test('user@example.com')) {
  console.log('邮箱格式不正确')
} else {
  console.log('邮箱格式正确')
}
```

---

### 场景 2：存在性检查

**案例 B2 —— 判断字符串中是否包含数字**

```ts
console.log(/\d/.test('abc123')) // true
console.log(/\d/.test('abcdef')) // false
```

---

### 场景 3：条件分支

**案例 B3 —— 根据是否包含关键字走不同逻辑**

```ts
const str = 'Hello World'
if (/world/i.test(str)) {
  console.log('包含 world（忽略大小写）')
} else {
  console.log('不包含 world')
}
```

---

### 场景 4：`g` 标志下的 `lastIndex` 副作用

**案例 B4 —— 连续调用 `test` 的交替结果**

```ts
const re = /foo/g
const str = 'foo foo'

console.log(re.test(str))       // true  （匹配第 1 个 foo，lastIndex = 3）
console.log(re.lastIndex)       // 3
console.log(re.test(str))       // true  （匹配第 2 个 foo，lastIndex = 7）
console.log(re.lastIndex)       // 7
console.log(re.test(str))       // false （从位置 7 开始已无 foo，lastIndex 重置为 0）
console.log(re.lastIndex)       // 0
```

> 说明：带 `g` 的 `test` 是**有状态**的。若只想做一次"是否存在"的判断，建议**不要带 `g`**，或在调用前重置 `re.lastIndex = 0`。

---

## 六、注意事项与兼容性

1. **`lastIndex` 副作用**：带 `g`/`y` 的正则对象是**有状态**的，连续调用 `test` 会相互影响，可能得到交替的 `true`/`false`。做纯校验时建议使用**不带 `g`/`y`** 的正则。
2. **重置状态**：若必须复用带 `g` 的正则，请在每次判断前执行 `re.lastIndex = 0`。
3. **`test` 与 `exec` 等价关系**：`re.test(str)` 等价于 `re.exec(str) != null`。
4. **锚点校验**：做完整格式校验时记得加 `^` 和 `$`，否则只做"包含"判断（如 `/\d/` 只判断是否含数字）。
5. **引擎差异**：HBuilderX **4.72 之前** Android 端使用 PCRE 引擎；**4.72 及之后**完全遵循 ECMAScript 标准（与 Web 行为一致）。跨端行为请以 4.72+ 为准。
6. **编译目标**：`RegExp` 在 UTS 中编译为 Kotlin 的 `io.dcloud.uts.UTSRegExp`。
