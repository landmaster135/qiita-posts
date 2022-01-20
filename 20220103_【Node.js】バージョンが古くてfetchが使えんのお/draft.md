# はじまり
Node.jsのための環境を以下の記事のように立ち上げました。

https://zenn.dev/kinkinbeer135ml/articles/6369ee73dd1508

そして、そのまま、fetchを使ったスクリプトが動くかどうかを試してみたのですが、
何か自分のスクリプトとは違うエラー（Unexpected Identifierみたいな）が出現しました。

そこで調べてみてこんな記事を見つけました。

https://chaika.hatenablog.com/entry/2019/01/23/083000

そこには、importを使うスクリプトの拡張子は`.mjs`でなければならないと書いてあったのですが、
fetchはメジャーなモジュールなので、流石にそんなヤバい挙動を2年も引き摺っていないだろうと思いました。
恐らく、Node.jsのバージョンを上げれば直るだろうと思い、この記事を書きました。

今回、立ち上げた環境は、ubuntuです。

# 問題のやつ
これが出てきたエラー画面です。
![20220103_01.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/265324/84dd0a2b-4b0b-6ee9-6ec7-96e5dc37bfef.jpeg)

「SyntaxError: Unexpected Identifier」が出てきてから色々調べましたが、上述の記事くらいしか良さげなものが見つかりませんでした。fetchモジュールのファイルの中でエラー起きるとかマジですか。。。
なので、原因というものは見つかりませんでしたが、
このときに使用していたNode.jsのバージョンが「10.19.0」であり、2020-02-05にリリースされたものでした。
2022-01-03時点で、推奨版が「16.13.1」なのでバージョンアップすれば直るのではと思いました。

https://nodejs.org/ja/download/releases/

# モジュールのバージョンアップ
モジュールをバージョンアップする処理もDockerfileに加えました。

~~~bash:Dockerfile（バージョンアップしないバージョン）
FROM ubuntu:latest
RUN apt-get update
RUN apt-get install nodejs -y
RUN apt-get install npm -y
RUN update-alternatives --install /usr/bin/node node /usr/bin/nodejs 10

ADD package.json .
ADD index.js .
CMD npm run start
~~~

~~~bash:Dockerfile（バージョンアップするバージョン）
FROM ubuntu:latest
RUN apt-get update

# for install n
RUN apt install -y curl

# install old nodejs and npm
RUN apt-get install nodejs -y
RUN apt-get install npm -y

# for version up nodejs
RUN npm install -g n
RUN n 16.13.1

# uninstall old nodejs and npm
RUN apt-get purge nodejs npm -y

# for version up npm
RUN npm install -g npm
RUN npm update -g

# to avoid "npm ERR! Tracker "idealTree" already exists"
WORKDIR /usr/app
COPY ./ /usr/app

# module etc...
RUN npm install node-fetch --save

# not need "update-alternatives" if "n" package manager
# RUN update-alternatives --install /usr/bin/node node /usr/bin/nodejs 10

# to get files from local
ADD package.json .
ADD index.mjs .
ADD README.md .
# CMD npm run start
~~~

これで仮想マシンをBuildしたら動くのかと思いきや、またエラーが出てきてしまいました。

~~~javascript:index.js
import fetch from 'node-fetch';
~~~

どうやら、`.mjs`を使わないとimportできない問題は顕在中のようです。

https://iwb.jp/nodejs-esmodule-code-import-error/

しょうがないので拡張子を変えたら、ES6で動いていたスクリプトがやっと動きました！
fetchが動きました！　時間掛かった～！

~~~javascript:index.mjs
import fetch from 'node-fetch';
~~~

fetchモジュールの方は、mjsファイルにする方ではなく、package.jsonに`"type": "module"`を追加することで、index.jsの状態でも使えるようにしてるみたいです。

# おしまい
- すんごい困ったら新しいバージョンにしてみる。
- 新しいバージョンの方がエラーメッセージも分かりやすい。
- importしたファイルは.mjsファイルにする。
