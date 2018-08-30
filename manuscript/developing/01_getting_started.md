# Начало работы

Перед началом работы, удостоверьтесь что вы используете актуальную версию [Node](http://nodejs.org/). Вам следует использовать как минимум последнюю LTS (long-term support) версию. В книге, конфигурация webpack ориентирована на функции именно LTS Node. Так-же, в вашем терминале должны быть доступны команды `node` и `npm`. [Yarn](https://yarnpkg.com/) хорошая альтернатива npm, и тоже подходит для этого руководства.

Возможно использование более контролируемого окружения, благодаря решениям вроде [Docker](https://www.docker.com/), [Vagrant](https://www.vagrantup.com/) или [nvm](https://www.npmjs.com/package/nvm). Vagrant более требовательный к ресурсам вашего компьютера, поскольку опирается на виртуальную машину. Vagrant полезен при работе в команде: каждый разработчик может иметь одинаковое окружение, которое обычно близко к окружению на продакшене.

T> Завершенная конфигурация доступна на [GitHub](https://github.com/survivejs-demos/webpack-demo).

{pagebreak}

## Создание проекта

Для начала, вам следует создать для проекта папку с файлом *package.json* внутри. npm использует этот файл для управления зависимостями проекта. Вот базовые команды для этого:

```bash
mkdir webpack-demo
cd webpack-demo
npm init -y # опция -y создает *package.json*, но вы можете его опустить, если нуждаетесь в большем контроле
```

Вы можете вносить дополнительные изменения в сгенерированный *package.json* вручную, несмотря на то, что часть команд модифицирует этот файл для вас автоматически. Официальная документация объясняет [опции package.json](https://docs.npmjs.com/files/package.json) более детально.

T> В файле *~/.npmrc* вы можете прописать опции, которые будут использоваться по умолчанию командой `npm init`.

T> Это идеальный момент для настройки контроля версий с помощью [Git](https://git-scm.com/). Вы можете создавать коммит на каждый свой шаг, и тег на каждую главу. Таким образом вам будет проще переключаться между изменениями, если потребуется.

T> Примеры в книге были отформатированы с помощью [Prettier](https://www.npmjs.com/package/prettier), с включенными опциями `"trailingComma": "es5",` и `"printWidth": 68`, для того что бы сделать изменения более опрятными, и они могли вместится на странице.

## Установка Webpack

Несмотря на то, что webpack может быть установлен глобально (`npm install webpack -g`), во избежание ошибок, хорошей практикой является добавление webpack в зависимости вашего проекта. В последствии, вы будете управлять конкретной версией webpack, которая используется в проекте. Такой подход отлично работает с настройкой **Continuous Integration** (CI). Система CI может установить ваши локальные зависимости, скомпилировать проект с их использованием, и затем отправить изменения на сервер.

Для того, что бы добавить webpack в ваш проект, выполните:

```bash
npm install webpack webpack-cli --save-dev # можно использовать -D для краткости
```

После этого, вы должны увидеть webpack в вашем *package.json*, в разделе `devDependencies`. Помимо локальной установки пакета в папку *node_modules*, npm так же сгенерирует исполняемый файл.

T> Вы можете использовать опции `--save` и `--save-dev` для разделения зависимостей приложения, и зависимостей разработки. Первая опция устанавливает пакет, и записывает его в раздел `dependencies` файла *package.json*, в то время как вторая записывает в раздел `devDependencies`.

T> [webpack-cli](https://www.npmjs.com/package/webpack-cli) поставляется с дополнительным функционалом, включая команды `init` и `migrate` которые позволяют быстрее создавать новые конфигурации webpack и мигрировать со старых версий на более новые.

## Процесс выполнения Webpack

Вы можете получить точный путь к исполняемым файлам с помощью команды `npm bin`. Скорее всего, команда укажет на директорию *./node_modules/.bin*. Попробуйте запустить webpack из этой папки с помощью `node_modules/.bin/webpack` или другой подобной команды.

После выполнения команды, вы должны увидеть версию webpack, ссылку на руководство по интерфейсу командной строки и обширный список опций. Большинство из них не будут использоваться в нашем проекте, но, по крайней мере, хорошо знать, что webpack наполнен функциональностью.

```bash
$ node_modules/.bin/webpack
Hash: 6736210d3313db05db58
Version: webpack 4.1.1
Time: 88ms
Built at: 3/16/2018 3:35:07 PM

WARNING in configuration
The 'mode' option has not been set. Set 'mode' option to 'development' or 'production' to enable defaults for this environment.

ERROR in Entry module not found: Error: Can't resolve './src' in '.../webpack-demo'
```

Данный вывод говорит нам о том, что webpack не может найти исходный файл для компиляции. Так же, отсутствует параметр `mode`, который применяет специфичные настройки для окружений разработки или продакшена.

Чтобы получить краткое представление о том, что выводит webpack, нам необходимо сделать следующее:

1. Создать *src/index.js*, который будет содержать `console.log("Hello world");`.
2. Выполнить команду `node_modules/.bin/webpack --mode development`. Webpack обнаружит данный файл, поскольку следует конвенциям Node.
3. Исследуйте файл *dist/main.js*. Вы должны увидеть стандартный код начальной загрузки, который добавляет webpack. Он начинает выполнение вашего кода. Под ним вы должны увидеть кое-что знакомое.

T> Попробуйте выполнить команду с опцией `--mode production` и сравните результат.

{pagebreak}

## Настройка ресурсов

Что бы сделать сборку более привлекательной, мы можем добавить другой модуль в наш проект, и начать разработку маленького приложения:

**src/component.js**

```javascript
export default (text = "Hello world") => {
  const element = document.createElement("div");

  element.innerHTML = text;

  return element;
};
```

После этого, нам следует изменить оригинальный файл таким образом, что бы импортировать новый файл и отобразить наше приложение через DOM:

**src/index.js**

```javascript
import component from "./component";

document.body.appendChild(component());
```

Исследуйте вывод после сборки (`node_modules/.bin/webpack --mode development`). Вы должны увидеть оба модуля в сборке, которую webpack записал в папку `dist`.

Для того, чтобы было легче исследовать вывод, передайте параметр `--devtool false`. Webpack по умолчанию генерирует карты кода, основанные на функции `eval`, и данный параметр отключит это поведение. Дополнительную информацию вы найдете в главе *Source Maps*.

Остается только одна проблема. Как нам протестировать приложение в браузере?

## Настройка плагина *html-webpack-plugin*

Проблема может быть решена созданием файла *index.html*, который будет указывать на сгенерированный файл. Но вместо того, чтобы делать это самим, мы можем использовать плагин  и настройку webpack.

Для начала, установите *html-webpack-plugin*:

```bash
npm install html-webpack-plugin --save-dev
```

Что бы подключить этот плагин с помощью webpack, создайте файл конфигурации, с кодом показанным ниже:

**webpack.config.js**

```javascript
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  plugins: [
    new HtmlWebpackPlugin({
      title: "Webpack demo",
    }),
  ],
};
```

Теперь настройка закончена, и вы можете попробовать следующее:

1. Собрать проект с помощью `node_modules/.bin/webpack --mode production`. Вы можете попробовать и `development` режим.
2. Войти в папку нашей сборки с помощью `cd dist`.
3. Запустить сервер, с использованием `serve` (`npm i serve -g`) или другой подобной команды.
4. Исследовать результат в веб-браузере. Вы должны увидеть там кое-что знакомое.

![Hello world](images/hello_01.png)

T> **Висячие запятые** используются в примерах, поскольку это делает более очевидным сравнение изменений примеров нашего кода.

## Исследование вывода

Если вы выполните `node_modules/.bin/webpack --mode production`, вы увидите следующий вывод:

```bash
Hash: aafe36ba210b0fbb7073
Version: webpack 4.1.1
Time: 338ms
Built at: 3/16/2018 3:40:14 PM
     Asset       Size  Chunks             Chunk Names
   main.js  679 bytes       0  [emitted]  main
index.html  181 bytes          [emitted]
Entrypoint main = main.js
   [0] ./src/index.js + 1 modules 219 bytes {0} [built]
       | ./src/index.js 77 bytes [built]
       | ./src/component.js 142 bytes [built]
Child html-webpack-plugin for "index.html":
     1 asset
    Entrypoint undefined = index.html
       [0] (webpack)/buildin/module.js 519 bytes {0} [built]
       [1] (webpack)/buildin/global.js 509 bytes {0} [built]
        + 2 hidden modules
```

Этот вывод говорит нам о многом:

* `Hash: aafe36ba210b0fbb7073` - хэш нашей сборки. Вы можете использовать его, что бы сделать недействительными ресурсы, используя заполнитель((???)) `[hash]`. Хэширование рассматривается детально в главе *Добавление хэшей к именам файлов*.
* `Version: webpack 4.1.1` - версия webpack.
* `Time: 338ms` - время, затраченное на создание сборки.
* `main.js  679 bytes       0  [emitted]  main` - название сгенерированного ресурса; его размер; ID **фрагмента** которому он относится; информация о статусе, которая говорит нам о том, каким образом он был сгенерирован и имя фрагмента.
* `index.html  181 bytes          [emitted]` - другой сгенерированный ресурс, созданный во время во время сборки.
* `[0] ./src/index.js + 1 modules 219 bytes {0} [built]` - ID входного ресурса, его имя, размер, ID входного фрагмента и то, каким образом он был сгенерирован.
* `Child html-webpack-plugin for "index.html":` - это специфичный для какого-нибудь плагина вывод. В данном случае, *html-webpack-plugin* создает этот вывод самостоятельно.

Examine the output below the `dist/` directory. If you look closely, you can see the same IDs within the source.

T> In addition to a configuration object, webpack accepts an array of configurations. You can also return a `Promise` and eventually `resolve` to a configuration for example.

T> If you want a light alternative to *html-webpack-plugin*, see [mini-html-webpack-plugin](https://www.npmjs.com/package/mini-html-webpack-plugin). It does less but it's also simpler to understand.

{pagebreak}

## Adding a Build Shortcut

Given executing `node_modules/.bin/webpack` is verbose, you should do something about it. Adjust *package.json* to run tasks as below:

**package.json**

```json
"scripts": {
  "build": "webpack --mode production"
},
```

Run `npm run build` to see the same output as before. npm adds *node_modules/.bin* temporarily to the path enabling this. As a result, rather than having to write `"build": "node_modules/.bin/webpack"`, you can do `"build": "webpack"`.

You can execute this kind of scripts through *npm run* and you can use *npm run* anywhere within your project. If you run the command as is, it gives you the listing of available scripts.

T> There are shortcuts like *npm start* and *npm test*. You can run these directly without *npm run* although that works too. For those in a hurry, you can use *npm t* to run your tests.

T> To go one step further, set up system level aliases using the `alias` command in your terminal configuration. You could map `nrb` to `npm run build` for instance.

{pagebreak}

## `HtmlWebpackPlugin` Extensions

Although you can replace `HtmlWebpackPlugin` template with your own, there are premade ones like [html-webpack-template](https://www.npmjs.com/package/html-webpack-template) or [html-webpack-template-pug](https://www.npmjs.com/package/html-webpack-template-pug).

There are also specific plugins that extend `HtmlWebpackPlugin`'s functionality:

* [favicons-webpack-plugin](https://www.npmjs.com/package/favicons-webpack-plugin) is able to generate favicons.
* [script-ext-html-webpack-plugin](https://www.npmjs.com/package/script-ext-html-webpack-plugin) gives you more control over script tags and allows you to tune script loading further.
* [style-ext-html-webpack-plugin](https://www.npmjs.com/package/style-ext-html-webpack-plugin) converts CSS references to inlined CSS. The technique can be used to serve critical CSS to the client fast as a part of the initial payload.
* [resource-hints-webpack-plugin](https://www.npmjs.com/package/resource-hints-webpack-plugin) adds [resource hints](https://www.w3.org/TR/resource-hints/) to your HTML files to speed up loading time.
* [preload-webpack-plugin](https://www.npmjs.com/package/preload-webpack-plugin) enables `rel=preload` capabilities for scripts and helps with lazy loading, and it combines well with techniques discussed in the *Building* part of this book.
* [webpack-cdn-plugin](https://www.npmjs.com/package/webpack-cdn-plugin) allows you to specify which dependencies to load through a Content Delivery Network (CDN). This common technique is used for speeding up loading of popular libraries.
* [dynamic-cdn-webpack-plugin](https://www.npmjs.com/package/dynamic-cdn-webpack-plugin) achieves a similar result.

{pagebreak}

## Conclusion

Even though you have managed to get webpack up and running, it does not do that much yet. Developing against it would be painful. Each time you wanted to check out the application, you would have to build it manually using `npm run build` and then refresh the browser. That's where webpack's more advanced features come in.

To recap:

* It's a good idea to use a locally installed version of webpack over a globally installed one. This way you can be sure of what version you are using. The local dependency also works in a Continuous Integration environment.
* Webpack provides a command line interface through the *webpack-cli* package. You can use it even without configuration, but any advanced usage requires configuration.
* To write more complicated setups, you most likely have to write a separate *webpack.config.js* file.
* `HtmlWebpackPlugin` can be used to generate an HTML entry point to your application. In the *Multiple Pages* chapter you will see how to generate multiple separate pages using it.
* It's handy to use npm *package.json* scripts to manage webpack. You can use it as a light task runner and use system features outside of webpack.

In the next chapter, you will learn how to improve the developer experience by enabling automatic browser refresh.
