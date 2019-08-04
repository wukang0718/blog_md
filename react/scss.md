# react引用scss全局的变量 react scss-load 配置一个全局变量文件

### 需要一个sass-resources-loader 的插件
 ```
    npm install sass-resources-loader --save-dev
 ```
 > 下载太慢的话，也可以使用cnpm

### 修改config/webpack.config.js

 > 在getStyleLoaders函数后面加上sass-resources-loader的配置

 ```
    {
       test: sassRegex,
       exclude: sassModuleRegex,
       use: getStyleLoaders(
         {
           importLoaders: 2,
           sourceMap: isEnvProduction && shouldUseSourceMap,
           // includePaths: [path.join(__dirname, "/src/assets/scss/index.scss")]
         },
         'sass-loader'
       ).concat([
            {
                loader: "sass-resources-loader",
                options: {
                    resources: path.join(__dirname, "../src/assets/scss/index.scss")
                }
            }
       ]),
       // Don't consider CSS imports dead code even if the
       // containing package claims to have no side effects.
       // Remove this when webpack adds a warning or an error for this.
       // See https://github.com/webpack/webpack/issues/6571
       sideEffects: true,
    },
    {
      test: sassModuleRegex,
      use: getStyleLoaders(
        {
          importLoaders: 2,
          sourceMap: isEnvProduction && shouldUseSourceMap,
          modules: true,
          getLocalIdent: getCSSModuleLocalIdent,
        },
        'sass-loader'
      ).concat([
        {
            loader: "sass-resources-loader",
            options: {
                resources: path.join(__dirname, "../src/assets/scss/index.scss")
            }
        }
      ]),
    },
 ```