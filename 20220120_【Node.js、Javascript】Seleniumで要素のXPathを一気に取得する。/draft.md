# はじまり
一般的にDOMから要素のXPathを取得する方法は、ブラウザの開発者ツールからとされています。
しかし、Seleniumで要素にアクセスするためにいちいちDOMを右クリックしてXPathを取得するのも面倒だと思います。僕の場合は、300回右クリックしてXPathをメモするのは嫌だったので、一気にXPathを取得する方法を探しました。
今回は、そのときのソースを掲載します。

# ソース
今回の方法では、2つファイルを作り実現しました。
作るファイルは、ブラウザで実行させるスクリプトが入ったファイルと、普通にseleniumを実行するファイルです。
Chromeで実行し、対象のCSSクラス名は`checkbox`でした。

まず、ブラウザ側のファイルとなります。
最後の方は文法としては変ですが、配列のインデックスをスクリプトに載せるために`"scriptSeparator"`と記述しています。

```javascript:ブラウザ側のファイル.js
function getXpathByElement (element) {
    var NODE_TYPE_ELEMENT_NODE = 1;

    if (element instanceof Array) {
        element = element[0];
    }

    if (element.nodeType != NODE_TYPE_ELEMENT_NODE) {
        throw new ErrorException('nodes other than the element node was passed. node_type:'+ element.nodeType +' node_name:'+ element.nodeName);
    }

    var stacker = [];
    var node_name = '';
    var node_count = 0;
    var node_point = null;

    do {
        node_name = element.nodeName.toLowerCase();
        if (element.parentNode.children.length > 1) {
            node_count = 0;
            node_point = null;
            for (i = 0;i < element.parentNode.children.length;i++) {
                if (element.nodeName == element.parentNode.children[i].nodeName) {
                    node_count++;
                    if (element == element.parentNode.children[i]) {
                        node_point = node_count;
                    }
                    if (node_point != null && node_count > 1) {
                        node_name += '['+ node_point +']';
                        break;
                    }
                }
            }
        }
        stacker.unshift(node_name);
    } while ((element = element.parentNode) != null && element.nodeName != '#document');

    return '/' + stacker.join('/').toLowerCase();
}

return getXpathByElement(document.getElementsByClassName("checkbox")["scriptSeparator"]);
```

そして、seleniumを実行するファイルです。

```javascript:Selenium用ファイル.js
const { Builder, By } = require('selenium-webdriver');
const fs     = require("fs");
const _      = require("lodash");

const getTextLines = (fileName) => {
    const separator = '\n';
    let text = fs.readFileSync(`./${fileName}`, 'utf8');
    let lines = text.toString().split(separator);
    return lines;
}

const getXpathsByClassName = async(driver, className) => {
    let elements = [];
    let xpath    = '';
    let xpaths   = [];
    const scriptFileName = 'lib/getXpathByElement.js';
    let script  = await getTextLines(scriptFileName).join('\t');
    let scripts = await script.toString().split('\"scriptSeparator\"');
    elements = await driver.findElements(By.className(className));
    for (let i = 0; i < elements.length; i++){
        xpath = await driver.executeScript(`${scripts[0]}${i}${scripts[1]}`);
        xpaths.push(_.cloneDeep(xpath));
    }
    return new Promise( resolve => resolve(xpaths) );
};

let driver;
describe("テスト", () => {
    before(() => {
        driver = new Builder().forBrowser("chrome").build();
    });

    after(() => {
        return driver.quit();
    });

    it{(`URLを開いたり、対象の要素を画面上に表示する。`), async () => {

        // URLを開いたり、対象の要素を画面上に表示する処理

    }};

    it(`XPath取得`, async () => {
        xpathArray = await getXpathsByClassName(driver, 'checkbox');
    });
});

```

# 他に試した方法

## `driver.findElements(By.className(className))`で取得した要素を`executeScript`に渡す。

ブラウザ側で認識している要素とどうやら違うようで、返り値がすべて`null`になったので、失敗。

## ブラウザ側でXPathの配列を作る。

メモリ不足エラーでブラウザが止まり、失敗。

# おしまい

ブラウザ側のファイルはこちらを参考にしています。とても助かりました！

https://qiita.com/ProjectICKX/items/eb4a48598a15675897cb
