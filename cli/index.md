{% raw %}
# コマンドラインツール {#command-line-tool}

cargo-deny は、コマンドラインツールとしても、[Rust のクレート](https://crates.io/crates/cargo-deny)としても使えます。まずは、コマンドラインツールの機能を見ていきましょう。

## バイナリからインストールする {#install-from-binaries}

主なプラットフォーム向けに、ビルド済みのバイナリをできるかぎり提供しています。[リリースのページ](https://github.com/EmbarkStudios/cargo-deny/releases)から、使っているプラットフォームに合ったバージョンをダウンロードしてください。

## Arch Linux でのインストール {#installation-on-arch-linux}

cargo-deny は Arch Linux の [extra リポジトリ](https://archlinux.org/packages/extra/x86_64/cargo-deny/)にあります。次のように [pacman](https://wiki.archlinux.org/title/Pacman) でインストールできます。

```bash
pacman -S cargo-deny
```

## ソースからインストールする {#install-from-source}

cargo-deny はソースからもインストールできます。

### 必要なもの {#pre-requisites}

cargo-deny は **[Rust](https://www.rust-lang.org/)** で書かれているので、**Cargo** でコンパイルする必要があります。まだ Rust をインストールしていない場合は、今すぐ [インストール](https://www.rust-lang.org/tools/install)してください。

cargo-deny は C のコードを使うクレートに依存しているので、gcc、clang、msvc などの C ツールチェーンもマシンに用意する必要があります。

### Crates.io 版のインストール {#install-cratesio-version}

Rust と Cargo がすでにインストールされていれば、cargo-deny のインストールは比較的簡単です。端末で次のコマンドを入力するだけです。

```bash
cargo install --locked cargo-deny
```

これで、[Crates.io](https://crates.io/) から最新リリースのソースコードを取得してコンパイルします。まだ追加していない場合は、Cargo の `bin` ディレクトリを `PATH` に追加する必要があります。

端末で `cargo deny help` を実行して、動くかどうか確かめてください。おめでとうございます。これで cargo-deny がインストールできました!

### Git 版のインストール {#install-git-version}

**[git 版](https://github.com/EmbarkStudios/cargo-deny)** には、**Crates.io** で次のバージョンとしてリリースされる予定の、最新の不具合修正と機能がすべて入っています。次のリリースまで待てない場合は、git 版を自分でビルドできます。

```bash
cargo install --locked --git https://github.com/EmbarkStudios/cargo-deny cargo-deny
```

端末で `cargo deny help` を実行して、動くかどうか確かめてください。おめでとうございます。これで cargo-deny がインストールできました!

## CI での使い方 {#ci-usage}

Github のリポジトリで cargo-deny を実行するための Github Action を用意しています。[こちら](https://github.com/EmbarkStudios/cargo-deny-action)を見てください。

Action を使いたくない場合は、上で説明したように cargo-deny を手動でダウンロード(またはインストール)できます。手始めに使えるように、コピーして使えるスクリプトの例を次に示します。

```bash
#!/bin/bash
set -eu

NAME="cargo-deny"
VS="0.8.5"
DIR="/tmp/$NAME"

mkdir $DIR

# tarball をダウンロードします
curl -L -o $DIR/archive.tar.gz https://github.com/EmbarkStudios/$NAME/releases/download/$VS/$NAME-$VS-x86_64-unknown-linux-musl.tar.gz

# tarball を一時ディレクトリに展開します
tar -xzvf $DIR/archive.tar.gz --strip-components=1 -C $DIR

# 現在のディレクトリで cargo deny check を実行します
$DIR/$NAME -L debug check bans licenses advisories
```
{% endraw %}
