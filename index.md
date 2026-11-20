{% raw %}
# [cargo-deny](https://github.com/EmbarkStudios/cargo-deny) {#cargo-deny}

cargo-deny は cargo のプラグインです。プロジェクトの依存グラフを検査（lint）して、すべての依存が期待や要件に合っているかを確かめられます。

## クイックスタート {#quickstart}

cargo-deny をインストールし、既定の設定でプロジェクトを初期化してから、プロジェクトに対してすべてのチェックを実行します。

```bash
cargo install --locked cargo-deny && cargo deny init && cargo deny check
```

## コマンドラインインターフェース {#command-line-interface}

cargo-deny は[コマンドラインツール](cli/index.html)として使うことを想定しています。使えるコマンドとオプションは、リンク先を参照してください。

## チェック {#checks}

cargo-deny は、いくつかの種類のチェックに対応しています。使えるチェックとその設定オプションは、[チェック](checks/index.html)を参照してください。

## API {#api}

cargo-deny は主に cargo のプラグインとして使うためのものですが、機能の大部分はライブラリにまとまっています。そのドキュメントは [docs.rs](https://docs.rs/cargo-deny) で見られます。

## GitHub Action {#github-action}

GitHub のプロジェクトでは、GitHub Action を使って、継続的インテグレーションの一部として cargo-deny を自動で実行できます。

```yaml
name: CI
on: [push, pull_request]
jobs:
  cargo-deny:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: EmbarkStudios/cargo-deny-action@v1
```

くわしくは、[`cargo-deny-action`](https://github.com/EmbarkStudios/cargo-deny-action) のリポジトリを参照してください。
{% endraw %}
