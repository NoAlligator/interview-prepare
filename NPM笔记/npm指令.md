# npm几种安装模式的区别

## `npm install X -g`

全局安装。

- **安装模块到全局**，不会在项目node_modules目录中保存模块包。
- 不会将模块依赖写入devDependencies或dependencies 节点。
- 运行 npm install 初始化项目时不会下载模块。

## `npm install X`

仅安装。

- 会把X包安装到node_modules目录中
- **不会修改package.json**
- 之后运行npm install命令时，不会自动安装X

## `npm install X --save`

作为运行时依赖安装。

- 会把X包安装到node_modules目录中
- 会在package.json的dependencies属性下添加X
- 之后运行npm install命令时，会自动安装X到node_modules目录中
- 之后运行npm install --production或者注明NODE_ENV变量值为production时，会自动安装msbuild到node_modules目录中,即是在线上环境运行时会将包安装

## `npm install X –save-dev`

作为开发依赖安装。

- 会把X包安装到node_modules目录中
- 会在package.json的devDependencies属性下添加X
- 之后运行npm install命令时，会自动安装X到node_modules目录中
- 之后运行npm install –production或者注明NODE_ENV变量值为production时，不会自动安装X到node_modules目录中