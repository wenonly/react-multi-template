# react多页面模板

## 运行
```
  yarn start
```

## 打包
```
  yarn build
```

## 描述

### 版本
    create-react-app 版本 3.3.0

### webpack 配置
 1. 弹出 `webpack` 配置
 ```
 yarn eject
 ```
 2. 修改 `config/paths.js` 文件
 ```
 const glob = require('glob');

 ...

 // 获取指定路径下的入口文件
function getEntries(globPath) {
  const files = glob.sync(globPath),
    entries = {};
  files.forEach(function(filepath) {
      const split = filepath.split('/');
      const name = split[split.length - 2];
      entries[name] = './' + filepath;
  });
  return entries;
}

const entries = getEntries('src/**/index.js');

function getIndexJs() {
  const indexJsList = [];
  Object.keys(entries).forEach((name) => {
    const indexjs = resolveModule(resolveApp, `src/${name}/index`)
    indexJsList.push({
      name,
      path: indexjs
    });
  })
  return indexJsList;
}
const indexJsList = getIndexJs()

...

// config after eject: we're in ./config/
module.exports = {
  dotenv: resolveApp('.env'),
  appPath: resolveApp('.'),
  appBuild: resolveApp('build'),
  appPublic: resolveApp('public'),
  appHtml: resolveApp('public/index.html'),
  appIndexJs: indexJsList,           # +++
  appPackageJson: resolveApp('package.json'),
  appSrc: resolveApp('src'),
  appTsConfig: resolveApp('tsconfig.json'),
  appJsConfig: resolveApp('jsconfig.json'),
  yarnLockFile: resolveApp('yarn.lock'),
  testsSetup: resolveModule(resolveApp, 'src/setupTests'),
  proxySetup: resolveApp('src/setupProxy.js'),
  appNodeModules: resolveApp('node_modules'),
  publicUrl: getPublicUrl(resolveApp('package.json')),
  servedPath: getServedPath(resolveApp('package.json')),
  entries                          # +++
};
 ```
 3. 修改 `config/webpack.config.js` 文件
 ```
 // 配置入口
  const entry = {}
  paths.appIndexJs.forEach(e => {
    entry[e.name] = [
      isEnvDevelopment &&
        require.resolve('react-dev-utils/webpackHotDevClient'),
      e.path
    ].filter(Boolean)
  });

...

return {
    entry
}

...

filename: isEnvProduction
? 'static/js/[name]/[name].[contenthash:8].js'
: isEnvDevelopment && 'static/js/[name]/[name].bundle.js',
chunkFilename: isEnvProduction
? 'static/js/[name]/[name].[contenthash:8].chunk.js'
: isEnvDevelopment && 'static/js/[name]/[name].chunk.js',

...

plugins: [
// Generates an `index.html` file with the <script> injected.
...Object.keys(paths.entries).map((name) => {
return new HtmlWebpackPlugin(
    Object.assign(
    {},
    {
        inject: true,
        chunks: [name],
        template: paths.appHtml,
        filename: name + '.html',
    },
    isEnvProduction
        ? {
            minify: {
            removeComments: true,
            collapseWhitespace: true,
            removeRedundantAttributes: true,
            useShortDoctype: true,
            removeEmptyAttributes: true,
            removeStyleLinkTypeAttributes: true,
            keepClosingSlash: true,
            minifyJS: true,
            minifyCSS: true,
            minifyURLs: true,
            },
        }
        : undefined
    )
)
}),
....
]

// 注释下面这部分
// new ManifestPlugin({
//   fileName: 'asset-manifest.json',
//   publicPath: publicPath,
//   generate: (seed, files, entrypoints) => {
//     const manifestFiles = files.reduce((manifest, file) => {
//       manifest[file.name] = file.path;
//       return manifest;
//     }, seed);
//     const entrypointFiles = entrypoints.main.filter(
//       fileName => !fileName.endsWith('.map')
//     );

//     return {
//       files: manifestFiles,
//       entrypoints: entrypointFiles,
//     };
//   },
// }),
 ```
 4. 修改检测文件是否存在的代码 `scripts/build.js` `scripts/start.js`
 ```
 // Warn and crash if required files are missing
 if (!checkRequiredFiles([paths.appHtml, ...paths.appIndexJs.map(e => e.path)])) {
    process.exit(1);
 }
 ```

  ### 页面路径
  ```
  http://localhost:3000/page1.html
  http://localhost:3000/page2.html
  http://localhost:3000/page3.html
  ```

  ### 添加页面方法

  复制`src`下面目录，重命名，保持目录结构
  
 