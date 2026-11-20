{% raw %}
# bans の診断 {#bans-diagnostics}

<!-- markdownlint-disable-next-line heading-increment -->

### `banned` {#banned}

[明示的に禁止された](cfg.html#deny)クレートが見つかりました。

### `allowed` {#allowed}

[明示的に許可された](cfg.html#allow)クレートが見つかりました。

### `not-allowed` {#not-allowed}

[`bans.allow`](cfg.html#allow) を使っているときに、そのリストにないクレートが見つかりました。

### `duplicate` {#duplicate}

同じクレートの複数の[バージョン](cfg.html#multiple-versions)が見つかりました。

### `skipped` {#skipped}

[`bans.skip`](cfg.html#skip) のエントリに一致するクレートのバージョンが見つかりました。

### `wildcard` {#wildcard}

あるクレートが、1 つ以上のクレートから[ワイルドカードの依存](cfg.html#wildcards)として取り込まれていました。

### `workspace-duplicate` {#workspace-duplicate}

ワークスペースの直接の依存が複数回参照されていて、そのすべての宣言で [`workspace = true`](cfg.html#workspace-duplicates) が使われていたわけではありませんでした。

### `unresolved-workspace-dependency` {#unresolved-workspace-dependency}

ワークスペースの依存（またはパッチ）が、正確にどのクレートに解決されたかを判断できませんでした。これは、cargo-deny の不具合である可能性が高いです。

### `unused-workspace-dependency` {#unused-workspace-dependency}

[`[workspace.dependencies]`](https://doc.rust-lang.org/cargo/reference/workspaces.html#the-dependencies-table) が宣言されていますが、ワークスペースのどこでも実際には使われていません。

### `unmatched-skip` {#unmatched-skip}

[`bans.skip`](cfg.html#skip) にあるクレートのバージョンが見つかりませんでした。

### `unmatched-skip-root` {#unmatched-skip-root}

[`bans.skip-tree`](cfg.html#skip-tree) にあるクレートのバージョンが見つかりませんでした。

### `unnecessary-skip` {#unnecessary-skip}

[`bans.skip`](cfg.html#skip) で指定されたクレートはグラフの中にありましたが、そのクレートにはバージョンが 1 つしかなかったため、`skip` のエントリは役に立っていません。

### `allowed-by-wrapper` {#allowed-by-wrapper}

`bans.deny` にあるクレートが、[`wrappers`](cfg.html#wrappers) のクレートから直接依存されていたため、許可されました。

### `unmatched-wrapper` {#unmatched-wrapper}

`bans.deny` にあるクレートには 1 つ以上の [`wrappers`](cfg.html#wrappers) のクレートがありましたが、そのリストにないクレートが、禁止されたクレートに直接依存していました。

### `unused-wrapper` {#unused-wrapper}

`bans.deny` にあるクレートが使っている[ラッパー](cfg.html#wrappers)が、どれにも一致しませんでした。

### `skipped-by-root` {#skipped-by-root}

[`bans.skip-tree`](cfg.html#skip-tree) にあるクレートのバージョンから推移的に参照されていたため、あるクレートが重複のチェックからスキップされました。

### `unmatched-root` {#unmatched-root}

[`bans.skip-tree`](cfg.html#skip-tree) にあるクレートのバージョンが見つかりませんでした。

### `build-script-not-allowed` {#build-script-not-allowed}

ビルドスクリプトを持っているのに、[`bans.allow-build-script`](cfg.html#allow-build-scripts) のリストに入っていないため、禁止されたクレートです。

### `exact-features-mismatch` {#exact-features-mismatch}

クレートのフィーチャが、設定されたフィーチャの集合と完全には一致せず、しかも [`bans.features.exact`](cfg.html#featuresexact) が `true` になっています。

### `feature-not-explicitly-allowed` {#feature-not-explicitly-allowed}

クレートのフィーチャで、[`bans.features.allow`](cfg.html#featuresallow) の集合にないフィーチャが使われていました。

### `feature-banned` {#feature-banned}

有効になっているクレートのフィーチャが、[`bans.features.deny`](cfg.html#features-deny) のリストにあります。

### `unknown-feature` {#unknown-feature}

[`bans.features.deny`](cfg.html#features-deny) または [`bans.features.allow`](cfg.html#features-allow) にあるフィーチャが、そのクレートに存在しません。

### `default-feature-enabled` {#default-feature-enabled}

あるクレートで `default` フィーチャが有効になっていて、しかも [`bans.external-default-features`](cfg.html#external-default-features) または [`bans.workspace-default-features`](cfg.html#workspace-default-features) が設定されていました。

### `path-bypassed` {#path-bypassed}

[`bans.build.bypass.allow.path`](cfg.html#path) で指定されたパスが、スキャンの対象外になりました。必要に応じて、その内容が SHA-256 チェックサムと一致することも確かめられています。

### `path-bypassed-by-glob` {#path-bypassed-by-glob}

あるパスが、1 つ以上の [glob パターン](cfg.html#allow-globs)に一致したため、スキャンの対象外になりました。

### `checksum-match` {#checksum-match}

ファイルの内容から計算した SHA-256 チェックサムが、設定にあるチェックサムと一致しました。

### `checksum-mismatch` {#checksum-mismatch}

ファイルの内容から計算した SHA-256 チェックサムが、設定にあるチェックサムと一致しませんでした。

### `denied-by-extension` {#denied-by-extension}

ファイルの拡張子が、[ユーザーが指定した](cfg.html#script-extensions)拡張子か、[組み込みの](cfg.html#enable-builtin-globs)拡張子のどちらかに一致しました。

### `detected-executable` {#detected-executable}

[ネイティブの実行ファイル](cfg.html#executables)が見つかりました。

### `detected-executable-script` {#detected-executable-script}

[インタプリタで実行されるスクリプト](cfg.html#interpreted)が見つかりました。

### `unable-to-check-path` {#unable-to-check-path}

ディスク上のファイルを開くか読み込むときに、I/O エラーが起きました。

### `features-enabled` {#features-enabled}

1 つ以上の [`required-features`](cfg.html#build-script-and-required-features) が有効になっていたため、[`build-script`](cfg.html#build-script-and-required-features) によるスキャンの省略が無視されました。

### `unmatched-bypass` {#unmatched-bypass}

[クレートの bypass](cfg.html#bypass) が、グラフの中のどのクレートにも一致しませんでした。

### `unmatched-path-bypass` {#unmatched-path-bypass}

[パスの bypass](cfg.html#bypassallow) が、クレートの中のどのファイルにも一致しませんでした。

### `unmatched-glob` {#unmatched-glob}

[glob の bypass](cfg.html#allow-globs) が、クレートの中のどのファイルにも一致しませんでした。

### `unmatched-allow-build-script` {#unmatched-allow-build-script}

[allow-build-scripts](cfg.html#allow-build-scripts) で、グラフの中に見つからないクレートが指定されていました。

### `allowed-crate-without-build-script` {#allowed-crate-without-build-script}

[allow-build-scripts](cfg.html#allow-build-scripts) で、ビルドスクリプトを持っていないクレートが指定されていました。

### `non-root-path` {#non-root-path}

パッケージのソースにあるパスが、パッケージのソースを起点としていませんでした。

### `non-utf8-path` {#non-utf8-path}

パッケージのソースの中で、utf8 でないパスが見つかりました。
{% endraw %}
