# UTSJSONObject 使用总结

> 文档来源：https://doc.dcloud.net.cn/uni-app-x/uts/buildin-object-api/utsjsonobject.html

## 一、创建实例

文档明确说明：UTSJSONObject 实例目前主要通过**两种方式**创建。

### 1. 对象字面量

```uts
const person: UTSJSONObject = {
  name: 'Tom',
  printName: () => {
    // ...
  }
}
// 返回指定键对应的值，不存在则返回 null
let name: string = person["name"] as string
// get 方法可简化为下标运算符 [] 访问
name = person['name'] as string
// 增加或更新指定键对应的值
person.set('name', 'Tom1')
// set 方法可简化为下标运算符 [] 赋值
person['name'] = 'Tom2'
```

### 2. 通过 JSON 对象 parse 字符串

```uts
// 写法1 推荐
let person1: UTSJSONObject = JSON.parseObject('{"name":"Tom"}')!

// 写法2 推荐
const person2: UTSJSONObject = JSON.parse<UTSJSONObject>('{"name":"Tom"}')!

// 写法3 若 as 转换的实际类型不匹配会导致 crash，建议先 instanceof 判断再 as
const parseRet3 = JSON.parse('{"name":"Tom"}')
if (parseRet3 instanceof UTSJSONObject) {
  const person = parseRet3 as UTSJSONObject
}
```

---

## 二、静态方法

### 1. `keys(object: UTSJSONObject): Array<String>`

以数组形式返回对象内**可枚举属性的名称列表**。

```uts
let obj = { name: "zhangsan", age: 11 }
let ret1 = UTSJSONObject.keys(obj).length
console.log(ret1) // 2
```

### 2. `assign(...items): UTSJSONObject`

输入一个或多个 UTSJSONObject，合并后返回**新的** UTSJSONObject，包含全部属性；**同名属性以后传入的为准**。

```uts
let target = { a: 1, b: 2 };
let source = { b: 4, c: 5 };
let returnedTarget = UTSJSONObject.assign(target, source);
```

> ⚠️ 文档明确提示：与 JS 的 `Object.assign` 不同，**这里每次返回的都是一个新的对象**。

### 3. `assign<T>(...items: T[]): T`

泛型版本，合并后返回新的泛型对象 `T`，同名属性以后传入的为准。

```uts
let target1 = { a: 1, b: 2 };
let source1 = { b: 4, c: 5 };
let returned = UTSJSONObject.assign<UTSJSONObject>(target1, source1);
console.log(returned)
```

---

## 三、实例方法

### 1. `parse()`

将当前 UTSJSONObject 转换为具体类型 `T`，**失败返回 null**。

### 2. `get(key: string): any | null` / `set(key: string, value: any | null)`

- `get`：获取属性，返回 `any` 或 `null`（不存在时）；**可简化为下标 `[]` 访问**。
- `set`：添加或更新属性，返回 `void`；**可简化为下标 `[]` 赋值**。

### 3. 类型化取值方法

| 方法 | 无默认值返回 | 带默认值重载 |
|---|---|---|
| `getAny(key)` | `any \| null` | `getAny(key, def): any` |
| `getBoolean(key)` | `boolean \| null` | `getBoolean(key, def): boolean` |
| `getNumber(key)` | `number \| null` | `getNumber(key, def): number` |
| `getString(key)` | `string \| null` | `getString(key, def): string` |
| `getJSON(key)` | `UTSJSONObject \| null` | `getJSON(key, def): UTSJSONObject` |
| `getArray(key)` | `Array<T> \| null` | `getArray(key, def: Array<T>): Array<T>` |
| `getArray(key)` | `Array<any> \| null` | `getArray(key, def: Array<any>): Array<any>` |

**keyPath 支持**：所有带 `def` 的重载，其 `key` 参数**可以是 keyPath**（如 `"cars[0]"`）。

```uts
let obj = {
  "cars": [
    { name: "car1", value: 100 }
  ]
}
let cars: Array<UTSJSONObject> | null = obj.getArray<UTSJSONObject>("cars")
cars![0].set("value", 20)
let firstCar = obj.getJSON("cars[0]")
console.log(firstCar!['value']) // 20
```

**泛型数组取值**（注意鸿蒙平台 any 类型受限）：

```uts
let obj = JSON.parseObject('{"name":"tom","tag":["student","user"]}')
// #ifdef APP-HARMONY
let noGenericArray: (any | null)[] | null = obj!.getArray("tag")
// #endif
// #ifndef APP-HARMONY
let noGenericArray = obj!.getArray("tag") // Array<*>
// #endif
let genericArray = obj!.getArray<string>("tag") // Array<string>
```

### 4. `toMap(): Map<string, any>`

将当前 UTSJSONObject 实例转换为 `Map` 实例。

```uts
person1 = JSON.parseObject('{"name":"Tom"}')!
person1.toMap().forEach((value: any | null, key: string) => {
  console.log(key)
  console.log(value)
})
```

---

## 四、典型使用场景

1. **解析后端 JSON**：`JSON.parseObject(...)` / `JSON.parse<UTSJSONObject>(...)` 后，用 `getString`/`getNumber`/`getArray` 等类型化取值。
2. **动态/未知字段**：用 `keys()` 遍历属性名，或 `getAny()` 取值。
3. **合并配置/参数**：用 `assign()` 合并多个对象（注意返回新对象）。
4. **嵌套数组/深层字段**：用 `getArray<T>()` + keyPath（如 `"cars[0]"`）定位。
5. **与 Map 互转**：`toMap()` 便于用 Map 的 API 遍历。

---

## 五、常见问题

### 1. 目标语言为 js 时，UTSJSONObject 实例方法可以被覆盖

> 文档提示：如非必要请勿利用此特性。

```ts
const a = {
  getString: 1
}
console.log(a.getString) // 1
```

### 2. UTSJSONObject 与 type 的互转

（文档此节标题为「UTSJSONObject 与 ty…」，本次抓取被截断，未取到完整正文。）

---

## 六、兼容性（uni-app x）

| 方法 | 值（按文档表格顺序） |
|---|---|
| `keys` / `assign` / `assign<T>` | 4.25 / 4.18 / x / 4.61 / 4.23 |
| `get`/`set`/`getAny`/`getBoolean`/`getNumber`/`getString`/`getJSON`/`getArray`/`toMap` | 4.0 / 3.90 / 4.11 / 4.61 |
| 带 `def` 默认值重载 | 4.51 / 3.90 / 4.11 / 4.61 |
| `parse()` | x / 3.90 / x / x |

**uni-app**：`keys`/`assign` 为 4.18 / 4.23 / √；其余实例方法为 3.90 / √ / √。
