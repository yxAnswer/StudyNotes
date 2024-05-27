# 初始化工程和配置editorconfig  eslint等



## 1.yarn 和 npm配置

### 环境安装和配置

1. 下载最新稳定版 Node.js 

2. 安装cnpm、yarn或者pnpm

```shell
# 安装yarn
npm install -g yarn 
# 安装pnpm
npm install -g pnpm
```

> npm 安装文档：https://pnpm.io/installation

### npm修改配置

在项目根目录（package.json同一目录）中新建 `.npmrc`文件，编辑文件内容如下：

```ini
registry=https://registry.npm.taobao.org 
```

保存后再使用`npm install`下载包的速度会快很多

### yarn修改配置

在项目根目录（package.json同一目录）中新建 `.yarnrc`文件，编辑文件内容如下：

```bash
registry "https://registry.npm.taobao.org"
```

### 命令行修改配置

```shell
npm config set registry https://registry.npm.taobao.org

yarn config set registry https://registry.npm.taobao.org
```

### pnpm介绍

**一、什么是pnpm**

快速的，节省磁盘空间的包管理工具。

**二、pnpm的特点**

1、快速

pnpm比其他包管理器快2倍。

2、高效

node_modules 中的文件为复制或链接自特定的内容寻址存储库。

3、支持monorepos

pnpm内置支持单仓多包。

4、严格

pnpm 默认创建了一个非平铺的 node_modules，因此代码无法访问任意包。

**pnpm使用命令**

```js
pnpm install 包名  //

pnpm i 包名

pnpm add 包名    // -S  默认写入dependencies

pnpm add -D    // -D devDependencies

pnpm add -g    // 全局安装

pnpm remove 包名 //移除

pnpm up                //更新所有依赖项

pnpm upgrade 包        //更新包

pnpm upgrade 包 --global   //更新全局包
```

### 使用vite创建react+typescript项目：

```shell
pnpm create vite 
yarn create vite
npm init vite@latest


```





## 2.EditorConfig

**项目根目录创建 .editorconfig 文件**

```js
# https://editorconfig.org
root = tue

[*]
charset = utf-8
indent_style = space
indent_size = 2
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
```

**配置解读**

root=true 对所有文件生效

end_of_line= lf 不同操作系统换行符不同

```js
[*] 对所有格式文件
end_of_line
lf | cr | crlf (大小写不限）
复制代码
end_of_line设置的换行符的表示方式。先看一下这几个值是什么意思

lf：全拼Line Feed，意思是换行，用符号 \n 表示
cr: 全拼Carriage Return， 意思是回车， 用符号 \r 表示
crlf：cr 和 lf的结合，回车换行，用符号 \r\n
```

insert_final_newline = true 代码最后新增一行

trim_trailing_whitespace = true 修剪尾随空格







## 3.Prettier

### 测试地址： [Prettier](https://prettier.io/playground/)

### 部分配置：

- `trailingComma` ：对象的最后一个属性末尾也会添加 `,` ，比如 `{ a: 1, b: 2 }` 会格式为 `{ a: 1, b: 2, }` 
- `tabWidth` ：缩进大小
- `semi` ：是否添加分号
- `singleQuote` ：是否单引号
- `jsxSingleQuote` ：jsx 语法下是否单引号。
- `endOfLine` ：与 `.editorconfig` 保持一致设置
- `printWidth` ：单行代码最长字符长度，超过之后会自动格式化换行
- `bracketSpacing` ：在对象中的括号之间打印空格， `{a: 5}` 格式化为 `{ a: 5 }` 
- `arrowParens` ：箭头函数的参数无论有几个，都要括号包裹。比如 `(a) => {}` ，如果设为 `avoid` ，会自动格式化为 `a => {}` 

### 安装配置流程

1. 安装prettier插件和vscode扩展 [Prettier - Code formatter](https://link.juejin.cn/?target=https%3A%2F%2Fmarketplace.visualstudio.com%2Fitems%3FitemName%3Desbenp.prettier-vscode "https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode") ：

`pnpm add prettier -D`

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6410957ae01f44daa37bc9b3024932dd~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

2. 在项目根目录新建一个文件夹 `.vscode` ，在此文件下再建一个 `settings.json` 文件

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f35d7fef8e3c41d1b784597dc34dcbcc~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

代码保存时会自动格式化代码

```json
{
  // 保存自动格式化代码
  "editor.formatOnSave": true,
  // 开启stylelint自动修复
  "editor.codeActionsOnSave": {
    "source.fixAll": true
  },
  "editor.defaultFormatter": "esbenp.prettier-vscode"
}
```

3. prettier配置   

   根目录创建配置文件： .prettierrc.cjs

```js
module.exports = {
  // 每行最大列，超过换行
  printWidth: 120,
  // 使用制表符而不是空格缩进
  useTabs: false,
  // 缩进
  tabWidth: 2,
  // 结尾不用分号
  semi: false,
  // 使用单引号
  singleQuote: true,
  // 在JSX中使用单引号而不是双引号
  jsxSingleQuote: true,
  // 箭头函数里面，如果是一个参数的时候，去掉括号
  arrowParens: 'avoid',
  // 对象、数组括号与文字间添加空格
  bracketSpacing: true,
  // 尾随逗号
  trailingComma: 'none',
}
```



## 4.ESLint

官方链接：[Getting Started with ESLint - ESLint - Pluggable JavaScript Linter](https://eslint.org/docs/latest/use/getting-started)

中文文档：[Configuring ESLint - ESLint中文文档](https://eslint.bootcss.com/docs/user-guide/configuring)

`Prettier` 都是为了解决**代码风格问题**，而 `ESLint` 是主要为了解决**代码质量问题**，它能在我们编写代码时就检测出程序可能出现的隐性BUG，通过 `eslint --fix` 还能自动修复一些代码写法问题，比如你定义了 `var a = 3` ，自动修复后为 `const a = 3` 。还有许多类似的强制扭转代码最佳写法的规则，在无法自动修复时，会给出红线提示，强迫开发人员为其寻求更好的解决方案。

### 首先在项目中安装 `eslint` ：

```bash
 # npm
 npm init @eslint/config
 # 或者
 npx eslint --init

 # 使用yarn时，需要先安装eslint插件才可以执行命令
 yarn add eslint -D

 yarn eslint --init

 # pnpm
 pnpm eslint --init
复制代码
```

### 初始化过程：

- How would you like to use ESLint?

  选择第三条 `To check syntax, find problems, and enforce code style` ，检查语法、检测问题并强制代码风格。

- What type of modules does your project use?

  采用的 ES6 模块系统导入导出，选择 `JavaScript modules (import/export)` 。

- Which framework does your project use?

  选择 `React` 。

- Does your project use TypeScript?

  选择 `Yes` 后生成的 `eslint` 配置文件会给我们默认配上支持 `Typescript` 的 `parse` 以及插件 `plugins` 等。

- Where does your code run?

    `Browser` 和 `Node` 环境都选上，之后可能会编写一些 `node` 代码。

- What format do you want your config file to be in?

  选择 `JavaScript` ，即生成的配置文件是 js 文件，配置更加灵活。

- Would you like to install them now with npm?

  当然 `Yes` 了～

在漫长的安装结束后，项目根目录下多出了新的文件 `.eslintrc.cjs` ，这便是我们的 `eslint` 配置文件了。其默认内容如下：

### 配置语法

```js
module.exports = {
  parser: {},  // 解析器
  extends: [], // 继承的规则 [扩展]
  plugins: [], // 插件
  rules: {}    // 规则
};
```

```javascript
module.exports = {
    env: {
        browser: true,
        es2021: true,
        node: true
    },
    extends: [
        "eslint:recommended",
        "plugin:react/recommended",
        "plugin:@typescript-eslint/recommended",
    ],
    overrides: [],
    parser: "@typescript-eslint/parser",
    parserOptions: {
        ecmaVersion: "latest",
        sourceType: "module",
    },
    plugins: ["react", "@typescript-eslint"],
    /*
     * "off" 或 0    ==>  关闭规则
     * "warn" 或 1   ==>  打开的规则作为警告（不影响代码执行）
     * "error" 或 2  ==>  规则作为一个错误（代码不能执行，界面报错）
     */
    rules: {
        'react/react-in-jsx-scope': 'off',
        'no-console': 'error', // 禁止使用console
        'no-unused-vars': 'error',// 禁止定义未使用的变量
        'no-debugger': 'error', // 禁止使用debugger
        'no-var': 'error', // 要求使用 let 或 const 而不是 var
    },
};
```

### Prettier和ESLint冲突问题

安装插件  eslint-plugin-prettier eslint-config-prettier 

```bash
npm install eslint-plugin-prettier eslint-config-prettier -D
```

prettier 添加到 extends的最后



## 5.ESLint问题说明

如果使用`yarn create vite`创建的项目，没有生成.eslintrc文件，则可以继续看`ESLint配置讲解`这一节课，如果当前创建的项目默认已经生成了eslintrc文件，则只需要看我们当前这节课即可，`ESLint配置讲解`这节课可以跳过。

### 通过yarn create vite后，自动生成了eslint，我还需要在安装eslint吗

不需要了，使用框架默认eslint即可，只需要配置rules规则



### .eslintrc.cjs报错

在eslintrc.cjs文件中，找到env对象，添加：node: true，设置node环境即可

> 如果你的文件不报错，可以忽略。

### .tsconfig.json报错

```js
"moduleResolution": "bundler",
```

Typescript发布5.0以后，默认模块的解析策略是bundler，是一种兼容妥协方案，当前只有vscode最新版本的软件是支持的，比如：1.78，所以我们需要升级vscode软件即可，当然，我们也可以修改它，把bundler修改为node，因为以前使用最多的是node解析。

参考文档：https://zhuanlan.zhihu.com/p/617501026

### .tsconfig.node.json报错

报错原因同上，只需要修改moduleResolution的值为node即可，或者升级vscode软件。

### main.tsx文件报错

`An import path cannot end with a '.tsx' extension. Consider importing 'components/ui/Card/Card' instead.ts(2691)`



是因为新版本在导入模块时，可以不添加`.tsx`扩展名，我们删除即可。

比如：

`import App from './App.tsx'`修改为：`import App from './App'`



### main.tsx React导入时报错

需要在tsconfig.json文件中添加`"allowSyntheticDefaultImports": true,` 并删除 `allowImportingTsExtensions`配置，或者直接升级到最新版本的vscode软件。

