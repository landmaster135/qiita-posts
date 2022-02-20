# はじまり

GitHub Actionsを構築する際に、SSH認証で引っ掛かったので、
あまり頻繁にシェルを触らない僕でも、本作業を楽に行えるように記事にしました。

# SSH鍵の生成

## 参考記事

この生成は、こちらの記事を参考にさせていただきました。

https://qiita.com/shobooon/items/70cc74cd20c8b5b4fe15

https://qiita.com/GakuNaitou/items/81dbbd3ea6211af71648

## 本記事の筆者からの補足

所有者のみに権限を付与する形でディレクトリを作る。

```shell
install -m 0700 -d ~/.ssh
```

`ssh-keygen`する。メアドはGitHubに登録してあるものが良いと書いてあったので、GitHubの通知用のメアドにしました。`52403447+`は何か勝手に付いてあったものです。付いていない方もいると思います。
（`ssh-keygen`入力時に、`-f ~/.ssh/id_rsa`のオプションを入れれば、この画面は出ないかも。）

```shell
ssh-keygen -t rsa -C 52403447+landmaster135@users.noreply.github.com
```

`Enter file in which to save the key(path)`と出てきますが、`~/.ssh/id_rsa`に相当する`path`がコマンドを実行したディレクトリと同じであれば、何も入力せずにEnterでOK。
パスフレーズも入力せずにEnterで一応鍵を生成できる。

```shell
+---[RSA 3072]----+
|===O++.+.  .     |
| oB.=Eo + o      |
|...X.  o +       |
|. + * . o..      |
|   * +  So.      |
|  . . .. +..     |
|      .  ooo     |
|      ..o = .    |
|      .o o .     |
+----[SHA256]-----+
```

これで生成完了。`ls`で確認して、秘密鍵と公開鍵の両方があるかどうかを確認する。

```shell
# ls
id_rsa  id_rsa.pub
```

# SSH鍵の内容をコピー

両方とも鍵があったので、参考記事の通り、`pbcopy`を行ってみる。
しかし、僕の場合は、動きませんでした。

```shell
# pbcopy < id_rsa
bash: pbcopy: command not found
```

僕はこの時、WindowsからDockerイメージ（Ubuntu）を起動して行っていたのですが、どうやら、pbcopyはUbuntuにないらしい？
なので、少し調べてみると、`xsel`というモジュールがある。今度はこれでやってみる。

```shell
# インストールして、
apt install xsel
# Let it xsel
xsel id_rsa --clipboard --input

xsel: Can't open display: (null)
: Inappropriate ioctl for device
```

しかし、できない。どうやら、Windowsのクリップボードにコピーするために何か通信の設定が必要みたいである。なので、WSL上では何か設定しなければならならないということが分かった。

面倒なので、普通に画面上で選択してコピーする。

# 生成した鍵をGitHub上に設定する。

以下は、こちらの記事を参考にしています。

https://zenn.dev/tokiya_horikawa/articles/215637f23c8407

## 秘密鍵の登録

![20220109_07.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/265324/c518ea2d-1631-70a2-4e76-37aee75c6c93.jpeg)

![20220109_08.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/265324/c27901a0-6a9e-de11-cb3a-ca967ec06957.jpeg)


## 公開鍵の登録

![20220109_09.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/265324/655435a1-1639-7dd7-60da-84758e10e1a9.jpeg)


# GitHub Workflowに、鍵認証の処理を書く。

```yaml

```


# おしまい

