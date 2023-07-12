# Vue2 创建项目初始化记录

技术选型：vue、vuex、vue-router、elementui、scss、eslint

所选vscode插件：

>Vue VsCode Snippets :vue  代码片段快速生成
>
>Vetur： 代码高亮
>
>Prettier -Code formatter：格式化代码
>
>ESlint: 代码检查、规范
>
>Auto Close Tag:  自动关闭标签



# 1、创建项目

- 安装node   http://nodejs.cn/download/  
- 安装vue-cli  
- 目前vue-cli 已处于维护状态，现在官方推荐使用 [`create-vue`](https://github.com/vuejs/create-vue) 来创建基于 [Vite](https://cn.vitejs.dev/) 的新项目。 另外请参考 [Vue 3 工具链指南](https://cn.vuejs.org/guide/scaling-up/tooling.html) 以了解最新的工具推荐。
- vue-cli 官方文档：https://cli.vuejs.org/zh/guide/installation.html

```shell
npm install -g @vue/cli
// 输出版本号即为安装成功
vue -V
```

- 创建项目

  `vue ui`  :可以通过图形界面的方式建。

  `vue create  项目名`  ：可以通过命令行方式

- 命令行方式创建

  `vue create myproject`

  ```powershell
  Vue CLI v4.2.3
  ? Please pick a preset: (Use arrow keys)
   base-learn (babel, router, vuex)   //自己预设的
    first-demo (babel, router, vuex)//自己预设的
    demo (stylus, babel, router, vuex)//自己预设的
    BASE (babel, router, vuex)  //自己预设的
    Base-Project (node-sass, babel, router, vuex, eslint)//自己预设的
    default (babel, eslint)
  > Manually select features                                                                                                                              
  ```

  选择` Manually select features  ` 进行手动配置。 上面这些都是预设的，自己设置保存就会生成，所以选择手动

  选择依赖项：

  按空格可以进行选择，这里选择 

   Babel、Router、Vuex、CSS Pre-processors、Linter /Formatter

  ```shell
  Vue CLI v4.2.3
  ? Please pick a preset: Manually select features
  ? Check the features needed for your project:
   (*) Babel
   ( ) TypeScript
   ( ) Progressive Web App (PWA) Support
   (*) Router
  >(*) Vuex
   (*) CSS Pre-processors
   (*) Linter / Formatter
   ( ) Unit Testing
   ( ) E2E Testing  
  ```

  这里不使用history 路由模式，我们应该用 哈希模式:

  ```shell
  Vue CLI v4.2.3
  ? Please pick a preset: Manually select features
  ? Check the features needed for your project: Babel, Router, Vuex, CSS Pre-processors, Linter
  ? Use history mode for router? (Requires proper server setup for index fallback in production) (Y/n) n   
  ```

  选择使用`Scss`  预处理：

  ```shell
  Vue CLI v4.2.3
  ? Please pick a preset: Manually select features
  ? Check the features needed for your project: Babel, Router, Vuex, CSS Pre-processors, Linter
  ? Use history mode for router? (Requires proper server setup for index fallback in production) No
  ? Pick a CSS pre-processor (PostCSS, Autoprefixer and CSS Modules are supported by default):
    Sass/SCSS (with dart-sass)
  > Sass/SCSS (with node-sass)
    Less
    Stylus         
  ```

  格式化 选择`ESLint +Prettier`

  ```shell
  Vue CLI v4.2.3
  ? Please pick a preset: Manually select features
  ? Check the features needed for your project: Babel, Router, Vuex, CSS Pre-processors, Linter
  ? Use history mode for router? (Requires proper server setup for index fallback in production) No
  ? Pick a CSS pre-processor (PostCSS, Autoprefixer and CSS Modules are supported by default): Sass/SCSS (with node-sass)
  ? Pick a linter / formatter config:
    ESLint with error prevention only
    ESLint + Airbnb config
    ESLint + Standard config
  > ESLint + Prettier             
  ```

  继续：

  ```shell
  Vue CLI v4.2.3
  ? Please pick a preset: Manually select features
  ? Check the features needed for your project: Babel, Router, Vuex, CSS Pre-processors, Linter
  ? Use history mode for router? (Requires proper server setup for index fallback in production) No
  ? Pick a CSS pre-processor (PostCSS, Autoprefixer and CSS Modules are supported by default): Sass/SCSS (with node-sass)
  ? Pick a linter / formatter config: Prettier
  ? Pick additional lint features: (Press <space> to select, <a> to toggle all, <i> to invert selection)
  >(*) Lint on save
   ( ) Lint and fix on commit       
  ```

  接下来，选择使用配置文件，不要都写在package.json，这样会很乱。

  ```shell
  Vue CLI v4.2.3
  ? Please pick a preset: Manually select features
  ? Check the features needed for your project: Babel, Router, Vuex, CSS Pre-processors, Linter
  ? Use history mode for router? (Requires proper server setup for index fallback in production) No
  ? Pick a CSS pre-processor (PostCSS, Autoprefixer and CSS Modules are supported by default): Sass/SCSS (with node-sass)
  ? Pick a linter / formatter config: Prettier
  ? Pick additional lint features: (Press <space> to select, <a> to toggle all, <i> to invert selection)Lint on save
  ? Where do you prefer placing config for Babel, ESLint, etc.? (Use arrow keys)
  > In dedicated config files
    In package.json                 
  ```

  可以选择是否保存。这个配置，下次创建项目的时候，直接用。

  ```shell
  ? Where do you prefer placing config for Babel, ESLint, etc.? In dedicated config files
  ? Save this as a preset for future projects? (y/N)     //自己确定
  ```

- 如果对于保存的预设不满意，或者以后保存预设多了，想删掉的话。去C:\Users\Administrator\\.vuerc   这个文件里面对应的预设json删掉即可

# 2、初始化配置

- 根目录创建vue.config.js 配置文件，以后在这里进行一些webpack的配置

```js
module.exports = {
    devServer:{
        port:8083,
        open:true  
    }
}
```

- eslint 格式化配置

vscode安装3个插件：

```
Vetur： 代码高亮
Prettier -Code formatter：格式化代码
ESlint: 代码检查、规范
# 还有比较好用的插件
Vue VSCode Snippets   //可以比较好自动生成代码
Auto  close Tag  //自动补全标签
```

打开vscode 文件——首选项——设置——扩展——eslint——打开settings.json

添加下面配置。

```json
 // 每次保存的时候将代码按eslint格式进行行行修复
    "eslint.autoFixOnSave": true,
    // 添加 vue ⽀支持
    "eslint.validate": [
        "javascript",
        "javascriptreact",
        {
            "language": "vue",
            "autoFix": true
        }
    ],
```

可⾃自定义eslint的⼀一些规则
在 `.eslintrc.js `中覆盖prettier规则即可，覆盖是为了了防⽌止冲突
在rules⾥里里配置  

## 2.1处理EsLint和 Prettier 格式化和校验冲突

解决方案： 配置prettier 和eslint 一致

根目录创建 `prettier.config.js` 文件

```js
module.exports = {
	// tab缩进大小,默认为2
	tabWidth: 2,
	// 使用tab缩进，默认false
	useTabs: true,
	// 使用分号, 默认true
	semi: false,
	// 使用单引号, 默认false(在jsx中配置无效, 默认都是双引号)
	singleQuote: true,
	// 行尾逗号,默认none,可选 none|es5|all
	// es5 包括es5中的数组、对象
	// all 包括函数对象等所有可选
	TrailingCooma: 'none',
	// 对象中的空格 默认true
	// true: { foo: bar }
	// false: {foo: bar}
	bracketSpacing: true,

	// 箭头函数参数括号 默认avoid 可选 avoid| always
	// avoid 能省略括号的时候就省略 例如x => x
	// always 总是有括号
	arrowParens: 'always',

	eslintIntegration: true
}

```

修改 `.eslintrc.js` 文件

```js
module.exports = {
	root: true,
	env: {
		node: true
	},
	extends: ['plugin:vue/essential', 'eslint:recommended', '@vue/prettier'],
	parserOptions: {
		parser: 'babel-eslint'
	},
	rules: {
		'no-console': process.env.NODE_ENV === 'production' ? 'warn' : 'off',
		'no-debugger': process.env.NODE_ENV === 'production' ? 'warn' : 'off',
		'space-before-function-paren': 0, // 忽略报错
		'prettier/prettier': [
			'error',
			{
				semi: false,
				singleQuote: true,
				// tab缩进大小,默认为2
				tabWidth: 2,
				// 使用tab缩进，默认false
				useTabs: true,
				// 是否使用分号, 默认true
				semi: false,
				// 使用单引号, 默认false(在jsx中配置无效, 默认都是双引号)
				singleQuote: true,
				// 行尾逗号,默认none,可选 none|es5|all
				// es5 包括es5中的数组、对象
				// all 包括函数对象等所有可选
				TrailingCooma: 'none',
				// 对象中的空格 默认true
				// true: { foo: bar }
				// false: {foo: bar}
				bracketSpacing: true,

				// 箭头函数参数括号 默认avoid 可选 avoid| always
				// avoid 能省略括号的时候就省略 例如x => x
				// always 总是有括号
				arrowParens: 'always'
			}
		]
	}
}

```

这样，你用prettier格式化以后，就不会被eslint校验出错。。 当然 也可以在seetings.json中进行配置.

# 3、配置scss全局变量量  

- **新建`_variable.scss ` 文件**

- - 在assets 文件夹下创建 scss文件夹

  - 在scss文件夹下创建`_varialbe.scss`文件

  - 在`_varialbe.scss`文件中声明一些全局变量统一管理

  - ```scss
    //内部定义文件，声明一些全局变量
    $theme-color:#33aef0
    ```

- **新建一个`reset.scss` 文件，初始化一些全局css配置**

```scss
html, body, div, span, applet, object, iframe,
h1, h2, h3, h4, h5, h6, p, blockquote, pre,
a, abbr, acronym, address, big, cite, code,
del, dfn, em, img, ins, kbd, q, s, samp,
small, strike, strong, sub, sup, tt, var,
b, u, i, center,
dl, dt, dd, ol, ul, li,
fieldset, form, label, legend,
table, caption, tbody, tfoot, thead, tr, th, td,
article, aside, canvas, details, embed,
figure, figcaption, footer, header, hgroup,
menu, nav, output, ruby, section, summary,
time, mark, audio, video {
	margin: 0;
	padding: 0;
	border: 0;
	font-size: 100%;
  font: inherit;
  vertical-align: baseline;
  box-sizing: border-box;
}
/* HTML5 display-role reset for older browsers */
article, aside, details, figcaption, figure,
footer, header, hgroup, menu, nav, section {
	display: block;
}
body {
	line-height: 1;
}
ol, ul {
	list-style: none;
}
blockquote, q {
	quotes: none;
}
blockquote:before, blockquote:after,
q:before, q:after {
	content: '';
	content: none;
}
a, a:hover{
  color: inherit;
  text-decoration: none;
}
table {
	border-collapse: collapse;
	border-spacing: 0;
}
html, body {
  width: 100%;
  height: 100%;
  background-color: #f5f5f5;
  font-family: 'PingFangSC-Light', 'PingFang SC', 'STHeitiSC-Light', 'Helvetica-Light', 'Arial', 'sans-serif';
}

// 公共样式
.fl{
  float: left;
}
.fr{
  float: right;
	.button-group-item{
		padding-left: 3px;
	}
}
//清除浮动
.clearfix{
  zoom:1;
  &:after{
    display:block;
    clear:both;
    content:"";
    visibility: hidden;
    height:0;
  }
}
```

- **在`vue.config.js` 配置文件中声明scss配置loader**

```js
  module.exports = {
    devServer: {
      port: 8083,
      open: true
    },
    // 配置scss全局变量量
    css: {
      loaderOptions: {
        sass: {
          data: `@import "@/assets/scss/_variable.scss";` //@import表示相对路径
        }
      }
    }
  };


  
```

```js
//注意哈，这里不同版本有区别
官方原版：
css: {
    loaderOptions: {
      sass: {
        //旧版sass-loader写法(8.0以下)
        data: `@import "@/assets/scss/_variable.scss";`,
      }
    }
}
修改后：
css: {
    loaderOptions: {
      sass: {
        //新版scss-loader(8.0及以上)
        prependData: `@import "@/assets/scss/_variable.scss";";`,
      }
    }
}

```



- **可以使用我们的全局属性了**。例如

```html

<style lang="scss" scoped>
#app {
  color: $theme-color;
}
</style>

```

遇到个问题： 当前下载的css-loader 是 8.1.2  ，结果报了个错

```shell
ValidationError: Invalid options object. Sass Loader has been initialized using an options object that does not match the API schema.
```

所以我把`css-loader` 的版本改为7.1.0 ，然后npm install  临时解决了这个问题。

- **使用 我们的 初始化的css**

```js
import Vue from "vue";
import App from "./App.vue";
import router from "./router";
import store from "./store";
import "@/assets/scss/reset.scss";  //引入它就起作用了。所有的页面初始化样式
Vue.config.productionTip = false;

new Vue({
  router,
  store,
  render: h => h(App)
}).$mount("#app");

```




