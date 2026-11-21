{% raw %}
# sources {#sources}

sources チェックは、クレートが信頼できる取得元からだけ来ていることを確かめます。

```bash
cargo deny check sources
```

## 使用例 - 既知の、または信頼できる取得元だけを許可する {#use-case---only-allowing-knowntrusted-sources}

Cargo は、さまざまな取得元からクレートを取得できます。具体的には、レジストリ、git リポジトリ、ローカルのファイルパスです。これは一般にとても便利で、開発にも柔軟に使えます。しかし、特に依存を git リポジトリに向け直す場合、プロジェクトが信頼しなければならない取得元が増えることを、リポジトリとしては明示的に同意したうえで受け入れたいことがあります。

このチェックを追加したきっかけについては、[Why npm lockfiles can be a security blindspot for injecting malicious modules](https://snyk.io/blog/why-npm-lockfiles-can-be-a-security-blindspot-for-injecting-malicious-modules/) を見てください。

## 使用例 - ベンダリングしたファイルの依存だけを使う {#use-case---only-using-vendored-file-dependencies}

プロジェクトによっては、ローカルのファイルの依存だけに対応したい場合があります。たとえば、完全に管理し、オフラインでビルドできるように、すべての依存をリポジトリにベンダリングしている場合です。これは、git とレジストリの取得元をすべて禁止することで実現できます。そうすれば、すべての依存が外部の取得元からではなく、ソース管理に追加されることを保証できます。

## 出力例 {#example-output}

![sources output](../../output/sources.svg)
{% endraw %}
