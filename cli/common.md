{% raw %}
# 共通オプション {#common-options}

サブコマンドには、サブコマンドの前に指定できる共通のオプションがいくつかあります。

## オプション {#options}

### `--manifest-path` {#--manifest-path}

操作の基準として使う `Cargo.toml` ファイルのパスです。

### `-c, --config <CONFIG>` {#-c---config-config}

使う設定ファイルのパスです。

指定しない場合の既定値は `<cwd>/deny.toml` です。

### `--metadata-path` {#--metadata-path}

cargo のメタデータの json のパスです。

既定では、`cargo metadata` を使ってメタデータの json を生成します。`cargo metadata` の出力のパスを指定すると、この動きを上書きできます。

### `--all-features`(単一のクレートまたはワークスペース) {#--all-features-single-crate-or-workspace}

対象とするクレートを決めるときに、すべてのフィーチャを有効にします。単一のクレートとワークスペースの両方で使えます。

### `--no-default-features`(単一のクレートのみ) {#--no-default-features-single-crate-only}

対象とするクレートを決めるときに、クレートの `default` フィーチャを無効にします。

### `--features`(単一のクレートのみ) {#--features-single-crate-only}

対象とするクレートを決めるときに有効にするフィーチャの、空白区切りのリストです。

### `--workspace` {#--workspace}

ほかの方法で除外されていないかぎり、ワークスペースのすべてのクレートを、操作対象のクレートグラフのルートとして使います。既定では、[仮想マニフェスト](https://doc.rust-lang.org/cargo/reference/manifest.html#virtual-manifest)を指定した場合、ワークスペースのすべてのクレートがルートとして使われます。しかし、ワークスペースの中にある通常のパッケージのマニフェストを指定した場合は、そのクレートだけがグラフのルートとして使われ、グラフに含まれるのは、そのクレートが依存しているワークスペースのほかのクレートだけになります。ワークスペースの中のサブクレートを指定しつつ、ワークスペースのほかのクレートもすべて含めたい場合は、このフラグを使えます。

### `--exclude-dev` {#--exclude-dev}

`true` にすると、すべての `dev-dependencies` が、ワークスペースのクレートのものも含めて、どのチェックで使うクレートグラフにも含まれなくなります。

### `--exclude` {#--exclude}

指定したパッケージをクレートグラフから除外します。ほかの cargo サブコマンドとは違い、`--workspace` フラグと一緒に使う必要はありません。このフラグは何度でも指定できます。

ほかの cargo サブコマンドと似た(ただし少し厳しい)[Package ID 指定](https://doc.rust-lang.org/cargo/commands/cargo-pkgid.html)を使います。

パッケージは[設定](../checks/cfg.html#the-exclude-field-optional)ファイルでも除外できます。コマンドラインでこれを指定すると、設定にリストがあれば、そのリストにパッケージ ID が追加されます。

### `-L, --log-level` {#-l---log-level}

メッセージのログレベルです。このレベル以上のログメッセージだけが出力されます。

指定できる値:

- `off` - 何も出力しません
- `error`
- `warn`(既定)
- `info`
- `debug`
- `trace`

### `--format` {#--format}

ログと診断のメッセージの両方の出力形式です。

指定できる値:

- `human`(既定) - やっかいな人間向けの出力です
- `json` - ログメッセージや診断を、それぞれ 1 行の JSON オブジェクトとして出力します
- `sarif` - [SARIF](https://sarifweb.azurewebsites.net/) 形式で出力します

### `--color` {#--color}

人間向けの形式の出力に色を付けるかどうかです。JSON の出力に使っても効果はありません。

指定できる値:

- `auto`(既定) - 出力先が TTY なら色を付けます
- `always` - 常に色を付けます
- `never` - どの出力にも色を付けません

### `-t, --target` {#-t---target}

クレートを絞り込むための 1 つ以上のプラットフォームです。ターゲット固有の依存は、指定したターゲットの少なくとも 1 つに一致しなければ無視されます。これは、最上位の [`targets = []`](../checks/cfg.html) の設定値を上書きします。

### `--exclude-unpublished` {#--exclude-unpublished}

指定すると、公開されていないワークスペースのメンバーをグラフのルートから除外します。

ワークスペースのメンバーは、`publish = false` が明示的に指定されていると、公開されていないものと見なされます。なお、除外されたワークスペースのメンバーも、cargo による最初の依存解決にはまだ使われます。そのため、実際に使われる依存のバージョンに影響することがあります。

### [`--locked`](https://doc.rust-lang.org/cargo/commands/cargo-fetch.html#option-cargo-fetch---locked) {#--locked}

既存の Cargo.lock ファイルを最初に生成したときとまったく同じ依存とバージョンが使われることを保証します。次のどちらかの状況になると、Cargo はエラーで終了します。

- ロックファイルがない。
- 依存の解決結果が違うため、Cargo がロックファイルを変更しようとした。

### [`--offline`](https://doc.rust-lang.org/cargo/commands/cargo-fetch.html#option-cargo-fetch---offline) {#--offline}

Cargo と `cargo-deny` が、どんな理由でもネットワークにアクセスしないようにします。このフラグがない場合、ネットワークにアクセスする必要があるのにネットワークが使えないと、Cargo はエラーで止まります。このフラグがある場合、Cargo はできるかぎりネットワークなしで処理を進めようとします。

これにより、オンラインのときとは違う依存解決になる場合があるので注意してください。Cargo は、インデックスのローカルのコピーでもっと新しいバージョンがあると示されていても、ローカルにダウンロード済みのクレートだけを使います。オフラインにする前に依存をダウンロードするには、cargo-fetch(1) コマンドを見てください。

このオプションを付けると、`cargo-deny` は助言のデータベースも取得しません。つまり、前回データベースを取得した後に追加・更新された助言はわからないので、依存グラフに対してチェックされません。また、`std-replacement-data` も取得しません。そのため、そのデータベースがまだファイルシステムにない場合は、クレートが std/core で部分的または完全に置き換えられているかどうかのチェックが行われません。

### [`--frozen`](https://doc.rust-lang.org/cargo/commands/cargo-fetch.html#option-cargo-fetch---frozen) {#--frozen}

`--locked` と `--offline` の両方を指定するのと同じです。
{% endraw %}
