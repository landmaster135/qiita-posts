# はじまり

今回、Seleniumで疑似要素の`content`属性の値を取得するコードを作成しましたので掲載します。

本ソースは、OutSystemsで作られたシステムにおいて、チェックボックスのチェックの有無を判断するために使用したので、同様にOutSystemsを使っている方も利用できるのではないでしょうか。

動かしたブラウザは、Chromeです。

# ソース

```javascript
const { Builder } = require('selenium-webdriver');

const getPseudoElementsContentsByClassName = async(driver, className, isAfter=true) => {
    const funcName = 'getPseudoElementsContentsByClassName';
    let afterOrBefore;
    if(isAfter === true){
        afterOrBefore = ':after';
    }else{
        afterOrBefore = ':before';
    }
    let contents = await driver.executeScript(`const elements = document.querySelectorAll(\"${className}\"); let contents = []; for(let i = 0; i < elements.length; i++){contents.push(window.getComputedStyle(elements[i], \"${afterOrBefore}\").content);}; return contents;`);
    return new Promise( resolve => resolve(contents) );
}

let driver;
describe("テスト", () => {
    before(() => {
        driver = new Builder().forBrowser("chrome").build();
    });

    after(() => {
        return driver.quit();
    });

    it(`疑似要素after取得`, async () => {
        let contentsCheckboxAfter = await getPseudoElementsContentsByClassName(driver, '.checkbox', true);
        console.log(contentsCheckboxAfter);
    });
});
```

僕がこのソースを実行した結果が、`" "`と出力されているチェックボックスはチェックが入っていて、`null`はチェックが入っていないチェックボックスという並びでした。

```bash:出力例
[
  '" "',  '" "',  'none', 'none', 'none', 'none', 'none',
  'none', 'none', '" "',  'none', 'none', 'none', 'none',
  '" "',  'none', '" "',  'none', 'none', 'none', 'none',
  'none', 'none', 'none', 'none', 'none', 'none', 'none',
  'none', 'none', 'none', 'none', 'none', '" "',  '" "',
  'none', 'none', '" "',  'none', '" "',  'none', 'none',
  'none', 'none', 'none', 'none', 'none', 'none', 'none',
  'none', 'none', 'none', 'none', 'none', 'none', '" "',
  '" "',  'none', 'none', 'none', 'none', 'none', 'none',
  '" "',  '" "',  'none', 'none', 'none', 'none', 'none',
  'none', 'none', 'none', 'none', 'none', 'none', 'none',
  'none', 'none', 'none', '" "',  '" "',  '" "',  'none',
  '" "',  'none', 'none', '" "',  'none', 'none', 'none',
  'none', 'none', 'none', 'none', 'none', 'none', 'none',
  'none', 'none',
  ... 218 more items
]
```

# おしまい

`checked`属性ではなく、`::before`と`::after`の判定になることで、`executeScript`を使う羽目になりだいぶ面倒になりました・・・。
デザインを凝ると、その分テストが大変になるんですなあ。と思いました。
