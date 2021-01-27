# Исключение неиспользуемого CSS

Фреймворки, такие как [Bootstrap](https://getbootstrap.com/), как правило, имеют много CSS. Часто вы используете только небольшую часть. Обычно вы включаете в сборку даже неиспользуемый CSS. Однако можно исключить части CSS-кода, которые вы не используете.

[PurifyCSS](https://www.npmjs.com/package/purifycss) — это инструмент, который может достичь этого путем анализа файлов. Он просматривает ваш код и вычисляет, какие CSS-классы используются. Часто для этого достаточно информации, чтобы удалить неиспользованный CSS из вашего проекта. Он также работает с одностраничными приложениями в определенной степени.

[uncss](https://www.npmjs.com/package/uncss) — хорошая альтернатива PurifyCSS. Он работает через PhantomJS и выполняет свою работу иначе. Вы можете использовать сам uncss как плагин для PostCSS.

W> Вы должны быть осторожны, если используете CSS-модули. Вы должны использовать **белый список** связанных классов, как описано в [readme-файле к purifycss-webpack](https://github.com/webpack-contrib/purifycss-webpack#usage-with-css-modules).

## Настройка Pure.css

Чтобы сделать демонстрацию более реалистичной, давайте установим [Pure.css](http://purecss.io/), небольшой CSS-фреймворк, а также будем использовать его в проекте, чтобы вы могли видеть PurifyCSS в действии. Эти два проекта никоим образом не связаны, несмотря на похожее название.

```bash
npm install purecss --save
```

{pagebreak}

Чтобы проект знал о Pure.css, импортируем его через выражение `import`:

**src/index.js**

```javascript
leanpub-start-insert
import "purecss";
leanpub-end-insert
...
```

T> Выражение `import` работает, потому что webpack разрешает данную зависимость из поля `"browser": "build/pure-min.css",` в файле *package.json* пакета Pure.css благодаря опции [resolve.mainFields](https://webpack.js.org/configuration/resolve/#resolve-mainfields). Webpack попытается разрешить зависимости в возможных полях `browser` и `module`, прежде чем смотреть в `main`.

Вам также следует сделать демонстрационный компонент с использованием класса из Pure.css, это будет наша отправная точка:

**src/component.js**

```javascript
export default (text = "Hello world") => {
  const element = document.createElement("div");

leanpub-start-insert
  element.className = "pure-button";
leanpub-end-insert
  element.innerHTML = text;

  return element;
};
```

Если вы запустите приложение (`npm start`), «Привет, мир» должен выглядеть как кнопка.

![Стилизированный текст «Привет, мир»](images/styled-button.png)

Создание билда приложения (`npm run build`) должно следующий вывод:

```bash
Hash: 36bff4e71a3f746d46fa
Version: webpack 4.1.1
Time: 739ms
Built at: 3/16/2018 4:26:49 PM
     Asset       Size  Chunks             Chunk Names
   main.js  747 bytes       0  [emitted]  main
  main.css   16.1 KiB       0  [emitted]  main
index.html  220 bytes          [emitted]
...
```

Как вы можете видеть, размер файла CSS увеличился, и это можно исправить с помощью PurifyCSS.

## Включение PurifyCSS

Использование PurifyCSS может привести к значительной экономии. В примере проекта они очищают и минимизируют Bootstrap (140 kB) в приложении, используя ~40% его селекторов до ~35 кБ. Это большая разница.

{pagebreak}

[purifycss-webpack](https://www.npmjs.com/package/purifycss-webpack) позволяет достичь аналогичных результатов. Вам следует использовать `MiniCssExtractPlugin` вместе с ним для достижения наилучших результатов. Сначала установите его и помощник [glob](https://www.npmjs.org/package/glob):

```bash
npm install glob purifycss-webpack purify-css --save-dev
```

Вам также нужна конфигурация PurifyCSS, как показано ниже:

**webpack.parts.js**

```javascript
const PurifyCSSPlugin = require("purifycss-webpack");

exports.purifyCSS = ({ paths }) => ({
  plugins: [new PurifyCSSPlugin({ paths })],
});
```

Затем часть должна быть связана с конфигурацией. Очень важно, чтобы плагин использовался *после* `MiniCssExtractPlugin`; в противном случае он не работает:

**webpack.config.js**

```javascript
...
leanpub-start-insert
const path = require("path");
const glob = require("glob");
leanpub-end-insert

const parts = require("./webpack.parts");

leanpub-start-insert
const PATHS = {
  app: path.join(__dirname, "src"),
};
leanpub-end-insert

...

const productionConfig = merge([
  ...
leanpub-start-insert
  parts.purifyCSS({
    paths: glob.sync(`${PATHS.app}/**/*.js`, { nodir: true }),
  }),
leanpub-end-insert
]);
```

W> Порядок имеет значение. Перед очисткой должно произойти извлечение CSS.

Если сейчас вы выполните `npm run build`, вы увидите что-то похожее на это:

```bash
Hash: 36bff4e71a3f746d46fa
Version: webpack 4.1.1
Time: 695ms
Built at: 3/16/2018 4:29:54 PM
     Asset       Size  Chunks             Chunk Names
   main.js  747 bytes       0  [emitted]  main
  main.css   2.07 KiB       0  [emitted]  main
index.html  220 bytes          [emitted]
...
```

Размер стилей заметно уменьшился. Вместо 16k у вас сейчас примерно 2k. Разница была бы еще более значимой для более крупных CSS-фреймворков.

PurifyCSS поддерживает [дополнительные опции](https://github.com/purifycss/purifycss#the-optional-options-argument), включая `minify`. Вы можете включить их через поле `purifyOptions` при создании экземпляра плагина. Учитывая, что PurifyCSS не может выбрать все классы, которые вы всегда используете, вы должны использовать массив `purifyOptions.whitelist` для определения селекторов, которые он должен оставить не смотря ни на что.

W> Использование PurifyCSS приводит к потере карт кода CSS, даже если вы включили их с определенной конфигурацией загрузчика, из-за своей внутренней обработки.

{pagebreak}

### Отрисовка критического пути

Идея [критического пути отрисовки](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/) рассматривает производительность CSS с другой точки зрения. Вместо оптимизации размера, он оптимизирует порядок отрисовки и выделяет CSS **в верхней части страницы (above-the-fold)**. Результат достигается путем отрисовки страницы, а затем выяснения, какие правила необходимы для получения позитивного результата.

[webpack-critical](https://www.npmjs.com/package/webpack-critical) и [html-critical-webpack-plugin](https://www.npmjs.com/package/html-critical-webpack-plugin) реализуют эту технику в виде плагина `HtmlWebpackPlugin`. С пакетом [isomorphic-style-loader](https://www.npmjs.com/package/isomorphic-style-loader) можно получить тот же результат при использовании webpack и React.

[critical-path-css-tools](https://github.com/addyosmani/critical-path-css-tools) от Эдди Османи (Addy Osmani) перечисляет другие связанные инструменты.

## Заключение

Использование PurifyCSS может привести к значительному уменьшению размера файла. Это в основном ценно для статических сайтов, которые полагаются на массивный фреймворк CSS. Чем более динамичным становится сайт или приложение, тем сложнее его надежно анализировать.

В итоге:

* Исключить неиспользованный CSS можно с помощью PurifyCSS. Он выполняет статический анализ с источником.
* Эта функциональность может быть включена через *purifycss-webpack*, и плагин следует применять *после* `MiniCssExtractPlugin`.
* В лучшем случае PurifyCSS может исключить большинство, если не все, неиспользуемых CSS-правил.
* Критический путь отрисовки — это еще одна методика CSS, которая выделяет CSS в начало страницы. Идея состоит в том, чтобы сделать что-то как можно быстрее, вместо ожидания загрузки CSS.

В следующей главе вы узнаете об **autoprefix**. Включение этой возможности упрощает написание вручную сложных настроек CSS, предназначенных для работы со старыми браузерами.
