# Минификация

Начиная с webpack версии 4, вывод для продакшена пропускается через минификатор UglifyJS по умолчанию. Тем не менее, полезно понимать данную технику и дальнейшие ее возможности.

## Минификация JavaScript

Целью **минификации** является конвертирование кода в более сжатую форму. Безопасные **преобразования** позволяют это сделать без потери смысла вашего кода, путем его переписывания. Хорошим примером работы таких преобразований здесь будет изменение имен переменных, или даже удаление целых блоков кода, исходя из того, что они в данный момент не доступны (`if (false)`).

Небезопасные трансформации могут поломать ваш код, поскольку они могут упустить что-то неявное, на что опирается базовый код. Например, Angular 1 при использовании модулей ожидает специфические имена переменных функций. В таком случае, переписывание имен переменных сломает ваш код, если вы не примете меры предосторожности.

### Изменение процесса минификации JavaScript

В webpack версии 4, процесс минификации контролируется с помощью двух полей конфигурации: флага `optimization.minimize` для включения данной функции и массива `optimization.minimizer` для настройки самого процесса.

Для изменения стандартных настроек, мы добавим [uglifyjs-webpack-plugin](https://www.npmjs.com/package/uglifyjs-webpack-plugin) в проект, что бы сделать его настройку возможной.

Сперва, добавьте плагин в проект:

```bash
npm install uglifyjs-webpack-plugin --save-dev
```

{pagebreak}

Чтобы привязать его к конфигурации, сперва определите соответствующее поле:

**webpack.parts.js**

```javascript
const UglifyWebpackPlugin = require("uglifyjs-webpack-plugin");

exports.minifyJavaScript = () => ({
  optimization: {
    minimizer: [new UglifyWebpackPlugin({ sourceMap: true })],
  },
});
```

А теперь, подключите к конфигурации:

**webpack.config.js**

```javascript
const productionConfig = merge([
  parts.clean(PATHS.build),
leanpub-start-insert
  parts.minifyJavaScript(),
leanpub-end-insert
  ...
]);
```

Теперь, если вы выполните `npm run build`, вы увидите результат, близкий к тому, что мы имели ранее. Результат может быть немного лучше, поскольку вы, вероятно, используете более новую версию UglifyJS.

T> Карты кода отключены по умолчанию. Вы можете включить их с помощью флага `sourceMap`. За дополнительными опциями, вам следует обратиться к страничке плагина *uglifyjs-webpack-plugin*.

T> Чтобы убрать все вызовы `console.log` из вывода, установите `uglifyOptions.compress.drop_console` в значение `true` как уже [было описано на Stack Overflow](https://stackoverflow.com/questions/49101152/webpack-v4-remove-console-logs-with-webpack-uglify).

{pagebreak}

## Другие способы минификации JavaScript

Хотя минификация по умолчанию и плагин *uglifyjs-webpack-plugin* подходят для нашего случая, есть и другие варианты, которые вы можете рассмотреть:

* [babel-minify-webpack-plugin](https://www.npmjs.com/package/babel-minify-webpack-plugin) под капотом опирается на [babel-preset-minify](https://www.npmjs.com/package/babel-preset-minify) и он был выпущен командой разработчиков Babel. Однако, он медленнее чем UglifyJS.
* [webpack-closure-compiler](https://www.npmjs.com/package/webpack-closure-compiler) исполняется параллельно и дает в разы меньший размер исходного файла, по сравнению с *babel-minify-webpack-plugin*. В качестве альтернативы, можете рассмотреть [closure-webpack-plugin](https://www.npmjs.com/package/closure-webpack-plugin).
* [butternut-webpack-plugin](https://www.npmjs.com/package/butternut-webpack-plugin) под капотом использует экспериментальный минификатор Рича Харриса [butternut](https://www.npmjs.com/package/butternut).

## Ускорение выполнения JavaScript-кода

Определенные решения позволяют вам обработать код таким образом, что он начнет исполняться быстрее. Они дополняют минификацию, и подобные техники могут быть разделены на **подъем области видимости (scope hoisting)**, **предисполнение (pre-evaluation)** и **оптимизация кода для парсера**, встроенного в движок браузера. Вполне возможно, что эти техники приведут к увеличению размера вашей сборки, в то время как сам код будет исполняться быстрее.

### Подъем области видимости (scope hoisting)

Webpack версии 4 по умолчанию использует эту технику в режиме продакшена. Он выносит все модули в единую область видимости, вместо создания замыканий для каждого из них. Данная процедура требует больше времени на создание сброки, но позволяет коду выполняться быстрее. [Подробнее о технике подъема области видимости](https://medium.com/webpack/brief-introduction-to-scope-hoisting-in-webpack-8435084c171f) вы можете прочитать в блоге webpack.

T>  Вы можете передать webpack флаг `--display-optimization-bailout` что бы получить отладочную информацию, касаемую результатов подъема области видимости.

### Предисполнение (pre-evaluation)

[prepack-webpack-plugin](https://www.npmjs.com/package/prepack-webpack-plugin) использует [Prepack](https://prepack.io/), который частично выполняет ваш JavaScript-код. Он переписывает те вычисления, которые могут быть произведены во время компиляции, и таким образом, улучшает время выполнения вашего кода. В качестве альтернативы, вы можете обратить внимание на [val-loader](https://www.npmjs.com/package/val-loader) и [babel-plugin-preval](https://www.npmjs.com/package/babel-plugin-preval).

### Оптимизация кода для парсера

[optimize-js-plugin](https://www.npmjs.com/package/optimize-js-plugin) дополняет другие решения, путем приведения некоторых функций к такому виду, что браузер может их жадно загрузить (eager loading). Это приводит к улучшению начального парсинга JavaScript-кода. Плагин опирается на [optimize-js](https://github.com/nolanlawson/optimize-js), созданный Ноланом Лоусоном.

## Минификация HTML

Если вы используете [html-loader](https://www.npmjs.com/package/html-loader) для загрузки HTML-файлов в ваш код, вы можете обработать их инструментом [posthtml](https://www.npmjs.com/package/posthtml) с помощью [posthtml-loader](https://www.npmjs.com/package/posthtml-loader). В этом случае, можете использовать [posthtml-minifier](https://www.npmjs.com/package/posthtml-minifier) чтобы минифицировать HTML.

## Минификация CSS

Загрузчик *css-loader* позволяет минифицировать CSS, пропуская ваш код через [cssnano](http://cssnano.co/). Данная минификация должна быть включена явно с помощью опции `minimize`. Вы также можете передать [специфичные опции cssnano](http://cssnano.co/optimisations/) для дополнительной настройки поведения.

[clean-css-loader](https://www.npmjs.com/package/clean-css-loader) позволяет вам использовать популярный CSS минификатор [clean-css](https://www.npmjs.com/package/clean-css).

[optimize-css-assets-webpack-plugin](https://www.npmjs.com/package/optimize-css-assets-webpack-plugin) еще один минифактор CSS, однако он позволяет вам выбрать плагин, с помощью которого будет производиться минификация. Использование `MiniCssExtractPlugin` может привести к дублирующемуся CSS-коду, поскольку он всего-лишь обединяет куски кода. `OptimizeCSSAssetsPlugin` позволяет избежать этой проблемы, поскольку работает уже с готовым результатом, и поэтому может дать более хороший результат.

### Настройка минификации CSS

Среди доступных решений, `OptimizeCSSAssetsPlugin` справляется с этим лучше всего. Чтобы включить его в нашу сборку, сперва установите его и [cssnano](http://cssnano.co/) first:

```bash
npm install optimize-css-assets-webpack-plugin cssnano --save-dev
```

{pagebreak}

Как и для JavaScript, вы можете обернуть их настройки в часть конфигурации (??????):

**webpack.parts.js**

```javascript
const OptimizeCSSAssetsPlugin = require(
  "optimize-css-assets-webpack-plugin"
);
const cssnano = require("cssnano");

exports.minifyCSS = ({ options }) => ({
  plugins: [
    new OptimizeCSSAssetsPlugin({
      cssProcessor: cssnano,
      cssProcessorOptions: options,
      canPrint: false,
    }),
  ],
});
```

W> Если вы используете флаг `--json` для сборки, что обсуждается в главе *Анализ процесса сборки*, вам необходимо также указать плагину `canPrint: false`.

{pagebreak}

Далее объеденить эти настройки с основной конфигурацией:

**webpack.config.js**

```javascript
const productionConfig = merge([
  ...
  parts.minifyJavaScript(),
leanpub-start-insert
  parts.minifyCSS({
    options: {
      discardComments: {
        removeAll: true,
      },
      // Запускать cssnano в безопасном режиме, что бы
      // избежать потенциально небезопасных изменений.
      safe: true,
    },
  }),
leanpub-end-insert
  ...
]);
```

Если вы соберете проект сейчас (`npm run build`), вы заметите что CSS уменьшился, поскольку в нем теперь отсустсвуют комментарии:

```bash
Hash: f51ecf99e0da4db99834
Version: webpack 4.1.1
Time: 3125ms
Built at: 3/16/2018 5:32:55 PM
           Asset       Size  Chunks             Chunk Names
      chunk.0.js  162 bytes       0  [emitted]
      chunk.1.js   96.8 KiB       1  [emitted]  vendors~main
         main.js   2.19 KiB       2  [emitted]  main
leanpub-start-insert
        main.css    1.2 KiB       2  [emitted]  main
vendors~main.css   1.32 KiB       1  [emitted]  vendors~main
leanpub-end-insert
  chunk.0.js.map  204 bytes       0  [emitted]
  chunk.1.js.map    235 KiB       1  [emitted]  vendors~main
...
```

T> [compression-webpack-plugin](https://www.npmjs.com/package/compression-webpack-plugin) позволяет возложить проблему создания сжатых файлов на webpack, что потенциально уменьшит время обработки на стороне сервера.

## Минификация изображений

Размер изображений может быть уменьшен при помощи [img-loader](https://www.npmjs.com/package/img-loader), [imagemin-webpack](https://www.npmjs.com/package/imagemin-webpack), и [imagemin-webpack-plugin](https://www.npmjs.com/package/imagemin-webpack-plugin). Эти пакеты под капотом используют оптимизаторы изображений.

Учитывая то, что они могут производить довольно затратные операции, может быть хорошей идеей использование вместе с этими плагинами *cache-loader* и *thread-loader*, что обсуждается в главе *Производительность*.

## Резюме

Минификация это самый простой шаг на пути к уменьшению размера вашей сборки. В итоге:

* Процесс **минификации** анализирует ваш код и преобразует его в более краткую форму, сохраняя при этом его смысл, если вы используете безопасные преобразования. Определенные небезопасные преобразования позволяют вам уменьшить размер еще больше, однако, в тоже самое время могут сломать ваш код, поскольку они, например, могут изменять названия переменных и аргументов функций.
* По умолчанию webpack в режиме продакшена производит минификацию с помощью UglifyJS. Другие решения, такие как *babel-minify-webpack-plugin*, обеспечивают аналогичную функциональность.
* Кроме кода JavaScript, возможна минификация других ресурсов, таких как CSS, HTML или изображения. Но для их минификации необходимо использование опредленных технологий, которые могут быть применены с помощью плагинов и загрузчиков, которые умеют их использовать.

В следующей главе, вы узнаете как применять к вашему коду технику "встряхивание дерева" (tree shaking).
