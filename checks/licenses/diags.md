{% raw %}
# licenses の診断 {#licenses-diagnostics}

<!-- markdownlint-disable-next-line heading-increment -->

### `rejected` {#rejected}

クレートの 1 つ以上のライセンスが、[許可](cfg.html#allow)するように設定されていなかったため、拒否されました。

### `accepted` {#accepted}

クレートのライセンス式は[許可](cfg.html#allow)されました。ただし、警告が出ている場合があります。

### `unlicensed` {#unlicensed}

クレートのライセンス式が見つからず、合成もできなかったため、ライセンスなしと見なされました。

### `skipped-private-workspace-crate` {#skipped-private-workspace-crate}

ワークスペースのメンバーが `publish = false` で、[スキップ](cfg.html#private)されました。

### `license-not-encountered` {#license-not-encountered}

[`licenses.allow`](cfg.html#allow) にあるライセンスが、どのクレートでも見つかりませんでした。

この診断は、[`licenses.unused-allowed-license`](cfg.html#unused-allowed-license) フィールドを `'allow'` に設定すると出なくなります。

### `license-exception-not-encountered` {#license-exception-not-encountered}

[`licenses.exception`](cfg.html#exceptions) が使われませんでした。適用先のクレートが見つからなかったためです。

### `empty-license-field` {#empty-license-field}

パッケージの `license` フィールドはありましたが、空でした。これは、以前は crates.io で許されていた不具合で、ほかのレジストリの実装でも許されている可能性があります。cargo-deny は、代わりにパッケージのソースからライセンスファイルを探そうとします。

### `gather-failure` {#gather-failure}

パッケージのソースから 1 つ以上のライセンスファイルを探そうとしたときに、エラーが起きました。

### `no-license-field` {#no-license-field}

パッケージが `license` フィールドを使っていませんでした。cargo-deny は、代わりにパッケージのソースからライセンスファイルを探そうとします。

### `missing-clarification-file` {#missing-clarification-file}

[明確化](cfg.html#clarify)で参照されているファイルが見つかりませんでした。

### `parse-error` {#parse-error}

クレートの SPDX 式を解析できませんでした。
{% endraw %}
