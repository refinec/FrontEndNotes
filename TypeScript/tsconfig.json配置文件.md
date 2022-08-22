# `tsconfig.json`配置文件

> 生成`tsconfig.json` 文件命令：`tsc --init`

TS 使用 `tsconfig.json` 作为其配置文件，它主要包含两块内容：

1. 指定待编译的文件

2. 定义编译选项

   另外，一般来说，`tsconfig.json` 文件所处的路径就是当前 TS 项目的根路径。

```json
{
  "compilerOptions": { // 用来配置编译选项
    "module": "commonjs",
    "noImplicitAny": true,
    "sourceMap": true,
    "declaration": true
  },
  "files": [ // 用来指定待编译文件.这里的待编译文件是指入口文件，任何被入口文件依赖的文件
    "app.ts",
    "foo.ts",
  ]
}
```

也可以使用 `include` 和 `exclude` 来指定和排除待编译文件：

```json
"include": [
  "src/**/*"
],
"exclude": [
  "node_modules",
  "**/*.spec.ts"
]
```

## 配置详解

> 下面简述 `compilerOptions` 中的选项配置，这些配置简单来说就是影响整个编译的过程和编译的结果。

* `target`: target 用来指定ts被编译为的ES版本.默认是被编译为ES3

  如：`es3`，`es5`，`es6`, `es2015`，`es2016`，`es2017`，`es2018`，`es2019`，`es2020`，`es2021`，`esnext`

* **`module`** ：module 模块化,指定要使用的模块化规范。如：`none`、`commonjs`、`amd`、`system`、`umd`、`es6`、`es2015` 或 `esnext`

* **`noImplicitAny`** ：存在隐式 any 时抛错 （默认为 false）

* **`sourceMap`** ：生成 map 文件 （默认为 false）

* **`declaration`** ：生成对应的 `.d.ts` 文件 (默认为 false)

`tsconfig.json`文件中指定了用来编译项目的根文件和编译选项。 一个项目可以通过以下方式之一来编译：

- 不带任何输入文件的情况下调用`tsc`，编译器会从当前目录开始去查找`tsconfig.json`文件，逐级向上搜索父目录。
- 不带任何输入文件的情况下调用`tsc`，且使用命令行参数`--project`（或`-p`）指定一个包含`tsconfig.json`文件的目录。

当命令行上指定了输入文件时，`tsconfig.json`文件会被忽略。

`tsconfig.json`文件可以是个空文件，那么所有默认的文件（如上面所述）都会以默认配置选项编译。

在命令行上指定的编译选项会覆盖在`tsconfig.json`文件里的相应选项。

查看所有可用的[compilerOptions编译选项](https://www.tslang.cn/docs/handbook/compiler-options.html)

```json
"compilerOptions": {
  "incremental": true, // TS编译器在第一次编译之后会生成一个存储编译信息的文件，第二次编译会在第一次的基础上进行增量编译，可以提高编译的速度
  "tsBuildInfoFile": "./buildFile", // 增量编译文件的存储位置
  "diagnostics": true, // 打印诊断信息 
  "target": "ES5", // 目标语言的版本
  "module": "CommonJS", // 生成代码的模板标准
  "outFile": "./app.js", // 将多个相互依赖的文件生成一个文件，可以用在AMD模块中，即开启时应设置"module": "AMD",
  "lib": ["DOM", "ES2015", "ScriptHost", "ES2019.Array"], // 编译过程中需要引入的库文件的列表(TS需要引用的库，即声明文件)，es5 默认引用dom、es5、scripthost,如需要使用es的高级版本特性，通常都需要配置，如es8的数组新特性需要引入"ES2019.Array",
  "allowJS": true, // 允许编译器编译JS，JSX文件
  "checkJs": true, // 允许在JS文件中报错，通常与allowJS一起使用
  "outDir": "./dist", // 指定输出目录
  "rootDir": "./", // 指定输出文件目录(用于输出)，用于控制输出目录结构
  "declaration": true, // 生成声明文件，开启后会自动生成声明文件
  "declarationDir": "./file", // 指定生成声明文件存放目录
  "emitDeclarationOnly": true, // 只生成声明文件，而不会生成js文件
  "sourceMap": true, // 生成目标文件的sourceMap文件
  "inlineSourceMap": true, // 生成目标文件的inline SourceMap，inline SourceMap会包含在生成的js文件中
  "declarationMap": true, // 为声明文件生成sourceMap
  "experimentalDecorators": true, // 启用实验性的ES装饰器
  "typeRoots": [], // 声明文件目录，默认时node_modules/@types
  "types": [ //指定引入的类型声明文件，默认是自动引入所有声明文件，一旦指定该选项，则会禁用自动引入，改为只引入指定的类型声明文件，如果指定空数组[]则不引用任何文件
    ],
  "removeComments":true, // 删除注释 
  "noEmit": true, // 不输出文件,即编译后不会生成任何js文件
  "noEmitOnError": true, // 发送错误时不输出任何文件
  "noEmitHelpers": true, // 不生成helper函数，减小体积，需要额外安装，常配合importHelpers一起使用
  "importHelpers": true, // 通过tslib引入helper函数，文件必须是模块
  "downlevelIteration": true, // 降级遍历器实现，如果目标源是es3/5，那么遍历器会有降级的实现
  "jsx": "react", // 在 .tsx文件里支持JSX
  "strict": true, // 开启所有严格的类型检查
  "alwaysStrict": true, // 在代码中注入'use strict'
  "noImplicitAny": true, // 不允许隐式的any类型
  "strictNullChecks": true, // 不允许把null、undefined赋值给其他类型的变量
  "strictFunctionTypes": true, // 不允许函数参数双向协变
  "strictPropertyInitialization": true, // 类的实例属性必须初始化
  "strictBindCallApply": true, // 严格的bind/call/apply检查
  "noImplicitThis": true, // 不允许this有隐式的any类型
  "noUnusedLocals": true, // 检查只声明、未使用的局部变量(只提示不报错)
  "noUnusedParameters": true, // 检查未使用的函数参数(只提示不报错)
  "noFallthroughCasesInSwitch": true, // 防止switch语句贯穿(即如果没有break语句后面不会执行)
  "noImplicitReturns": true, //每个分支都会有返回值
  "esModuleInterop": true, // 允许export=导出，由import from 导入
  "allowUmdGlobalAccess": true, // 允许在模块中全局变量的方式访问umd模块
  "moduleResolution": "node", // 模块解析策略，ts默认用node的解析策略，即相对的方式导入
  "jsx": "preserve", // TS的三种JSX模式 'preserve','react','react-native'
  "baseUrl": "./", // 解析非相对模块的基地址，默认是当前目录
  "paths": { // 路径映射，相对于baseUrl（指定模块的路径，和baseUrl有关联），和webpack中resolve.alias配置一样
    // 如使用jq时不想使用默认版本，而需要手动指定版本，可进行如下配置
    "jquery": ["node_modules/jquery/dist/jquery.min.js"],
    "src": [ //指定后可以在文件之直接 import * from 'src';
        "./src"
      ],
  },
  "rootDirs": ["src","out"], // 将多个目录放在一个虚拟目录下，用于运行时，即编译后引入文件的位置可能发生变化，这也设置可以虚拟src和out在同一个目录下，不用再去改变路径也不会报错
  "listEmittedFiles": true, // 打印输出文件
  "listFiles": true// 打印编译的文件(包括引用的声明文件)
}
 
// 指定一个匹配列表（属于自动指定该路径下的所有ts相关文件）
"include": [
   "src/**/*"
],
// 指定一个排除列表（include的反向操作）
 "exclude": [
   "demo.ts"
],
// 指定哪些文件使用该配置（属于手动一个个指定文件）
 "files": [
   "demo.ts"
]
```

```ts
{
    "compilerOptions": { // "compilerOptions"可以被忽略，这时编译器会使用默认值。
        "module": "commonjs", // 指定生成哪个模块系统代码
        "noImplicitAny": true, // 在表达式和声明上有隐含的 any类型时报错
        "removeComments": true, // 删除所有注释，除了以 /!*开头的版权信息
        "preserveConstEnums": true, // 保留 const和 enum声明
        "sourceMap": true // 生成相应的 .map文件。
    },
    "files": [ // "files"指定一个包含相对或绝对文件路径的列表
        "core.ts",
        "sys.ts",
        "types.ts",
        "scanner.ts",
        "parser.ts",
        "utilities.ts",
        "binder.ts",
        "checker.ts",
        "emitter.ts",
        "program.ts",
        "commandLineParser.ts",
        "tsc.ts",
        "diagnosticInformationMap.generated.ts"
    ],
    
    //"include"和"exclude"属性指定一个文件glob匹配模式列表。
    "include": [ 
        "src/**/*"
    ],
    "exclude": [
        "node_modules",
        "**/*.spec.ts"
    ]
}
```

 **支持的`glob`通配符有**：

- `*` 匹配0或多个字符（不包括目录分隔符）
- `?` 匹配一个任意字符（不包括目录分隔符）
- `**/` 递归匹配任意子目录

**如果一个glob模式里的某部分只包含`*`或`.*`**，那么仅有支持的文件扩展名类型被包含在内（比如默认`.ts`，`.tsx`，和`.d.ts`， 如果 `allowJs`设置能`true`还包含`.js`和`.jsx`）。

- **如果`"files"`和`"include"`都没有被指定**，编译器默认包含当前目录和子目录下所有的TypeScript文件（`.ts`, `.d.ts` 和 `.tsx`），排除在`"exclude"`里指定的文件。JS文件（`.js`和`.jsx`）也被包含进来如果`allowJs`被设置成`true`。 
- **如果指定了 `"files"`或`"include"`**，编译器会将它们结合一并包含进来。
- **使用 `"outDir"`指定的目录下的文件永远会被编译器排除，除非你明确地使用`"files"`将其包含进来（这时就算用`exclude`指定也没用）**。

* **使用`"include"`引入的文件可以使用`"exclude"`属性过滤**。 然而，通过 `"files"`属性明确指定的文件却总是会被包含在内，不管`"exclude"`如何设置。 如果没有特殊指定， `"exclude"`默认情况下会排除`node_modules`，`bower_components`，`jspm_packages`和`<outDir>`目录。

* **任何被`"files"`或`"include"`指定的文件所引用的文件也会被包含进来**。 `A.ts`引用了`B.ts`，因此`B.ts`不能被排除，除非引用它的`A.ts`在`"exclude"`列表中

* 注意**编译器不会去引入那些可能做为输出的文件**；比如，假设我们包含了`index.ts`，那么`index.d.ts`和`index.js`会被排除在外。 通常来讲，不推荐只有扩展名的不同来区分同目录下的文件。

### 常用的

1. `include`
   指定编译文件默认是编译当前目录下所有的ts文件
2. `exclude`
   指定排除的文件
3. `target`
   指定要编译生成的目标 js 版本，例如`es5` 、`es6`
4. `allowJS` 
   是否允许编译js文件
5. `removeComments`
   是否在编译过程中删除文件中的注释
6. `rootDir`
   编译文件的目录
7. `outDir`
   输出的目录
8. `sourceMap`
   代码源文件
9. `strict`
   严格模式
10. `module`
    默认common.js  可选es6模式 amd  umd 等

## `@types`，`typeRoots`和`types`

**默认所有*可见的*"`@types`"包会在编译过程中被包含进来**。 `node_modules/@types`文件夹下以及它们子文件夹下的所有包都是*可见的*； 也就是说， `./node_modules/@types/`，`../node_modules/@types/`和`../../node_modules/@types/`等等

- **如果指定了`typeRoots`**，*只有*`typeRoots`下面的包才会被包含进来:

  ```json
  {
     "compilerOptions": {
         "typeRoots" : ["./typings"] // 这个配置文件会包含所有./typings下面的包，而不包含./node_modules/@types里面的包
     }
  }
  ```

- **如果指定了`types`**，只有被列出来的包才会被包含进来

  ```json
  {
     "compilerOptions": {
         // 这个tsconfig.json文件将仅会包含 ./node_modules/@types/node，./node_modules/@types/lodash和./node_modules/@types/express。
         // node_modules/@types/*里面的其它包不会被引入进来。
          "types" : ["node", "lodash", "express"]
     }
  }
  ```

  指定`"types": []`来**禁用自动引入**`@types`包。

  注意，自动引入只在你使用了全局的声明（相反于模块）时是重要的。 如果你使用 `import "foo"`语句，TypeScript仍然会查找`node_modules`和`node_modules/@types`文件夹来获取`foo`包。

## 使用`extends`继承配置

> `tsconfig.json`文件可以利用`extends`属性从另一个配置文件里继承配置。

**`extends`的值是一个字符串，包含指向另一个要继承文件的路径。来至所继承配置文件的`files`，`include`和`exclude`*覆盖*源配置文件的属性。**

```json
# configs/base.json
{
  "compilerOptions": {
    "noImplicitAny": true,
    "strictNullChecks": true
  }
}
# tsconfig.json
{
  "extends": "./configs/base",
  "files": [
    "main.ts",
    "supplemental.ts"
  ]
}
# tsconfig.nostrictnull.json
{
  "extends": "./tsconfig",
  "compilerOptions": {
    "strictNullChecks": false
  }
}
```

## `compileOnSave`

在最顶层设置`compileOnSave`标记，可以让IDE在保存文件的时候根据`tsconfig.json`重新生成文件

要想支持这个特性需要Visual Studio 2015， TypeScript1.8.4以上并且安装[atom-typescript](https://github.com/TypeStrong/atom-typescript#compile-on-save)插件。

```json
{
    "compileOnSave": true,
    "compilerOptions": {
        "noImplicitAny" : true
    }
}
```

## 模式

http://json.schemastore.org/tsconfig.

## 项目引用

顶层属性`references`,它是一个对象的数组，指明要引用的工程

```json
{
    "compilerOptions": {
        // The usual
    },
    "references": [
        { "path": "../src" }
    ]
}
```

**每个引用的`path`属性都可以指向到包含`tsconfig.json`文件的目录，或者直接指向到配置文件本身（名字是任意的）**。

当你引用一个工程时，会发生下面的事：

- 导入引用工程中的模块实际加载的是它*输出*的声明文件（`.d.ts`）。
- 如果引用的工程生成一个`outFile`，那么这个输出文件的`.d.ts`文件里的声明对于当前工程是可见的。
- 构建模式（后文）会根据需要自动地构建引用的工程。

当你拆分成多个工程后，会显著地加速类型检查和编译，减少编辑器的内存占用，还会改善程序在逻辑上进行分组。

## Webpack

`npm install ts-loader --save-dev` 查看[更多关于ts-loader的详细信息](https://www.npmjs.com/package/ts-loader) 还有 [awesome-typescript-loader](https://www.npmjs.com/package/awesome-typescript-loader)

```js
# webpack.config.js
module.exports = {
    entry: "./src/index.tsx",
    output: {
        filename: "bundle.js"
    },
    resolve: {
        // 添加“.ts”和“.tsx”作为可解析扩展名。
        extensions: ["", ".webpack.js", ".web.js", ".ts", ".tsx", ".js"]
    },
    module: {
        loaders: [
            // 所有扩展名为“.ts”或“.tsx”的文件都将由“ts-loader”处理
            { test: /\.tsx?$/, loader: "ts-loader" }
        ]
    }
};
```

