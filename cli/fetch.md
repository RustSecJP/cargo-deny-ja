{% raw %}
# `fetch` コマンド {#the-fetch-command}

`check` サブコマンドや `list` サブコマンドで使う可能性のある、リモートのデータを取得します。

## 引数 {#args}

### `SOURCES` {#sources}

指定できる値:

- db - 設定された助言のデータベースを取得します。既定は <https://github.com/RustSec/advisory-db> です。
- index - グラフにあるクレートについて、クレートのソースとインデックスの情報を取得します。
- std-replacement - <https://github.com/EmbarkStudios/std-replacement-data/tree/collated> から std-replacement-data を取得します
- all(既定) - 上のすべてを取得します。
{% endraw %}
