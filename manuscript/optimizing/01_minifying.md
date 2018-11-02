# Минификация

Начиная с webpack 4, сборка для продакшена по умолчанию минифицируется с помощью UglifyJS. Тем не менее, хорошо понимать технику и дальнейшие возможности.

## Минификация JavaScript

Цель **минификации** состоит в преобразовании кода в сжатую форму. Безопасные **преобразования** делают это без потери смысла, переписывая код. В качестве хороших примеров применения этой техники можно назвать переименование переменных или даже удаление целых блоков кода, если они в конечном итоге не будут выполнены (`if (false)`).

Небезопасные преобразования могут нарушить работу кода, поскольку они могут удалить что-то неявное, на что опирается основный код. К примеру, Angular 1 при использовании модулей ожидает определённое именование параметра функции. Переименование параметров нарушит корректное выполнение кода до тех пор, пока не будут приняты меры предосторожности.

### Изменение процесса минификации JavaScript

В webpack 4 процесс минификации управляется через два поля конфигурации: флаг `optimization.minimize` для переключения минификации и массив `optimization.minimizer` для настройки этого процесса.

Для настройки этих значений по умолчанию, мы добавим в проект пакет [uglifyjs-webpack-plugin](https://www.npmjs.com/package/uglifyjs-webpack-plugin), чтобы его впоследствии настроить.

Сначала установите плагин в проект:

```bash
npm install uglifyjs-webpack-plugin --save-dev
```

{pagebreak}

Теперь, чтобы добавить его к конфигурации, сперва определите для него часть кода:

**webpack.parts.js**

```javascript
const UglifyWebpackPlugin = require("uglifyjs-webpack-plugin");

exports.minifyJavaScript = () => ({
  optimization: {
    minimizer: [new UglifyWebpackPlugin({ sourceMap: true })],
  },
});
```

А теперь присоедините его к конфигурации продакшена:

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

Если сейчас выполнить `npm run build`, то вы увидите результат, похожий на тот, что был раньше. Конечный результат у вас может быть, возможно, немного лучше за счёт использования более новой версии UglifyJS.

T> Карты кода отключены по умолчанию, соответственно включить их можно через флаг `sourceMap`. Вам следует посмотреть на страницу с *uglifyjs-webpack-plugin* для получения дополнительных параметров.

T> Для удаления вызовов `console.log` из исходных файлов в конечном результате, установите опцию `uglifyOptions.compress.drop_console` на значение `true`, как [обсуждается в вопросе на Stack Overflow](https://stackoverflow.com/questions/49101152/webpack-v4-remove-console-logs-with-webpack-uglify).

{pagebreak}

## Другие способы минификации JavaScript

Хотя по умолчанию и *uglifyjs-webpack-plugin* работают для текущего варианта использования, существуют еще несколько вариантов, которые стоит рассмотреть:

* Пакет [babel-minify-webpack-plugin](https://www.npmjs.com/package/babel-minify-webpack-plugin) полагается внутри на [babel-preset-minify](https://www.npmjs.com/package/babel-preset-minify), разработанный командой Babel. Однако он медленнее, чем UglifyJS.
* Пакет [webpack-closure-compiler](https://www.npmjs.com/package/webpack-closure-compiler) работает параллельно и в результате иногда выдаёт ещё меньше по размеру, чем пакет *babel-minify-webpack-plugin*. Другим возможным вариантом является пакет [closure-webpack-plugin](https://www.npmjs.com/package/closure-webpack-plugin).
* Пакет [butternut-webpack-plugin](https://www.npmjs.com/package/butternut-webpack-plugin) в своей основе использует экспериментальный минификатор Рича Харриса (Rich Harris) [butternut](https://www.npmjs.com/package/butternut).

## Ускорение выполнения JavaScript

Специальные решения позволяют предварительно обрабатывать код с целью, чтобы он работал быстрее. Они дополняют технику минификации и могут быть разделены на **подъем области видимости (*scope hoisting)**, **предварительное выполнение (pre-evaluation)** и **улучшение анализа кода (improving parsing)**. Возможно, эти методы часто увеличивают общий размер сборки, позволяя ускорить выполнение.

### Поднятие области видимости

Начиная с webpack 4, по умолчанию применяется подъем области видимости в режиме продакшена. Поднимаются все модули в одну область видимости, вместо того написания каждый в отдельную анонимную функцию. Этот процесс замедляет сборку, но взаимен создаёт её более быстрой для выполнения. Прочитайте [эту статью о подъеме области видимости]((https://medium.com/webpack/brief-introduction-to-scope-hoisting-in-webpack-8435084c171f)) в блоге webpack.

T> Передайте `--display-optimization-bailout` в webpack для получения отладочной информации, связанной с результатами подъема.

### Предварительное выполнение

Пакет [prepack-webpack-plugin](https://www.npmjs.com/package/prepack-webpack-plugin) использует [Prepack](https://prepack.io/), ограниченный интерпретатор JavaScript. Он переписывает вычисления, которые могут выполняться во время компиляции и, следовательно, ускоряет выполнение кода. Смотрите также пакеты [val-loader](https://www.npmjs.com/package/val-loader) и [babel-plugin-preval](https://www.npmjs.com/package/babel-plugin-preval) в качестве альтернативных решений.

### Улучшение анализа кода

Пакет [optimize-js-plugin](https://www.npmjs.com/package/optimize-js-plugin) дополняет другие решения, обертывая функции c энергичной моделью вычислений, улучшая способ первоначального анализа JavaScript-кода. Плагин использует пакет [optimize-js](https://github.com/nolanlawson/optimize-js) Нолана Лоусона (Nolan Lawson).

## Минификация HTML

При использовании HTML-шаблонов в коде с использованием пакета [html-loader](https://www.npmjs.com/package/html-loader), их можно предварительно обработать их через [posthtml](https://www.npmjs.com/package/posthtml) с помощью пакета [posthtml-loader](https://www.npmjs.com/package/posthtml-loader). Можно использовать пакет [posthtml-minifier](https://www.npmjs.com/package/posthtml-minifier) для минификации HTML-кода.

## Минификация CSS

Загрузчик *css-loader* позволяет минифицировать CSS с помощью [cssnano](http://cssnano.co/). Включите явно опцию `minim`, если требуется минификация. Вы также можете передать [специфичные для cssnano опции](http://cssnano.co/optimisations/) для дальнейшей настройки поведения.

Пакет [clean-css-loader](https://www.npmjs.com/package/clean-css-loader) позволяет использовать популярный CSS-минификатор [clean-css](https://www.npmjs.com/package/clean-css).

Пакет [optimize-css-assets-webpack-plugin](https://www.npmjs.com/package/optimize-css-assets-webpack-plugin) — плагин для минификации CSS-ресурсов с возможностью указать собственный минификатор (по умолчанию используется *cssnano*). Использование пакета *mini-css-extract-plugin* может привести к дублированию CSS, поскольку он только объединяет текстовые фрагменты. Пакет *optimize-css-assets-webpack-plugin* устраняет эту проблему последующей обработкой полученного результата, и, следовательно, может привести к лучшему результату.

### Настройка CSS-минификации

Из доступных решений плагин *optimize-css-assets-webpack-plugin* объединяет всё лучшее из них. Для подключения его к настройке, сначала установите его и [cssnano](http://cssnano.co/):

```bash
npm install optimize-css-assets-webpack-plugin cssnano --save-dev
```

{pagebreak}

Как и случае с минификацией JavaScript, вы можете обернуть настройку плагина в часть конфигурации:

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

W> При использовании вывода webpack в формате JSON (с помощью опции `--json`), как описано в главе *Анализ сборки*, вам следует сконфигурировать настройку, использовав `canPrint: false` для плагина. 

{pagebreak}

Теперь присоединитесь эту часть к основной конфигурации:

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
      // Выполнение cssnano в безопасном режиме для предотвращения
      // потенциальных небезопасных преобразований кода.
      safe: true,
    },
  }),
leanpub-end-insert
  ...
]);
```

Если сейчас собрать проект (`npm run build`), вы заметите, что CSS стал меньше, поскольку были удалены комментарии:

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

T> Пакет [compression-webpack-plugin](https://www.npmjs.com/package/compression-webpack-plugin) позволяет отбросить проблему создания сжатых файлов в webpack для возможного сохранения времени обработки на сервере.

## Минификация изображений

Размер изображения можно уменьшить с помощью пакетов [img-loader](https://www.npmjs.com/package/img-loader), [imagemin-webpack](https://www.npmjs.com/package/imagemin-webpack) и [imagemin-webpack-plugin](https://www.npmjs.com/package/imagemin-webpack-plugin). В основе перечисленных пакетов используются оптимизаторы изображений.

Может быть хорошей идеей вместе с ними использовать пакеты *cache-loader* и *thread-loader*, как описано в главе *Производительность*, учитывая, что они могут быть значительными операциями.

## Резюме

Минимизация - это самый доступный шаг, который вы можете предпринять, чтобы сделать вашу сборку меньше. Подведём итоги:

* **Процесс минификации** анализирует исходный код и превращает его в меньший по размеру код, без потери функциональных возможностей, при условии, если используются безопасные преобразования. Конкретные небезопасные преобразования позволяют достичь еще меньших по размеру результатов при потенциальном нарушении кода, если он, например, зависит от точные имён параметров.
* Webpack выполняет минимизацию в режиме продакшена, используя по умолчанию UglifyJS. Другие решения, такие как *babel-minify-webpack-plugin*, обеспечивают аналогичную функциональность с издержками.
* Помимо JavaScript, можно также миницифировать другие ресурсы, включая CSS, HTML и изображения. Для их минификации требуются определённые технологии, которые должны применяться через загрузчики и собственные плагины.

В следующей главе вы научитесь применять tree shaking к коду.
