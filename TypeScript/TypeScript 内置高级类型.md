## 1.Partial 可选

>  将定义的接口中的属性设置为可选属性。

看一下[源码](https://so.csdn.net/so/search?q=源码&spm=1001.2101.3001.7020)

`Partial`源码

```ts
/**
 * Make all properties in T optional
  将T中的所有属性设置为可选
 */
type Partial<T> = {
    [P in keyof T]?: T[P];
};
```

1. `keyof`将一个接口对象的全部属性取出来，变成联合类型

2. `in` 我们可以理解成`for in P` 就是`key` 遍历  `keyof T`  就是联合类型的每一项

3.  `？`这个操作就是将每一个属性变成可选项

4.  `T[P]` 索引访问操作符，与 **JavaScript** 种访问属性值的操作类似

示例

```ts
interface User {
  name: string;
  age: number;
  password: string;
}

// 不好的做法 💩
interface PartialUser {
  name?: string;
  age?: number;
  password?: string;
}

// 正确的做法 ✅
type PartialUser = Partial<User>;
// 转换之后就是这样
type PartialUser = {
    name?: string | undefined;
    age?: number | undefined;
    password?: string | undefined;
}
```

## 2.Required 必需

> 与`Partial`相反，`Required`可以将接口中的可选属性设置为必需。

源码

```ts
/**
 * Make all properties in T required
 */
type Required<T> = {
    [P in keyof T]-?: T[P];
};
```

示例

```ts
interface User {
  name?: string;
  age?: number;
  password?: string;
}

// 不好的做法 💩
interface RequiredUser {
  name: string;
  age: number;
  password: string;
}

// 正确的做法 ✅
type RequiredUser = Required<User>;
```

## 3.Readonly 只读

> 将属性设置为**只读**。

源码

```ts
/**
 * Make all properties in T readonly
 */
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};
```

示例

```ts
interface User {
  role: string;
}

// 不好的做法 💩
const user: User = { role: "ADMIN" };
user.role = "USER";

// 正确的做法 ✅
type ReadonlyUser = Readonly<User>;
const user: ReadonlyUser = { role: "ADMIN" };
user.role = "USER"; // 将属性'role'设置为只读后，再次对'role'设值就会报错
```

## 4.Record 映射类型

> 构造一个对象类型，其属性key是`Keys`，属性value是`Tpye`。被用于映射一个类型的属性到另一个类型。

源码

```ts
/**
 * Construct a type with a set of properties K of type T
 */
type Record<K extends keyof any, T> = {
    [P in K]: T;
};
```

* `keyof any` 返回 `string | number | symbol `的联合类型
* `extends` 来约束我们的类型
* `in` 我们可以理解成`for in P` 就是`key` 遍历 `keyof any` 就是`string | number | symbol`联合类型的每一项

示例

```ts
interface Address {
  street: string;
  pin: number;
}

// 不好的做法 💩
interface Addresses {
  home: Address;
  office: Address;
}

// 正确的做法 ✅
type AddressesRecord = Record<"home" | "office", Address>;
```

## 5.Pick 选取

> 就是从一个复合类型中，取出几个想要的类型的组合

源码

```ts
/**
 * From T, pick a set of properties whose keys are in the union K
 */
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
};
```

示例

```ts
interface User {
  name: string;
  age: number;
  password: string;
}

// 不好的做法 💩
interface UserPartial {
  name: string;
  age: number;
}

// 正确的做法 ✅
type UserPartial = Pick<User, "name" | "age">;
```

## 6.Omit 剔除

> `Omit`是TypeScript3.5新增的一个辅助类型，它的作用主要是：**以一个类型为基础支持剔除某些属性，然后返回一个新类型**。

源码

```ts
/**
 * Construct a type with the properties of T except for those in type K.
 */
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

示例

```ts
interface User {
  name: string;
  age: number;
  password: string;
}

// 不好的做法 💩
interface UserPartial {
  name: string;
  age: number;
}

// 正确的做法 ✅
type UserPartial = Omit<User, "password" | "age">;
```

## 7.Exclude 属性排除

> 将类型中其中一些属性**排除**，并创建排除属性后的新类型。

源码

```ts
/**
 * Exclude from T those types that are assignable to U
 */
type Exclude<T, U> = T extends U ? never : T;
```

示例

```ts
type Role = "ADMIN" | "USER" | "GUEST";

// 不好的做法 💩
type NonAdminRole = "USER" | "GUEST";

// 正确的做法 ✅
type NonAdmin = Exclude<Role, "ADMIN">; // "USER" | "GUEST"
```

## 8.Extract 属性提取

> 它通过从可分配给联合的类型中提取所有联合成员来创建新类型。

源码

```ts
/**
 * Extract from T those types that are assignable to U
 */
type Extract<T, U> = T extends U ? T : never;
```

示例

```ts
type Role = "ADMIN" | "USER" | "GUEST";

// 不好的做法 💩
type AdminRole = "ADMIN";

// 正确的做法 ✅
type Admin = Extract<Role, "ADMIN">; // "ADMIN"
```

## 9.NonNullable 剔除`null`和`undefined`

> 通过从类型中排除`null`和`undefined`来创建新类型。

源码

```ts
/**
 * Exclude null and undefined from T
 */
type NonNullable<T> = T extends null | undefined ? never : T;
```

示例

```ts
type Role = "ADMIN" | "USER" | null;

// 不好的做法 💩
type NonNullableRole = "ADMIN" | "USER";

// 正确的做法 ✅
type NonNullableRole = NonNullable<Role>; // "ADMIN" | "USER"
```

## 10.Uppercase 属性大写

> 构建一个类型，定义其属性全是小写。然后使用`Uppercase`方法将其全部转化为大写。

源码

```ts
/**
 * Convert string literal type to uppercase
 */
type Uppercase<S extends string> = intrinsic;
```

示例

```ts
type Role = "admin" | "user" | "guest";

// 不好的做法 💩
type UppercaseRole = "ADMIN" | "USER" | "GUEST";

// 正确的做法 ✅
type UppercaseRole = Uppercase<Role>; // "ADMIN" | "USER" | "GUEST"
```

## 11.Lowercase 属性小写

> 与前面一个例子相反，先构建一个属性全是大写的类型，然后使用`Lowercase`方法将其全部转化为小写。

源码

```ts
/**
 * Convert string literal type to lowercase
 */
type Lowercase<S extends string> = intrinsic; // intrinsic 译 固有的
```

示例

```ts
type Role = "ADMIN" | "USER" | "GUEST";

// 不好的做法 💩
type LowercaseRole = "admin" | "user" | "guest";

// 正确的做法 ✅
type LowercaseRole = Lowercase<Role>; // "admin" | "user" | "guest"
```

## 12.Capitalize 属性首字母大写

> 将所有属性的首字母大写。

源码

```ts
/**
 * Convert first character of string literal type to uppercase
 */
type Capitalize<S extends string> = intrinsic;
```

示例

```ts
type Role = "admin" | "user" | "guest";

// 不好的做法 💩
type CapitalizeRole = "Admin" | "User" | "Guest";

// 正确的做法 ✅
type CapitalizeRole = Capitalize<Role>; // "Admin" | "User" | "Guest"
```

## 13.Uncapitalize 首字母小写

> 与`Capitalize`相反，将所有属性取消首字母大写。

源码

```ts
/**
 * Convert first character of string literal type to lowercase
 */
type Uncapitalize<S extends string> = intrinsic;
```

示例

```ts
type Role = "Admin" | "User" | "Guest";

// 不好的做法 💩
type UncapitalizeRole = "admin" | "user" | "guest";

// 正确的做法 ✅
type UncapitalizeRole = Uncapitalize<Role>; // "admin" | "user" | "guest"
```