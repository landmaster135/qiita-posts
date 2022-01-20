# はじまり

はじめてSeleniumを使って悪戦苦闘した熱が冷めない内に、成果を上げておきます。
Seleniumで、Office365から立ち上げるアプリの直リンクへアクセスした際に、Office365へのログインが求められた際に切り抜けたソースを貼っておきます。
ブラウザはChromeで行いました。

# ソース

コメントアウトしているところはなくても動きますが、Seleniumのコードの動作確認ではあったほうがより役立ったという感覚がありました。
画面遷移の度に10秒くらいは余裕を持っていたほうが良いと思います。driver.waitは試していませんが、他のソースにも転用するために`driver`に依存しないものにしました。
`waitTimeToDisplay`は、各々のアプリによって時間を設定していただければと思います。

```javascript
const { Builder, By } = require('selenium-webdriver');
const { it } = require('mocha');

const sleep = waitTime => new Promise( resolve => setTimeout(resolve, waitTime) );

const openAppViaO365 = async(driver, url, username, password, waitTimeToDisplay=40000) => {
    // const funcName = 'openAppViaO365';
        
    await driver.get(url);
    await driver.manage().window().maximize();
    // await console.log(funcName, "aaaaaaaaaaaaaaaaaa");
    await sleep(10000); 
    
    await driver.findElement(By.id("i0116")).sendKeys(username);
    await driver.findElement(By.id("idSIButton9")).click();
    // await console.log(funcName, "bbbbbbbbbbbbbbbbbb");
    await sleep(10000);
    
    await driver.findElement(By.id("i0118")).sendKeys(password);
    await driver.findElement(By.id("idSIButton9")).click();
    // await console.log(funcName, "cccccccccccccccccc");
    await sleep(10000);
    
    await driver.findElement(By.id("idBtn_Back")).click();
    // await console.log(funcName, "dddddddddddddddddd");
    await sleep(waitTimeToDisplay);
};

let driver;
describe("テスト", () => {
    before(() => {
        driver = new Builder().forBrowser("chrome").build();
    });

    after(() => {
        return driver.quit();
    });

    it(`Office365ログイン`, async () => {
        await openAppViaO365(driver, "https://~", "<username>", "<password>")
    });

});
```

# おしまい
同じテスト自動化ツールのCypressだとこのログインになかなか苦戦しましたが、Seleniumだと案外あっさりできました。
Cypressだと`it`ブロックが切り替わると表示されていた画面が閉じてしまったりもしますしね・・・
