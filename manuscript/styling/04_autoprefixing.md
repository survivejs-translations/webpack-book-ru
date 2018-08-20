# Автоматическое добавление браузерных префиксов

Может быть сложным вспомнить, какие префиксы браузеров вы должны использовать для определенных правил CSS, чтобы поддерживать большое количество пользователей. **Autoprefixing или автоматическое добавление браузерных префиксов** решает эту проблему. Он может быть включен через PostCSS и плагин [autoprefixer](https://www.npmjs.com/package/autoprefixer). *autoprefixer* использует сервис [Can I Use](http://caniuse.com/) для выяснения, какие правила должны быть с префиксом, и его поведение может быть настроено, как будет показано в этой главе. 

## Настройка автодобавления браузерных префиксов

Для включения этой возможности в текущую настройку нужно добавить небольшое дополнение. Сначала установите *postcss-loader* и *autoprefixer*:

```bash
npm install postcss-loader autoprefixer --save-dev
```

Затем добавьте фрагмент кода, включающий `autoprefixer`:

**webpack.parts.js**

```javascript
exports.autoprefix = () => ({
  loader: "postcss-loader",
  options: {
    plugins: () => [require("autoprefixer")()],
  },
});
```

{pagebreak}

Для подключения загрузчика с извлечением CSS, подключите его следующим образом:

**webpack.config.js**

```javascript
const productionConfig = merge([
  parts.extractCSS({
leanpub-start-delete
    use: "css-loader",
leanpub-end-delete
leanpub-start-insert
    use: ["css-loader", parts.autoprefix()],
leanpub-end-insert
  }),
  ...
]);
```

Чтобы проверить, что настройка работает, нам следует что-нибудь добавить, измените CSS для того, как показано ниже:

**app/main.css**

```css
...

leanpub-start-insert
.pure-button {
  -webkit-border-radius: 1em;
  border-radius: 1em;
}
leanpub-end-insert
```

Если вы знаете, какие браузеры вы предпочитаете поддерживать, можно создать файл [.browserslistrc](https://www.npmjs.com/package/browserslist). Различные инструменты учитывают конфигурацию, заданную в этом файле, включая *autoprefixer*.

T> Вы можете проверить CSS через [Stylelint](http://stylelint.io/). Его можно настроить таким же образом через *postcss-loader* как `autoprefixer` выше.

{pagebreak}

Настройте файл следующим образом:

**.browserslistrc**

```
> 1% # Browser usage over 1%
Last 2 versions # Or last two versions
IE 8 # Or IE 8
```

Если теперь вы запустите сборку приложения (`npm run build`) и изучите встроенный CSS, вы сможете найти там CSS без части webkit:

```css
...

leanpub-start-insert
.pure-button {
  border-radius: 1em;
}
leanpub-end-insert
```

*autoprefixer* может **удалить** ненужные правила, а также добавить правила, которые требуются на основе определения браузера.

## Заключение

Автодобавление браузерных префиксов — удобная техника, поскольку она уменьшает объем работы, необходимый при создании CSS. Вы можете поддерживать минимальные требования к браузеру в файле *.browserslistrc*. Этот инструмент может затем использовать эту информацию для генерации оптимального вывода (CSS).

В итоге:

* Автоматическое добавление браузерных префиксов можно включить через PostCSS-плагин *autoprefixer*.
* Автоматическое добавление браузерных префиксов дописывает недостающие определения CSS на основе минимального определения браузера.
* *.browserslistrc* — стандартный файл, который работает с другими инструментами, а не только с *autoprefixer*
