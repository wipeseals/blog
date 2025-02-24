# blog

wipeseals のブログです。 url: [https://blog.wipeseals.me](https://blog.wipeseals.me)

[Zola](https://www.getzola.org/)を使用してビルドできるブログのリポジトリです。

## セットアップ

1. Zola をインストールします。インストール方法は[公式ドキュメント](https://www.getzola.org/documentation/getting-started/installation/)を参照してください。
2. このリポジトリをクローンします。

   ```sh
   git clone https://github.com/wipeseals/blog.git
   cd blog
   ```

## ビルドとプレビュー

以下のコマンドを実行して、ブログをビルドし、ローカルサーバーでプレビューします。

```sh
zola serve
```

ブラウザで[http://127.0.0.1:1111](http://127.0.0.1:1111)にアクセスすると、ブログをプレビューできます。

## デプロイ

ブログをビルドしてデプロイするには、以下のコマンドを実行します。

```sh
zola build
```

`public`ディレクトリに生成されたファイルをウェブサーバーにアップロードしてください。

## ライセンス

このプロジェクトは MIT ライセンスの下で公開されています。詳細は`LICENSE`ファイルを参照してください。
