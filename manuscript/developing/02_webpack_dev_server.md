# webpack-dev-server

Такие инструменты, как [LiveReload](http://livereload.com/) или [Browsersync](http://www.browsersync.io/) позволяют обновлять браузер по мере разработки, а также избежать полной перезагрузки страницы при изменении CSS. Есть возможность настроить Browsersync для работы с webpack с помощью [browser-sync-webpack-plugin](https://www.npmjs.com/package/browser-sync-webpack-plugin), но у webpack есть другие козыри в рукаве.

## Режим `watch` и *webpack-dev-server*

Первым шагом на пути к лучшему окружению разработки является использование webpack в его режиме **слежения**. Вы можете включить этот режим, передав в webpack параметр `--watch`. Например: `npm run build -- --watch`.

Когда этот режим включен, webpack отслеживает изменения, внесенные в ваши файлы, и пересобирает проект автоматически. *webpack-dev-server* (WDS) реализует режим слежения и идет еще дальше.

WDS - это сервер разработки, который работает **в памяти**. Это означает, что содержимое сборки не записывается в файлы, но сохраняется в памяти. Это различие важно понимать, когда вы пытаетесь отладить ваш код или стили.

По умолчанию, WDS обновляет содержимое страницы браузера автоматически пока вы разрабатываете приложение, так что вам не приходится это делать самому. Однако, он поддерживает и более продвинутую функцию webpack, **Горячую Замену Модулей** (HMR).

HMR позволяет обновлять состояние браузера без полной перезагрузки страницы, что делает эту функцию особенно удобной при использовании с библиотеками вроде React, в которой обновление страницы уничтожает состояние вашего приложения. Приложение *Горячая Замена Модулей* объясняет эту функцию детально.

WDS предоставляет интерфейс, который позволяет исправлять код на лету, однако для эффективной работы этой функции, вы должны реализовать данный интерфейс для клиентской части кода. Это довольно тривиальная задача для чего-то, вроде CSS, поскольку у него нет состояния, но решение этой задачи с JavaScript фреймворками и библиотеками гораздо сложнее.

## Запись файлов из WDS

Несмотря на то, что WDS который работает в памяти по умолчанию - это хорошее решение, в плане производительности, иногда лучше будет записывать файлы в вашу файловую систему. Это может быть необходимым, когда вы интегрируетесь с другим сервером, который ожидает присутствия файлов. [write-file-webpack-plugin](https://www.npmjs.com/package/write-file-webpack-plugin) позволяет вам это сделать.

W> Вам следует использовать WDS исключительно для разработки. Если вы хотите размещать(???) ваше приложение с помощью других стандартных решений, как например Apache или Nginx.

## Начало работы с WDS

Для того, что бы начать работу с WDS, сперва установите его:

```bash
npm install webpack-dev-server --save-dev
```

Как и прежде, даная команда сгенерирует для нас исполняемый файл в папке `npm bin`, и вы сможете запустить *webpack-dev-server* из нее. После запуска WDS, в вашем распоряжении окажется сервер разработки по адресу `http://localhost:8080`. Автоматическое обновление браузера тоже будет включено, хоть и на фундаментальном уровне(???).

{pagebreak}

## Привязка WDS к проекту

Для интеграции WDS в наш проект, определим npm скрипт для его запуска. Что бы следовать конвенциям npm, нужно назвать его *start*, как на примере ниже:

**package.json**

```json
"scripts": {
leanpub-start-insert
  "start": "webpack-dev-server --mode development",
leanpub-end-insert
  "build": "webpack --mode production"
},
```

T> WDS загружает вашу конфигурацию, как и сам webpack. Здесь работают одинаковые правила.

Теперь, если вы выполните *npm run start* или *npm start*, вы должны увидеть кое-что в терминале:

```bash
> webpack-dev-server --mode development

ℹ ｢wds｣: Project is running at http://localhost:8080/
ℹ ｢wds｣: webpack output is served from /
ℹ ｢wdm｣: Hash: eb06816060088d633767
Version: webpack 4.1.1
Time: 608ms
Built at: 3/16/2018 3:44:04 PM
     Asset       Size  Chunks                    Chunk Names
   main.js    338 KiB    main  [emitted]  [big]  main
index.html  181 bytes          [emitted]
Entrypoint main [big] = main.js
...
```

{pagebreak}

Сервер запущен, и если вы откроете `http://localhost:8080/` в вашем браузере, вы увидите кое-что знакомое:

![Hello world](images/hello_01.png)

Теперь, если вы попытаетесь изменить ваш код, вы увидите новый вывод в вашем терминале. Браузер тоже должен реагировать на изменение кода перезагрузкой страницы.

T> WDS попытается запустится на другом порту, в случае если стандартный уже занят. Вывод в терминале расскажет вам о том, на каком порту он запустится. Вы можете отладить данную ситуацию, с помощью команд, вроде `netstat -na | grep 8080`. В Unix системах, если что-то уже запущено на порту 8080, данная команда выведет сообщение.

T> В дополнение к режимам `production` и `development`, есть и третий режим, `none`, который отключает все и близок к поведению, который был в распоряжении пользователей webpack в версиях, предшествующих 4.

## Настройка WDS через конфигурацию webpack

Функциональность WDS может быть настроена через поле `devServer` в конфигурации webpack. Вы можете задать большинство настроек и через CLI, но управление ими через webpack это более приемлемый подход.

{pagebreak}

Включите дополнительный функционал, как показано ниже:

**webpack.config.js**

```javascript
...

module.exports = {
leanpub-start-insert
  devServer: {
    // Display only errors to reduce the amount of output.
    stats: "errors-only",

    // Parse host and port from env to allow customization.
    //
    // If you use Docker, Vagrant or Cloud9, set
    // host: options.host || "0.0.0.0";
    //
    // 0.0.0.0 is available to all network devices
    // unlike default `localhost`.
    host: process.env.HOST, // Defaults to `localhost`
    port: process.env.PORT, // Defaults to 8080
    open: true, // Open the page in browser
  },
leanpub-end-insert
  ...
};
```

After this change, you can configure the server host and port options through environment parameters (example: `PORT=3000 npm start`).

T> [dotenv](https://www.npmjs.com/package/dotenv) allows you to define environment variables through a *.env* file. *dotenv* allows you to control the host and port setting of the setup quickly.

T> Enable `devServer.historyApiFallback` if you are using HTML5 History API based routing.

## Enabling Error Overlay

WDS provides an overlay for capturing compilation related warnings and errors:

**webpack.config.js**

```javascript
module.exports = {
  devServer: {
    ...
leanpub-start-insert
    overlay: true,
leanpub-end-insert
  },
  ...
};
```

Run the server now (`npm start`) and break the code to see an overlay in the browser:

![Error overlay](images/error-overlay.png)

T> If you want even better output, consider [error-overlay-webpack-plugin](https://www.npmjs.com/package/error-overlay-webpack-plugin) as it shows the origin of the error better.

W> WDS overlay does *not* capture runtime errors of the application.

## Enabling Hot Module Replacement

Hot Module Replacement is one of those features that set webpack apart. Implementing it requires additional effort on both server and client-side. The *Hot Module Replacement* appendix discusses the topic in greater detail. If you want to integrate HMR to your project, give it a look. It won't be needed to complete the tutorial, though.

## Accessing the Development Server from Network

It's possible to customize host and port settings through the environment in the setup (i.e., `export PORT=3000` on Unix or `SET PORT=3000` on Windows). The default settings are enough on most platforms.

To access your server, you need to figure out the ip of your machine. On Unix, this can be achieved using `ifconfig | grep inet`. On Windows, `ipconfig` can be utilized. An npm package, such as [node-ip](https://www.npmjs.com/package/node-ip) come in handy as well. Especially on Windows, you need to set your `HOST` to match your ip to make it accessible.

{pagebreak}

## Making It Faster to Develop Configuration

WDS will handle restarting the server when you change a bundled file, but what about when you edit the webpack config? Restarting the development server each time you make a change tends to get boring after a while. The process can be automated as [discussed in GitHub](https://github.com/webpack/webpack-dev-server/issues/440#issuecomment-205757892) by using [nodemon](https://www.npmjs.com/package/nodemon) monitoring tool.

To get it to work, you have to install it first through `npm install nodemon --save-dev`. After that, you can make it watch webpack config and restart WDS on change. Here's the script if you want to give it a go:

**package.json**

```json
"scripts": {
  "start": "nodemon --watch webpack.config.js --exec \"webpack-dev-server --mode development\"",
  "build": "webpack --mode production"
},
```

It's possible WDS [will support the functionality](https://github.com/webpack/webpack-cli/issues/15) itself in the future. If you want to make it reload itself on change, you should implement this workaround for now.

{pagebreak}

## Polling Instead of Watching Files

Sometimes the file watching setup provided by WDS won't work on your system. It can be problematic on older versions of Windows, Ubuntu, Vagrant, and Docker. Enabling polling is a good option then:

**webpack.config.js**

```javascript
const path = require("path");
const webpack = require("webpack");

module.exports = {
  devServer: {
    watchOptions: {
      // Delay the rebuild after the first change
      aggregateTimeout: 300,

      // Poll using interval (in ms, accepts boolean too)
      poll: 1000,
    },
  },
  plugins: [
    // Ignore node_modules so CPU usage with poll
    // watching drops significantly.
    new webpack.WatchIgnorePlugin([
      path.join(__dirname, "node_modules")
    ]),
  ],
};
```

The setup is more resource intensive than the default, but it's worth trying out.

{pagebreak}

## Alternate Ways to Use *webpack-dev-server*

You could have passed the WDS options through a terminal. It's clearer to manage the options within webpack configuration as that helps to keep *package.json* nice and tidy. It's also easier to understand what's going on as you don't need to dig out the answers from the webpack source.

Alternately, you could have set up an Express server and use a middleware. There are a couple of options:

* [The official WDS middleware](https://webpack.js.org/guides/development/#using-webpack-dev-middleware)
* [webpack-hot-middleware](https://www.npmjs.com/package/webpack-hot-middleware)
* [webpack-isomorphic-dev-middleware](https://www.npmjs.com/package/webpack-isomorphic-dev-middleware)

There's also a [Node API](https://webpack.js.org/configuration/dev-server/) if you want more control and flexibility.

W> There are [slight differences](https://github.com/webpack/webpack-dev-server/issues/616) between the CLI and the Node API.

## Other Features of *webpack-dev-server*

WDS provides functionality beyond what was covered above. There are a couple of relevant fields that you should be aware of:

* `devServer.contentBase` - Assuming you don't generate *index.html* dynamically and prefer to maintain it yourself in a specific directory, you need to point WDS to it. `contentBase` accepts either a path (e.g., `"build"`) or an array of paths (e.g., `["build", "images"]`). The value defaults to the project root.
* `devServer.proxy` - If you are using multiple servers, you have to proxy WDS to them. The proxy setting accepts an object of proxy mappings (e.g., `{ "/api": "http://localhost:3000/api" }`) that resolve matching queries to another server. Proxy settings are disabled by default.
* `devServer.headers` - Attach custom headers to your requests here.

T> [The official documentation](https://webpack.js.org/configuration/dev-server/) covers more options.

## Development Plugins

The webpack plugin ecosystem is diverse, and there are a lot of plugins that can help specifically with development:

* [case-sensitive-paths-webpack-plugin](https://www.npmjs.com/package/case-sensitive-paths-webpack-plugin) can be handy when you are developing on case-insensitive environments like macOS or Windows but using case-sensitive environment like Linux for production.
* [npm-install-webpack-plugin](https://www.npmjs.com/package/npm-install-webpack-plugin) allows webpack to install and wire the installed packages with your *package.json* as you import new packages to your project.
* [react-dev-utils](https://www.npmjs.com/package/react-dev-utils) contains webpack utilities developed for [Create React App](https://www.npmjs.com/package/create-react-app). Despite its name, they can find use beyond React. If you want only webpack message formatting, consider [webpack-format-messages](https://www.npmjs.com/package/webpack-format-messages).
* [start-server-webpack-plugin](https://www.npmjs.com/package/start-server-webpack-plugin) is able to start your server after webpack build completes.

{pagebreak}

## Output Plugins

There are also plugins that make the webpack output easier to notice and understand:

* [system-bell-webpack-plugin](https://www.npmjs.com/package/system-bell-webpack-plugin) rings the system bell on failure instead of letting webpack fail silently.
* [webpack-notifier](https://www.npmjs.com/package/webpack-notifier) uses system notifications to let you know of webpack status.
* [nyan-progress-webpack-plugin](https://www.npmjs.com/package/nyan-progress-webpack-plugin) can be used to get tidier output during the build process. Take care if you are using Continuous Integration (CI) systems like Travis as they can clobber the output. Webpack provides `ProgressPlugin` for the same purpose. No nyan there, though.
* [friendly-errors-webpack-plugin](https://www.npmjs.com/package/friendly-errors-webpack-plugin) improves on error reporting of webpack. It captures common errors and displays them in a friendlier manner.
* [webpack-dashboard](https://www.npmjs.com/package/webpack-dashboard) gives an entire terminal based dashboard over the standard webpack output. If you prefer clear visual output, this one comes in handy.

{pagebreak}

## Conclusion

WDS complements webpack and makes it more friendly for developers by providing development oriented functionality.

To recap:

* Webpack's `watch` mode is the first step towards a better development experience. You can have webpack compile bundles as you edit your source.
* WDS can refresh the browser on change. It also implements **Hot Module Replacement**.
* The default WDS setup can be problematic on specific systems. For this reason, more resource intensive polling is an alternative.
* WDS can be integrated into an existing Node server using a middleware. Doing this gives you more control than relying on the command line interface.
* WDS does far more than refreshing and HMR. For example proxying allows you to connect it to other servers.

In the next chapter, you learn to compose configuration so that it can be developed further later in the book.
