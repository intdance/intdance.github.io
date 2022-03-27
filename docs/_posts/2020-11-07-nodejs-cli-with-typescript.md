---
layout: post
title: 使用 Typescript 开发 Nodejs 命令行工具
date: 2020-11-07
updateDate: 2020-11-09
categories: [ Nodejs ]
---

* content
{:toc}

本文记录了搭建基于 TypeScript 的 Nodejs 命令行开发环境的全过程。
<!-- more -->

## 为何使用TypeScript
首先，对于编写类库或者工具而言，使用 TypeScript 的最大好处就是其提供了类型机制，可以避免我们犯一些低级错误。

其次，配合编辑器(如 VS Code)，TypeScript 能提供强大的代码提示功能，我们不需要记忆很多API的具体使用，在编写代码时编辑器会自动进行提示。比如引入了 `http` 之后，输入 `http.` 就会提示可以使用的各个方法和属性，并给出详细的说明。

> 同是微软旗下，VS Code 具有非常强大便利的功能，强烈推荐使用 VS Code 进行 TypeScript 和 Nodejs 开发。

最后，使用 TypeScript 是大势所趋，很多大公司都在推 TypeScript，使用 TypeScript 开发，可以让我们对 TS 使用更加熟练。

## 初始化工程
建立命令行工具，需要先创建一个 npm 包。下文将使用 npm 工具来完成包的初始化和依赖的安装。

首先创建一个文件夹，然后运行初始化命令：
```bash
mkdir ts-node-demo && cd ts-node-demo
npm init
```

控制台会出现一系列提示, 按照需求输入即可，然后一路回车，完成之后输入 `yes` 。
```bash
package name: (typescript-cli) 
version: (1.0.0) 
description: a cli in typescript
entry point: (index.js) 
test command: 
git repository: 
keywords: CLI,TypeScript
author: YourName
license: (ISC) MIT
```
初始化之后本地文件夹会出现一个 `package.json` 文件。我们的 npm 包就已经初始化完成了。为了避免误发布，我们在 `package.json` 中做一个更改:

```diff
- private: false,
+ private: true,
```
## 初始化 Git
在当前目录下运行：
```bash
git init
```
然后在当前目录创建 `.gitignore` 文件,指定忽略 `node_modules` 文件夹：
```bash
node_modules/
lib/
```

## 引入 Node 类型
既然是开发 Nodejs 程序，为了获得合适的类型校验和代码提示，我们需要引入 Nodejs 的类型文件：
```bash
npm i -D @types/node
```

## 引入 typescript
```bash
npm i typescript
```

然后需要初始化 `tsconfig` 文件。

```bash
./node_modules/.bin/tsc --init
```
上述命令会在当前文件夹下面创建一个 `tsconfig` 文件，用来指导 TypeScript 进行编译。
在里面有非常多的配置项，并且有非常详细的解释，我们做两个更改来适配我们的项目：
```diff
+ "sourceMap": true,
+ "outDir": "lib",
+ "moduleResolution": "node",
```
- `"sourceMap": true` 指定生成 sourceMap，方便我们调试程序；
- `"outDir": "lib"` 指定 TypeScript 将编译结果输出到 `./lib` 文件夹；
- `"moduleResolution": "node"` 表明我们将使用 Nodejs 的方式去寻找模块。

然后在与 `compilerOptions` 平级的地方增加选项：
```diff
+ "include": [
+    "src/**/*"
+ ]
```
这表示我们只会编译 `src` 目录下的 `.ts` 文件。

## 编写代码
在当前目录下创建 `src` 文件夹，并创建 `index.ts`：
```
mkdir src && echo "console.log('Your cli is running.');" > src/index.ts
```
然后运行：
```bash
./node_modules/.bin/tsc 
```
可以发现在文件夹下出现了 `lib/` 目录，里面就是 `index.ts` 编译之后的 js 文件。

## 创建运行脚本
每次编译都需要引用 `node_modules` 里面的 `tsc` 命令，有些繁琐，有三种方法可以解决：
1. 全局安装 typescript 包：
```bash
npm i typescript -g
```
就可以直接使用 `tsc` 命令了。

2. 使用 `npx` 执行
`npx` 是 npm 提供的命令，其会自动下载对应的包并执行.
```bash
npx tsc
```

3. 创建 npm 脚本
在 `package.json` 中的 `script` 中增加一行脚本：  
```diff
{
    "script": {
+       "build": "tsc"
    }
}
```


这里我们采用第3种方法，写入脚本后可以执行：
```bash
npm run build
```
也会成功进行编译。

## 注册命令
开发 Nodejs 命令行工具，就是提供一个可以直接调用的命令，而不是使用下面这种方式执行文件：
```bash
node lib/index.js
```

我们想要的效果是执行一个命令就能调用我们的 js 文件。

首先在当前文件夹创建文件 `bin/node-cli-demo` :

```bash
mkdir bin && touch bin/node-cli-demo.js
```

然后在文件中写入以下内容：
```js
#!/usr/bin/env node
require('../lib/index.js');
```

npm 包注册的命令需要在 `package.json` 中进行声明，增加如下内容：
```diff
{
    "name": "typescript-cli",
    "version": "0.0.1",
+   "bin": {
+       "node-cli-demo": "./bin/node-cli-demo.js"
+   },
}
```
这表示当执行 `node-cli-demo` 这个命令时就去执行我们的 `./bin/node-cli-demo.js` 文件。

最后在当前目录调用 `npm link` ，这条命令会把我们本地注册的命令放到 Nodejs 安装目录的 `bin` 文件夹下。在安装 Nodejs 时系统将该文件夹添加到命令查找的路径中。所以我们就可以直接使用我们刚刚注册的命令：

```bash
node-cli-demo
// Your cli is running.
```

## 自动监听文件变动
我们希望每次更改了 `.ts` 文件之后，不必手动执行 `npm run build` 就能看到最新的效果，可以使用 typescript 的 `--watch` 选项，在 `package.json` 中的 `script` 中增加 `start` 命令：
```diff
{
    "script": {
+       "start": "tsc --watch"
    }
}
```

在当前目录下运行命令：
```bash
npm start
```

然后对 `src/index.ts` 文件做一些更改，另开一个控制台窗口，运行 `node-cli-demo`，会发现打印的内容已经更新了。
这样我们在开发时就只需要关注代码编写，而不用考虑编译的问题了。

接下来我们就可以在 `src` 文件里面写我们的具体代码了！

## 总结
使用 TypeScript 开发 Nodejs 命令行的流程如下：
1. 安装 typescript 并进行配置；
2. 在 `package.json` 中声明命令并使用 `npm link` 将其链接到全局命令中；
3. 使用 `tsc --watch` 自动监听文件变动并重新编译；
4. 运行注册过的命令，查看效果。

## 配置ESLint
使用 ESLint 校验我们的代码，可以避免一些低级错误。而 TypeScript 现在推荐采用 ESLint 来检查代码。我们可以为我们的工程配置 ESLint。

### 安装依赖
首先安装依赖：

```bash
npm i -D eslint @typescript-eslint/parser  @typescript-eslint/eslint-plugin
```

- @typescript-eslint/parser 是用来解析 TypeScript 代码的，类似于 @babel/parser；
- @typescript-eslint/eslint-plugin 是 ESLint 插件，用来配置具体的检查规则。

### 设置配置
在根目录下创建 `.eslintrc`, 写入以下内容：
```json
{
    "root": true,
    "parser": "@typescript-eslint/parser",
    "plugins": [
        "@typescript-eslint"
    ],
    "extends": [
        "eslint:recommended",
        "plugin:@typescript-eslint/eslint-recommended",
        "plugin:@typescript-eslint/recommended"
    ],
    "rules": {
        "no-console": "error"
    },
    // set eslint env
    "env": {
        "node": true
    }
}
```

- `root: true` 表示当前目录就是寻找 ESLint 的最终配置，这样 ESLint 就不会再往上层寻找配置；
- `parse` 指定了使用 `@typescript-eslint/parser` 来解析我们的 TypeScript 代码；
- `plugins` 指定使用的 ESLint 插件；
- `extends` 指定了额外使用的规则配置。插件指定了一系列自定义的规则配置，只有在 `extends` 中指定才会生效。
- `rules` 中可以扩展我们自己的规则。
- `env` 中可以指定我们代码运行的环境，这样就可以自动判断某些代码是不是有错误。比如上述配置了 `node: true` ，我们在使用 `require` 的时候就不会报错了。

### 运行校验命令
在 `package.json` 的 `script` 中写入：
```json
{
    "script": {
        "lint": "eslint ./src --ext .ts"
    }
}
```
上述命令指定了对 `./src` 目录下扩展名为 `.ts` 的文件进行校验。

然后运行 `npm run lint`，会发现控制台出现报错，我们的 ESLint 已经生效了。
> 配置 VSCode 的 ESLint 插件，编辑器在开发时就会自动提示有问题的代码，帮助我们编写符合规范的代码。

### 忽略某些文件
我们可以指定某些目录下的文件不进行校验,在当前目录下创建 `.eslintignore` ,类似 `.gitignore`,然后在里面写入需要忽略的目录或文件：
```
node_modules
```

至此，ESLint 也配置完成了！

以上就是搭建 Nodejs 命令行的 TypeScript 开发环境的全部内容了，希望能帮到大家~

欢迎关注我，后续会继续写更多优质博客与大家分享。同时，有任何问题，欢迎一起讨论~
