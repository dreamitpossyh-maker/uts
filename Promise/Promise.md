# UTS Promise 使用总结

> 文档来源：<https://doc.dcloud.net.cn/uni-app-x/uts/buildin-object-api/promise.html>
> 适用：uni-app x / uni-app 的 UTS 环境

Promise 是 UTS 内置的对象 API，用于处理异步操作。它代表一个异步操作的最终完成（或失败）及其结果值。

---

## 一、构造函数 `new Promise(fn)`

创建一个新的 Promise 对象。该构造函数主要用于封装还没有添加 Promise 支持的函数。

### 参数

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `fn` | `(resolve: (value: T) => void, reject: (reason?: any) => void) => void` | 是 | 在构造函数中执行的 function。 |

### 示例

```uts
// 常规方式
let p = new Promise<number>((resolve, reject) => {
  let success = true;
  setTimeout(() => {
    if (success) {
      resolve(2024)
    } else {
      let error = new Error("something going wrong")
      reject(error)
    }
  }, 10);
});
p.then((res: number) => {
  console.log(res) // 2024
})

// 单个函数（只使用 resolve）
let p1 = new Promise<any | null>((resolve: (value: any | null) => void) => {
  setTimeout(function () {
    resolve(null)
  }, 10);
})
p1.then((res: any | null) => {
  console.log(res) // null
})

// 指定函数类型（同时使用 resolve 和 reject）
let p2 = new Promise((resolve: (value: string) => void, reject: (error: any | null) => void) => {
  setTimeout(function () {
    reject(null)
  }, 10);
})
p2.then()
  .catch((error: any | null) => {
    console.log(error) // null
  })
```

> **注意**：UTS 是强类型语言，泛型、参数、返回值都需要显式标注类型。

---

## 二、实例方法

### 1. `then()`

将一个兑现处理器和拒绝处理器附加到 Promise 上，并返回一个新的 Promise，解决为调用处理器得到的返回值，或者如果 Promise 没有被处理（即相关处理器 `onFulfilled` 或 `onRejected` 不是函数），则以原始敲定值解决。

**返回值**：`Promise<T>` —— 一个新的 Promise。

```uts
let p = new Promise<string>((reslove, reject) => {
  setTimeout(function () {
    reslove("hello world")
  }, 13);
})

p.then(
  (res: string): Promise<string> => {
    console.log(res) // "hello world"
    let p0 = new Promise<string>((reslove, reject) => {
      var success = true;
      setTimeout(function () {
        success = true;
        if (success) {
          reslove(res + " 2024")
        } else {
          reject("first then p0 reject message")
        }
      }, 8);
    })
    return p0;
  },
  (err: any | null): void => {
    throw new Error("first then throw error")
  }
)
  .catch(
    (err): string => {
      return "first catch return message"
    }
  )
  .then(
    (res) => {
      console.log(res) // "hello world 2024"
    }
  )
  .catch(
    (err) => {
      console.log(err, "this is seconded catch")
    }
  )
  .finally(() => {
    console.log("finally") // "finally"
  })
```

### 2. `then(onFulfilled, onRejected?)`

将一个兑现处理器和拒绝处理器附加到 Promise 上，并返回一个新的 Promise。

#### 参数

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `onFulfilled` | `(res: T) => R` / `() => R` / `(res: T) => Promise<R>` / `() => Promise<R>` | 是 | 一个在此 Promise 对象被兑现时异步执行的函数。它的返回值将成为 `then()` 返回的 Promise 对象的兑现值。 |
| `onRejected` | `(...args?: any) => any` | 否 | 一个在此 Promise 对象被拒绝时异步执行的函数。它的返回值将成为 `catch()` 返回的 Promise 对象的兑现值。 |

**返回值**：`Promise<R>` —— 一个新的 Promise。

> `onFulfilled` 支持多种签名：无参、带参、返回普通值或返回 Promise，UTS 会根据实际写法进行类型推断。

### 3. `catch()`

将一个拒绝处理回调函数附加到 Promise 上，并返回一个新的 Promise，如果回调被调用，则解决为回调的返回值，如果 Promise 被兑现，解决为其原始兑现值。

**返回值**：`Promise<T>` —— 一个新的 Promise。

```uts
let p = Promise.reject()
p.then()
  .catch((res: any | null) => {
    console.log(res) // null
  })

// catch 指定返回值
new Promise<string>((resolve, reject) => {
  setTimeout(() => {
    reject("2024")
  }, 12)
})
  .then()
  .catch((res: any | null): string => {
    console.log(res!) // "2024"
    return "hello world"
  })
  .then((res) => {
    console.log(res) // "hello world"
  })

// catch 返回 Promise
new Promise<string>((resolve, reject) => {
  setTimeout(() => {
    reject("2024")
  }, 12)
})
  .then()
  .catch((res: any | null): Promise<number> => {
    console.log(res!) // "2024"
    let p = new Promise<number>((resolve, reject) => {
      resolve(1)
    })
    return p
  })
  .then((res) => {
    console.log(res) // 1
  })
```

### 4. `catch(onRejected)`

将一个拒绝处理回调函数附加到 Promise 上，并返回一个新的 Promise。

#### 参数

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `onRejected` | `(res?: any) => R` / `() => R` / `() => Promise<R>` | 是 | 一个在此 Promise 对象被拒绝时异步执行的函数。它的返回值将成为 `catch()` 返回的 Promise 对象的兑现值。 |

**返回值**：`Promise<R>` —— 一个新的 Promise。

### 5. `finally(callback)`

在 Promise 敲定（兑现或拒绝）后执行的回调，无论结果如何都会执行。它不接收参数，也不改变 Promise 的结果。

```uts
p.finally(() => {
  console.log("finally") // 无论成功或失败都会执行
})
```

---

## 三、静态方法

### 1. `Promise.resolve()`

返回一个已兑现（resolved）的 Promise。

```uts
let p = Promise.resolve(2024)
p.then((res) => {
  console.log(res) // 2024
})
```

### 2. `Promise.reject(value?)`

返回一个已拒绝（rejected）的 Promise。

```uts
let p = Promise.reject()
p.then()
  .catch((res: any | null) => {
    console.log(res) // null
  })
```

### 3. `Promise.all(arr)`

接收一个 Promise 数组，只有当所有 Promise 都兑现时才兑现，返回结果数组；只要有一个拒绝就立即拒绝。

```uts
let p1 = Promise.resolve(1)
let p2 = Promise.resolve(2)
Promise.all([p1, p2]).then((res) => {
  console.log(res) // [1, 2]
})
```

### 4. `Promise.race(arr)`

接收一个 Promise 数组，返回最先敲定（兑现或拒绝）的那个 Promise 的结果。

```uts
let p1 = new Promise<number>((resolve) => {
  setTimeout(() => resolve(1), 100)
})
let p2 = new Promise<number>((resolve) => {
  setTimeout(() => resolve(2), 10)
})
Promise.race([p1, p2]).then((res) => {
  console.log(res) // 2
})
```

### 5. `Promise.any(arr)`

接收一个 Promise 数组，只要有一个兑现就返回该兑现值；只有全部拒绝时才拒绝。

```uts
let p1 = Promise.reject("err1")
let p2 = Promise.resolve(2)
Promise.any([p1, p2]).then((res) => {
  console.log(res) // 2
})
```

### 6. `Promise.allSettled(arr)`

接收一个 Promise 数组，等待所有 Promise 都敲定（无论兑现或拒绝），返回每个结果的状态对象数组。

```uts
let p1 = Promise.resolve(1)
let p2 = Promise.reject("err")
Promise.allSettled([p1, p2]).then((res) => {
  console.log(res) // [{ status: "fulfilled", value: 1 }, { status: "rejected", reason: "err" }]
})
```

---

## 四、兼容性

### uni-app x 兼容性

| Android | iOS | HarmonyOS | Web | 微信小程序 |
| --- | --- | --- | --- | --- |
| 4.0 | 3.9 | 4.11 | 4.61 | 4.31 |

### uni-app 兼容性

| App | H5 | 微信小程序 |
| --- | --- | --- |
| 3.9 | 4.31 | √ |

---

## 五、注意事项（Bug & Tips）

- **编译目标差异**：
  - 编译到 **kotlin** 时，Promise 对应 `io.dcloud.uts.UTSPromise`。
  - 编译到 **swift** 时，Promise 对应 `UTSPromise`（自 HBuilderX 4.31 起支持）。
  - uvue 在 iOS 上使用 JS 驱动，因此不受 4.31 版本限制。
- **强类型要求**：UTS 中泛型、参数、返回值都需要显式标注类型，否则可能编译报错。
- **`then()` 无参调用**：`p.then()` 会透传原始敲定值，常用于只关心 `catch` 的场景。
- **`catch` 返回值**：`catch` 回调的返回值会成为后续 `then` 的兑现值，可用于错误恢复。

---

## 六、方法速查表

| 方法 | 类型 | 说明 |
| --- | --- | --- |
| `new Promise(fn)` | 构造函数 | 创建 Promise，封装异步操作 |
| `then()` | 实例方法 | 透传原始敲定值，返回新 Promise |
| `then(onFulfilled, onRejected?)` | 实例方法 | 附加兑现/拒绝处理器 |
| `catch()` | 实例方法 | 透传，返回新 Promise |
| `catch(onRejected)` | 实例方法 | 附加拒绝处理器 |
| `finally(callback)` | 实例方法 | 敲定后执行，不改变结果 |
| `Promise.resolve()` | 静态方法 | 返回已兑现的 Promise |
| `Promise.reject(value?)` | 静态方法 | 返回已拒绝的 Promise |
| `Promise.all(arr)` | 静态方法 | 全部兑现才兑现 |
| `Promise.race(arr)` | 静态方法 | 返回最先敲定的结果 |
| `Promise.any(arr)` | 静态方法 | 任一兑现即兑现 |
| `Promise.allSettled(arr)` | 静态方法 | 等待全部敲定，返回状态数组 |
