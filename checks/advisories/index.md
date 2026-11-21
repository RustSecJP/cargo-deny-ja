{% raw %}
# advisories {#advisories}

advisories チェックは、助言データベースを調べて、クレートの問題を見つけるためのものです。

```bash
cargo deny check advisories
```

## 使用例 - セキュリティ上の脆弱性を見つける {#use-case---detecting-security-vulnerabilities}

セキュリティ上の脆弱性は、たいていの人にとって「あまりうれしくない」ものです。幸い、Rust にはすぐれた[助言データベース](https://github.com/RustSec/advisory-db)があります。cargo-deny はこれを使って、（既知の）セキュリティ上の脆弱性があるクレートを使っていないかをチェックできます。

同じ形式に従っていれば、上の既定のデータベースの代わりに、またはそれに加えて、独自の助言データベースを使うこともできます。

## 使用例 - 保守されていないクレートを見つける {#use-case---detecting-unmaintained-crates}

[助言データベース](https://github.com/RustSec/advisory-db)には、保守されていないクレートについての助言も入っています。たいていの場合、利用者はこうしたクレートを避けて、より活発に保守されているクレートを使いたいはずです。既定では、`unmaintained` の助言はすべてエラーになります。ただし、次の設定を使うと、ワークスペースから保守されていないクレートに直接依存している場合にだけエラーにできます。

```toml
[advisories]
unmaintained = 'workspace'
```

## 出力例 {#example-output}

![advisories の出力](../../output/advisories.svg)
{% endraw %}
