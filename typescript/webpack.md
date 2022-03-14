# webpack

### 通常情况下，实际开发中我们都需要使用构建工具对代码进行打包，TS同样也可以结合构建工具一起使用，下边以webpack为例来实现如何构建工具使用TS

- 步骤

    1. 初始化项目

        - 进入根目录， 执行命令 ```npm init -y ```
          - 主要作用：创建package.json文件

    2. 下载构建工具

        - ```npm i -D webpack webpack-cli webpack-dev-server typescript ts-loader clean-webpack-plugin ....```
          - 安装所需要的包
            - webpack
              - 构建工具webpack
            - webpack-cli
              - webpack的命令行工具
            - webpack-dev-server
              - webpack的开发服务器
            - typescript
              - ts编译器
            - ts-loader
              - ts加载器，用于在webpack中编译ts文件
            - html-webpack-plugin
              - webpack中html插件，用来自动创建html文件
            - clean-webpack-plugin
              - webpack中的清除插件，每次构建都会先清除目录

    3. 根目录下创建webpack的配置文件webpack.config.js

        - ```javascript
           const path = require('path');
           const HtmlWebpackPlugin = require("html-webpack-plugin");
           const { CleanWebpackPlugin } = require("clean-webpack-plugin");

           module.exports = {
               optiomzation:{
                   minimize: false // 关闭代码压缩，可选
               },

               // 项目主文件路口
               entry: "./src/index.ts", 

               // 一种提供源代码到构建代码映射技术
               devtool: "inline-source-map", 

               // 项目打包后输出所在文件位置
               output: {
                   // 输出路径，可以通过node.js的 path模块到resolve方法来匹配当前路径。__dirname是node.js的一个变量，代表当前文件的绝对路径
                   path: path.resolve(_dirname,"dist"), 
                   // 输出文件名
                   filename: 'bundle.js', 
                   environment: {
                       arrowFunction: false // 关闭webpack的箭头函数 可选
                   }
               },
               
               // 编译格式
               resolve: {
                   extensions:[".ts",".js"]
               },

               // 模式
               mode: "development",

               // 模块（loader）
               module:{
                   rules: [
                       {
                           test: /\.ts$/,
                           use: {
                               loader: "ts-loader"
                           },
                           exclude: /node_modules/
                       }
                   ]
               },

               // 插件
               plugins: [
                   new CleanWebpackPlugin(),
                   new HtmlWebpackPlugin({
                       template: './src/index.html'
                   })
               ]

           }
          ```
    
    4. 根目录下创建tsconfig.json，配置可以根据自己需求

        - ```json
            {
                "compilerOptions": {
                    "target": "ES2015",
                    "module": "ES2015",
                    "strict": true
                }
            }
          ```

    5. 修改package.json添加如下配置

        - ```json
            {
                "scripts": {
                    "buuld": "webpack",
                    "dev": "webpack serve --open"
                }
            }
          ```
    
    6. 在src下创建ts文件，并在执行 ```npm run build```对代码进行编译，或者执行```npm run dev```来启动项目

### Babel

- 经过一系列对配置，使得TS和webpack已经结合到一起了，除了webpack，开发中还经常需要结合babel来对代码进行转换

    1. 安装依赖包：
       - ```npm i -D @babel/core @babel/preset-env babel-loader core-js```
       - 共安装了4个包
         - @babel/core
           - babel的核心工具
         - @babel/preset-env
           - babel的预定义环境
         - @babel-loader
           - babel在webpack中的加载器
         - core-js
           - core-js用来使用老版本的浏览器支持新版ES语法
    
    2. 修改webpack.config.js配置文件：
       
        -  ```javascript
            module: {
                rules: [
                    {
                        test:/\.ts$/,
                        use: [
                            {
                                loader:"babel-loader",
                                options: {
                                    presets: [
                                        [
                                            "@babel/preset-env",
                                            {
                                                useBuiltIns: "usage",
                                                corejs: {
                                                    version: 3
                                                },
                                                targets: {
                                                    chrome: "60",
                                                    firefox: "60",
                                                    ie: "9",
                                                    safari: "10",
                                                    edge: "17"
                                                }
                                            }
                                        ]
                                    ]
                                }
                            },
                            {
                                loader: "ts-loader"
                            }
                        ],
                        exclude: /node_modules/
                    }
                ]
            }
           ```