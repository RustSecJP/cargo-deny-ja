{% raw %}
# advisories の診断 {#advisories-diagnostics}

<!-- markdownlint-disable-next-line heading-increment -->

### `vulnerability` {#vulnerability}

あるクレートについて、`vulnerability` の助言が見つかりました。

### `notice` {#notice}

あるクレートについて、`notice` の助言が見つかりました。

### `unmaintained` {#unmaintained}

あるクレートについて、[`unmaintained`](cfg.html#unmaintained) の助言が見つかりました。

### `unsound` {#unsound}

あるクレートについて、`unsound` の助言が見つかりました。

### `yanked` {#yanked}

レジストリのインデックスから [yank された](cfg.html#yanked)バージョンを使っているクレートが見つかりました。

### `index-failure` {#index-failure}

レジストリのインデックス（ふつうは crates.io）を読み込むか更新しようとしたときにエラーが起きたため、cargo-deny はどのクレートについても、現在 yank されているかどうかをチェックできませんでした。

### `index-cache-load-failure` {#index-cache-load-failure}

あるクレートについて、キャッシュされたインデックスの詳細を読み込めませんでした。

### `advisory-not-detected` {#advisory-not-detected}

[`advisories.ignore`](cfg.html#ignore) にある助言が、どのクレートにも当てはまりませんでした。これは、その助言が[取り下げられた](https://docs.rs/rustsec/latest/rustsec/advisory/struct.Metadata.html#structfield.withdrawn)場合や、クレートのバージョンが、助言の対象となる影響を受けるバージョンの範囲から外れた場合に起こります。

この診断は、[`advisories.unused-ignored-advisory`](cfg.html#unused-ignored-advisory) フィールドを `'allow'` に設定すると出なくなります。

### `advisory-ignored` {#advisory-ignored}

[`advisories.ignore`](cfg.html#ignore) にある助言が見つかりました。

### `advisory-ignore-expired` {#advisory-ignore-expired}

[`advisories.ignore`](cfg.html#ignore) にある助言が、その ignore の [`expiry`](cfg.html#expiry) または既定の [`ignore-expiry`](cfg.html#ignore-expiry) に従って、期限切れになりました。

### `advisory-ignore-disallowed-dependent` {#advisory-ignore-disallowed-dependent}

助言の対象となるクレートに直接依存しているクレートが、[`ignore.allow`](cfg.html#allow) のリストにありませんでした。

### `advisory-ignore-allowed-dependent-missing` {#advisory-ignore-allowed-dependent-missing}

[`ignore.allow`](cfg.html#allow) のリストにある [PackageSpec](../cfg.html#package-specs) が、助言の対象となるどのクレートにも直接依存していませんでした。

### `advisory-withdrawn` {#advisory-withdrawn}

[`advisories.ignore`](cfg.html#ignore) にある助言が取り下げられました。この助言は、無視していなくても今後は診断を出しません。そのため、[`advisories.ignore`](cfg.html#ignore) から安全に取り除けます。

### `unknown-advisory` {#unknown-advisory}

[`advisories.ignore`](cfg.html#ignore) にある助言が、設定されたどの助言データベースにも見つかりませんでした。ふつうは、書き間違いがあることを示しています。今のところ、助言がデータベースから削除されることはないからです。少なくとも、正式な [advisory-db](https://github.com/rustsec/advisory-db) ではそうです。

### `yanked-ignored` {#yanked-ignored}

yank されたクレートのバージョンが、[`advisories.ignore`](cfg.html#ignore) によって無視されました。

### `yanked-not-detected` {#yanked-not-detected}

yank されたクレートのバージョンが [`advisories.ignore`](cfg.html#ignore) で無視されるように指定されていますが、クレートグラフの中に見つかりませんでした。

この診断は、[`advisories.unused-ignored-advisory`](cfg.html#unused-ignored-advisory) フィールドを `'allow'` に設定すると出なくなります。
{% endraw %}
