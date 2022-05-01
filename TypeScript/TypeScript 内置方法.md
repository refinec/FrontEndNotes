## Partial

>  将定义的接口中的属性设置为可选属性。

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
```

## Required

> 与Partial相反，Required可以将接口中的可选属性设置为必需。

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

## Readonly

> 将属性设置为只读。

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

## Record

> 构造一个对象类型，其属性key是`Keys`,属性value是`Tpye`。被用于映射一个类型的属性到另一个类型。

```ts
interface Address {
  street: string;
  pin: number;
}

interface Addresses {
  home: Address;
  office: Address;
}

// Alternative ✅
type AddressesRecord = Record<"home" | "office", Address>;
```

## Pick

> 就是从一个复合类型中，取出几个想要的类型的组合，例如：

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

## Omit

> `Omit`是TypeScript3.5新增的一个辅助类型，它的作用主要是：**以一个类型为基础支持剔除某些属性，然后返回一个新类型**。

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
type UserPartial = Omit<User, "password">;
```

## Exclude

> 将类型中其中一些属性排除，并创建排除属性后的新类型。

```ts
type Role = "ADMIN" | "USER" | "GUEST";

// 不好的做法 💩
type NonAdminRole = "USER" | "GUEST";

// 正确的做法 ✅
type NonAdmin = Exclude<Role, "ADMIN">; // "USER" | "GUEST"
```

## Extract

> 它通过从可分配给联合的类型中提取所有联合成员来创建新类型。

```ts
type Role = "ADMIN" | "USER" | "GUEST";

// 不好的做法 💩
type AdminRole = "ADMIN";

// 正确的做法 ✅
type Admin = Extract<Role, "ADMIN">; // "ADMIN"
```

## NonNullable

> 通过从类型中排除`null`和`undefined`来创建新类型。

```ts
type Role = "ADMIN" | "USER" | null;

// 不好的做法 💩
type NonNullableRole = "ADMIN" | "USER";

// 正确的做法 ✅
type NonNullableRole = NonNullable<Role>; // "ADMIN" | "USER"
```

## Uppercase

> 构建一个类型，定义其属性全是小写。然后使用`Uppercase`方法将其全部转化为大写。

```ts
type Role = "admin" | "user" | "guest";

// 不好的做法 💩
type UppercaseRole = "ADMIN" | "USER" | "GUEST";

// 正确的做法 ✅
type UppercaseRole = Uppercase<Role>; // "ADMIN" | "USER" | "GUEST"
```

## Lowercase

> 与前面一个例子相反，先构建一个属性全是大写的类型，然后使用`Lowercase`方法将其全部转化为小写。

```ts
type Role = "ADMIN" | "USER" | "GUEST";

// 不好的做法 💩
type LowercaseRole = "admin" | "user" | "guest";

// 正确的做法 ✅
type LowercaseRole = Lowercase<Role>; // "admin" | "user" | "guest"
```

## Capitalize

> 将所有属性的首字母大写。

```ts
type Role = "admin" | "user" | "guest";

// 不好的做法 💩
type CapitalizeRole = "Admin" | "User" | "Guest";

// 正确的做法 ✅
type CapitalizeRole = Capitalize<Role>; // "Admin" | "User" | "Guest"
```

## Uncapitalize

> 与`Capitalize`相反，将所有属性取消首字母大写。

```ts
type Role = "Admin" | "User" | "Guest";

// 不好的做法 💩
type UncapitalizeRole = "admin" | "user" | "guest";

// 正确的做法 ✅
type UncapitalizeRole = Uncapitalize<Role>; // "admin" | "user" | "guest"
```