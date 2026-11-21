{% raw %}
# sources の診断 {#sources-diagnostics}

<!-- markdownlint-disable-next-line heading-increment -->

### `git-source-underspecified` {#git-source-underspecified}

`git` の取得元が、[`sources.required-git-spec`](cfg.html#required-git-spec) で求められる最低限の指定子を満たさない指定を使っています。

### `allowed-source` {#allowed-source}

クレートの取得元が、[`sources.allow-git`](cfg.html#allow-git) または [`sources.allow-registry`](cfg.html#allow-registry) で明示的に許可されています。

### `allowed-by-organization` {#allowed-by-organization}

クレートの取得元が、[`sources.allow-org`](cfg.html#allow-org) のエントリで明示的に許可されました。

### `source-not-allowed` {#source-not-allowed}

クレートの取得元が明示的に許可されていませんでした。

### `unmatched-source` {#unmatched-source}

[`sources.allow-git`](cfg.html#allow-git) または [`sources.allow-registry`](cfg.html#allow-registry) で許可された取得元が見つかりませんでした。

この診断は、[`sources.unused-allowed-source`](cfg.html#unused-allowed-source) フィールドを "allow" に設定すると出なくなります。

### `unmatched-organization` {#unmatched-organization}

[`sources.allow-org`](cfg.html#allow-org) で許可された取得元が見つかりませんでした。
{% endraw %}
