> AST 是 **Abstract Syntax Tree**（抽象语法树）的缩写，是源代码的树状结构表示。它是编译器、解释器、代码分析工具（如 ESLint、Babel）的核心数据结构，用于表示代码的语法结构，忽略空格、注释等无关细节。

* AST仅保留语法相关的必要信息（如忽略括号、分号、注释）
* AST以树状结构表示源代码，以便程序理解和处理。编译器（如 GCC、Clang）或解释器（如 Python 解释器）通过 AST 生成机器码或字节码。

Babel通过AST把ES6转义为ES5、Prettier通过AST进行代码格式化、ESLint通过AST检查代码是否存在语法错误。

```js
function add(a, b) {
  return a + b;
}
```

对应的 AST（简化版）：

```
Program
└── FunctionDeclaration
    ├── Identifier (name: "add")
    ├── Parameters
    │   ├── Identifier (name: "a")
    │   └── Identifier (name: "b")
    └── BlockStatement
        └── ReturnStatement
            └── BinaryExpression (+)
                ├── Identifier (name: "a")
                └── Identifier (name: "b")
```

## 如何生成 AST？

- **JavaScript**：使用 [Esprima](https://esprima.org/) 或 Babel 的解析器。

  ```javascript
  const esprima = require('esprima');
  const code = 'const x = 1;';
  const ast = esprima.parseScript(code);
  console.log(ast);
  ```

  

- **Python**：使用内置的 `ast` 模块。

  ```python
  import ast
  code = "print('hello')"
  tree = ast.parse(code)
  print(ast.dump(tree))
  ```