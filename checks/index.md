{% raw %}
# チェック {#checks}

cargo-deny は、プロジェクトのクレートグラフに対して行える、いくつかの種類のチェックに対応しています。既定では、`cargo deny check` は対応しているチェックを**すべて**実行します。あるチェックの設定が明示されていない場合は、そのチェックの既定の設定を使います。

## [licenses](./licenses/index.html) {#licenses}

各クレートのライセンス情報をチェックします。

## [bans](./bans/index.html) {#bans}

グラフの中の特定のクレートと、重複をチェックします。

## [advisories](./advisories/index.html) {#advisories}

助言データベースを調べて、セキュリティ上の脆弱性があるクレート、`Unmaintained`（保守されていない）とされたクレート、ソースのレジストリから yank されたクレートがないかをチェックします。

## [sources](./sources/index.html) {#sources}

各クレートの取得元の場所をチェックします。
{% endraw %}
