---
title: 参考 lerna 的方式拆分类单页应用
date: 2018-11-03 22:51:53
author: Jayden
tags:
---

## 前言

<!-- more -->

在公司的全日制游戏项目中，原本的项目结构是这样的：
```
.
├── README.md
├── build
├── config
├── package.json
├── src
│	  ├── assets
│	  ├── common
│	  ├── components
│	  ├── config
│	  ├── filters
│	  ├── lib
│	  ├── pages
│			├── 2bridge
│			├── 2chase
│			├── 3bridge
│			├── 3chase
│			├── 4chase
│			├── 5chase
│			├── bridge
│			├── chase
│			├── chaseOld
│			├── coin
│			├── dinosaur
│			├── klgo
│			├── machine
│			├── minigame
│			├── ramp
│			└── stage
│	  ├── store
│	  └── utils
├── static
└── steamGame
```

在 `src/pages/` 中，是每个独立的游戏项目，有独立的入口文件，但是部分通用的文件又在外部集合，包括静态资源。这种做法带来的坏处是不同的游戏之间耦合严重，开发和发布的时候构建项目的速度很慢（项目庞大），不能单独发布单个游戏，在开发的过程中新建游戏需要重复复制粘贴文件。导致开发的效率低下。最后决定参考 create-react-app 的做法，并且用类似 lerna 管理 npm 包的方式来控制单个游戏的打包。新的目录结构如下：
![](http://cdn.meishakeji.com/frontend-blog/img/structure.png)

其中 `create-steam-game` 和 `steam-game-scripts` 是参考 `create-react-app` 和 vue-cli 的做法。而单独打包更新游戏的命令集成在 `steam-game-scripts` 里面。`steam-game-assets` 则是部分通用的库。每个游戏拥有独立的 `package.json` ，然后每个游戏在games 独立成一个文件夹，可以各自引入依赖，避免相互之间的影响，并且在开发过程中可以单独开发单个游戏。

## create-steam-game

`create-steam-game` 基本参考 `create-react-app` 的做法。

全局安装后可以 通过 `create-steam-game gameName` 来创建新的游戏，会创建相应的游戏文件夹，并且安装 `steam-game-scripts`，然后调用 `steam-game-scripts` 中的 `init.js`，来安装相应的依赖和复制默认的文件，安装好的目录文件如下：

```
.
└── package.json
└── src
    ├── App.vue
    ├── assets
    ├── components
    ├── config
    ├── filters
    ├── lang
    ├── main.js
    ├── router.js
    ├── store.js
    ├── utils
    └── views
```

```js
function createSteamGame(name, packages) {
  /**
   1. 检查文件夹名是否规范
 */
  const rootPath = path.resolve(name);
  const gameName = path.basename(rootPath);
  packages = getPackageToInstall(packages);
  checkGameName(gameName);

  /**
   2. 判断文件夹是否存在并创建
  */
  fs.ensureDirSync(rootPath);

  /**
   3. 非空文件夹则退出
 */
  if (!emptyDirectory(rootPath)) {
    console.log(`The directory ${chalk.green(gameName)} is not empty.`);
    console.log();
    console.log('Either try using a new directory name, or remove the files.');
    process.exit(1);
  }

  /**
   4. 创建 package.json
 */
  const packageJson = {
    name: gameName,
    version: '0.1.0',
    private: true,
  };

  fs.writeFileSync(
    path.join(rootPath, 'package.json'),
    JSON.stringify(packageJson, null, 2) + os.EOL
  );

  const useYarn = shouldUseYarn();

  run(rootPath, gameName, packages, useYarn);
}
```

```js

function run(root, gameName, packages, useYarn) {
  const allDependencies = [
    'babel-polyfill',
    'vue',
    'vue-router',
    'vuex',
    'meisha-fe-watch',
    'katex',
    ...packages,
  ];

  const allDevDependencies = ['steam-game-scripts'];

  console.log('Installing packages. This might take a couple of minutes.');

  checkIfOnline(useYarn)
    .then(isOnline => {
      console.log(`Installing ${chalk.cyan(allDependencies.join(' '))}...`);
      console.log();

      return install(root, useYarn, allDependencies, isOnline).then(() =>
        install(root, useYarn, allDevDependencies, isOnline, true)
      );
    })
    .then(async () => {
      await executeNodeScript(
        {
          cwd: root,
          args: [],
        },
        [root, gameName],
        `
      var init = require('steam-game-scripts/scripts/init.js');
      init.apply(null, JSON.parse(process.argv[1]));
      `
      );
    })
    .catch(reason => {
      console.log();
      console.log('Aborting installation.');
      if (reason.command) {
        console.log(`  ${chalk.cyan(reason.command)} has failed.`);
      } else {
        console.log(chalk.red('Unexpected error. Please report it as a bug:'));
        console.log(reason);
      }
      console.log();

      // On 'exit' we will delete these files from target directory.
      const knownGeneratedFiles = ['package.json', 'yarn.lock', 'node_modules'];
      const currentFiles = fs.readdirSync(path.join(root));
      currentFiles.forEach(file => {
        knownGeneratedFiles.forEach(fileToMatch => {
          // This remove all of knownGeneratedFiles.
          if (file === fileToMatch) {
            console.log(`Deleting generated file... ${chalk.cyan(file)}`);
            fs.removeSync(path.join(root, file));
          }
        });
      });
      const remainingFiles = fs.readdirSync(path.join(root));
      if (!remainingFiles.length) {
        // Delete target folder if empty
        console.log(
          `Deleting ${chalk.cyan(`${gameName}/`)} from ${chalk.cyan(
            path.resolve(root, '..')
          )}`
        );
        process.chdir(path.resolve(root, '..'));
        fs.removeSync(path.join(root));
      }
      console.log('Done.');
      process.exit(1);
    });
}
```

安装好之后就可以进入开发了！

## steam-game-scripts

steam-game-scripts 的主要工作是负责搭建和打包项目。

```bash
steam-game-scripts start # 开发
steam-game-scripts start --gameName # 开发单个游戏
steam-game-scripts build --gameName # 打包单个游戏
steam-game-scripts test # 测试
```

由于我司是使用自建的发布系统进行项目的打包发布。于是需要在发布系统上在打包的时候添加自定义的 npm 执行命令。所以这里使用了 --options 的方式来支持单个游戏的打包。也可以搭建多个新游戏的项目仓库来分开打包。

```js
const scriptIndex = args.findIndex(
  x => x === 'build' || x === 'start' || x === 'test'
);
const script = scriptIndex === -1 ? args[0] : args[scriptIndex];
const nodeArgs = scriptIndex > 0 ? args.slice(0, scriptIndex) : [];

switch (script) {
  case 'build':
  case 'start':
  case 'init':
  case 'test': {
    const result = spawn.sync(
      'node',
      nodeArgs
        .concat(require.resolve('../scripts/' + script))
        .concat(args.slice(scriptIndex + 1)),
      { stdio: 'inherit' }
    );
    if (result.signal) {
      if (result.signal === 'SIGKILL') {
        console.log(
          'The build failed because the process exited too early. ' +
            'This probably means the system ran out of memory or someone called ' +
            '`kill -9` on the process.'
        );
      } else if (result.signal === 'SIGTERM') {
        console.log(
          'The build failed because the process exited too early. ' +
            'Someone might have called `kill` or `killall`, or the system could ' +
            'be shutting down.'
        );
      }
      process.exit(1);
    }
    process.exit(result.status);
  }
  default:
    console.log('Unknown script "' + script + '".');
    console.log('Perhaps you need to update steam-game-scripts?');
    break;
}
```

steam-game-scripts/scripts/start.js

```js
'use strict';
const WebpackDevServer = require('webpack-dev-server');
const webpack = require('webpack');
const webpackConfig = require('../build/webpack.dev.conf.js');
const createDevServerConfig = require('../build/webpackDevServer.config.js');
const FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin');
const portfinder = require('portfinder');
const config = require('../config');
const utils = require('../build/utils');
const HOST = process.env.HOST || '0.0.0.0';

portfinder.basePort = process.env.PORT || config.dev.port;
portfinder.getPort((err, port) => {
  if (err) {
  } else {
    const devServerConfig = createDevServerConfig(port);
    webpackConfig.plugins.push(
      new FriendlyErrorsPlugin({
        compilationSuccessInfo: {
          messages: [
            `Your application is running here: http://${
              config.dev.host
            }:${port}`,
          ],
        },
        onErrors: config.dev.notifyOnErrors
          ? utils.createNotifierCallback()
          : undefined,
      })
    );

    var compiler = webpack(webpackConfig);
    var devServer = new WebpackDevServer(compiler, devServerConfig);

    devServer.listen(port, HOST, err => {
      if (err) {
        return console.log(err);
      }
    });

    ['SIGINT', 'SIGTERM'].forEach(function(sig) {
      process.on(sig, function() {
        devServer.close();
        process.exit();
      });
    });
  }
});

```

未完待续...



