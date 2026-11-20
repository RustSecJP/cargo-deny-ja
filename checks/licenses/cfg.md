{% raw %}
# `[licenses]` セクション {#the-licenses-section}

`cargo deny check license` のすべての設定が入ります。

## 例 {#example}

```toml
[licenses]
unused-allowed-license = "warn"
confidence-threshold = 0.95
allow = [
    "EUPL-1.2",
    "Apache-2.0 WITH LLVM-exception",
]

[licenses.private]
ignore = true
registries = ["sekrets"]

[[licenses.exceptions]]
allow = ["Zlib"]
name = "adler32"
version = "0.1.1"

[[licenses.clarify]]
name = "ring"
expression = "MIT AND ISC AND OpenSSL"
license-files = [
    { path = "LICENSE", hash = 0xbd0eed23 }
]
```

## SPDX 識別子 {#spdx-identifiers}

ライセンスの設定セクションで使う識別子は、すべて有効な SPDX v2.1 の短い識別子である必要があります。[SPDX License List](https://spdx.org/licenses/) のバージョン 3.11 にあるものを使うか、`LicenseRef-` を前に付けて独自の識別子を使ってください。

```toml
allow = [
    # Apache ライセンスの識別子
    "Apache-2.0",
    # 独自のライセンス識別子
    "LicenseRef-Embark-Custom",
]

# SPDX のリストにないライセンスを使うクレートには、
# 独自のライセンス参照を指定できます
[[licenses.clarify]]
crate = "a-crate"
expression = "LicenseRef-Embark-Custom"
license-files = [
    { path = "LICENSE", hash = 0x001c7e6c },
]
```

ライセンス識別子には、`WITH <exception-id>` をライセンス識別子の後ろに付けることで、[例外](https://spdx.org/licenses/exceptions-index.html)を組み合わせることもできます。例外と組み合わせたライセンスは、例外のない同じライセンスとは別のものとして扱われます。

```toml
allow = [
    # Apache ライセンスの識別子
    "Apache-2.0",
    # Apache ライセンス + LLVM-exception
    "Apache-2.0 WITH LLVM-exception",
]
```

### `include-dev` {#include-dev}

`true` にすると、`dev-dependencies` についてもライセンスがチェックされます。既定では false です。`dev-dependencies` は下流のクレートでは使われず、バイナリの成果物にも含まれないためです。

### `include-build` {#include-build}

`true` にすると、`build-dependencies` についてライセンスがチェックされます。既定では true です。build-dependencies はビルドの成果物に影響することがあり、ライセンスに関係することが多いためです。ライセンスのチェックから `build-dependencies` を外したい場合は、これを `false` にしてください。

### `version` {#version}

```toml
version = 2
```

version フィールドは(この文書を書いた時点では)もう使われていません。次のフィールドは削除されており、今はエラーになります。

- `unlicensed` - 削除されました。クレートにライセンスがない場合は、修正のために issue や PR を出してください。それまでの間は、[明確化](#the-clarify-field-optional)を追加できます。
- `deny` - 削除されました。明示的に許可しないかぎり、すべてのライセンスは拒否されます
- `copyleft` - 削除されました。明示的に許可しないかぎり、すべてのライセンスは拒否されます
- `allow-osi-fsf-free` - 削除されました。明示的に許可しないかぎり、すべてのライセンスは拒否されます
- `default` - 削除されました。明示的に許可しないかぎり、すべてのライセンスは拒否されます

### `allow` {#allow}

明示的に許可するライセンスです。このリストにないライセンスは既定で拒否されます。そのため、ここに挙げたライセンスのどれもプロジェクトが使うライセンス式を満たせない場合、ライセンスのチェックは失敗します。

#### GNU ライセンスについての注意 {#note-on-gnu-licenses}

- GPL
- AGPL
- LGPL
- GFDL

GNU ライセンスは、言うまでもなく、SPDX のリストにあるほかのどのライセンスとも違っていて、扱いが面倒です。cargo-deny のバージョン 0.18.3 以前では、GNU ライセンスもほかのライセンスと同じように扱い、ほかのライセンスと同じ規則に従わせようとしていました。

0.18.4 からは、GNU ライセンスは厳密に扱われます。たとえば、`GPL-3.0`(非推奨)、`GPL-3.0-or-later`、`GPL-2.0-or-later` のライセンスは許可したいが、`GPL-3.0-only` は許可したくない場合は、次のようにする必要があります。

```toml
allow = ['GPL-2.0-or-later', 'GPL-3.0', 'GPL-3.0-or-later']
```

これは、プロジェクトが自分のライセンスを非推奨の形式である `GPL-2.0` と宣言している場合、`GPL-2.0-only` は使えず、`GPL-2.0` にしか一致しないということでもあります。

### `exceptions` {#exceptions}

ライセンスの設定は、ふつうクレートグラフ全体に適用されます。しかしこれは、あるライセンスを 1 つ許可すると、実際にそのライセンスを使うクレートが 1 つしかなくても、あらゆるクレートに対して許可されることを意味します。`exceptions` フィールドは、特定のクレートに対してだけ追加のライセンスを許可するためのものです。どこで使われてもかまわないライセンスと、もっと慎重に選びたく、将来暗黙のうちに許可されてほしくないライセンスとを、はっきり区別できます。

このフィールドでは、例外を適用するクレートを選ぶのに [PackageSpecs](../cfg.html#package-specs) を使います。

### 追加の例外設定ファイル {#additional-exceptions-configuration-file}

cargo-deny の設定は全体で共通にして、例外だけをプロジェクトごとに持ちたい場合があります。これは、最上位の `Cargo.toml` マニフェストファイルから見て次のいずれかの場所に、プロジェクトの例外ファイルを置くことで実現できます。

`cargo-deny` は次のファイルを探します。`<cwd>/deny.exceptions.toml`、`<cwd>/.deny.exceptions.toml`、`<cwd>/.cargo/deny.exceptions.toml`

exceptions フィールドだけを設定してください。

```toml
exceptions = [
    # 各エントリは、クレートとバージョンの制約、そしてそれ専用の許可リストです。
    { allow = ["CDDL-1.0"], crate = "inferno" },
]
```

#### `allow` {#allow-1}

全体の `allow` フィールドとまったく同じです。

```toml
[licenses]
allow = [
    "Apache-2.0",
    "MIT",
]
exceptions = [
    # Apache-2.0 でも MIT でもライセンスできないクレートはこれだけなので、
    # これにだけ例外を追加します。こうすると、このライセンスを必要とする
    # クレートをほかにも追加したときに警告が出ます
    { crate = "cloudabi", allow = ["BSD-2-Clause"] },
]
```

### `confidence-threshold` {#confidence-threshold}

`cargo-deny` は、LICENSE ファイルのライセンスを判定するのに [spdx](https://github.com/EmbarkStudios/spdx) を使います。ライセンスの文章は、作者や著作権の年などによってばらつきがあるので、askalano は判定に `0.0`(確信なし)から `1.0`(完全一致)までの確信度のスコアを付けます。確信度のしきい値は、スコアがしきい値に届かない場合にライセンスの判定を退けるために使われます。

`0.0` - `1.0`(既定値 `0.8`)

### `clarify` {#clarify}

まれなケースとして、クレートに機械で読みやすいライセンス情報がなく、既定では cargo-deny に「unlicensed」と見なされてしまうことがあります。そのクレートを使うための(できれば)一時的な対処として、クレートのソースにある 1 つ以上のファイルにもとづいて SPDX 式を手動で割り当て、クレートの明確化を指定できます。cargo-deny は、クレートのソースファイルが明確化のハッシュと完全に一致しているかぎり、その式を使います。

このフィールドでは、明確化を適用するクレートを選ぶのに [PackageSpecs](../cfg.html#package-specs) を使います。

```toml
[[licenses.clarify]]
crate = "webpki"
expression = "ISC"
license-files = [
    { path = "LICENSE", hash = 0x001c7e6c },
]
```

#### `expression` {#expression}

クレートのライセンス要件として指定する [SPDX ライセンス式][SPDX-expr] です。

#### `license-files` {#license-files}

ライセンス式がそのクレートのバージョンにまだ当てはまることを確かめるためにチェックする、1 つ以上のファイルが入ります。

##### `path` {#path}

正しい情報の基準として使うファイルの、クレートから見た相対パスです。

##### `hash` {#hash}

ファイルの内容から計算した、中身を意識しなくてよいハッシュです。このハッシュは、cargo-deny が対象のファイルのライセンスを判定できなかったときに、ライセンスのチェックの出力から得られます。

### `private` {#private}

自分の非公開のワークスペースのクレートについてライセンスをチェックしても、役に立たなかったり、望まれなかったりすることがよくあります。private フィールドを使うと、そのチェックを外せます。

#### `ignore` {#ignore}

`true` にすると、ワークスペースのメンバーが公開されていない場合、そのライセンス式はチェックされません。

```toml
# Cargo.toml
[package]
name = "sekret"
license = "¯\_(ツ)_/¯"
publish = false # "private"!
```

```toml
# deny.toml
[licenses]
# これで sekret パッケージは無視されます
private = { ignore = true }
```

### `registries` {#registries}

ワークスペースのクレートを公開する可能性のある、非公開のレジストリのリストです。ワークスペースのメンバーが非公開のレジストリ**だけ**に公開される場合、`private.ignore = true` であれば、そのメンバーも無視されます。

```toml
# Cargo.toml
[package]
name = "sekret"
license = "¯\_(ツ)_/¯"
publish = ["sauce"]
```

```toml
# deny.toml
[licenses]
# これでも無視されます!
private = { ignore = true, registries = ["sauce"] }
```

### `ignore-sources` {#ignore-sources}

クレートの取得元となるレジストリのうち、ライセンスをチェックしないもののリストです。

```toml
# deny.toml
[licenses.private]
ignore = true
ignore-sources = ["https://sekretz.com/super/secret-index"]
```

[SPDX-expr]: https://spdx.github.io/spdx-spec/v2.3/SPDX-license-expressions/

### `unused-allowed-license` {#unused-allowed-license}

[`allow`](#allow) リストにあるライセンスのどれかが依存グラフの中で見つからなかったときに、どうするかを決めます。

- `warn`(既定) - `license.allow` にあるが、どのクレートでも使われていないライセンスごとに警告を出します。
- `allow` - `licenses.allow` リストにある使われていないライセンスは無視されます。
- `deny` - `licenses.allow` リストに使われていないライセンスがあるとエラーになり、ライセンスのチェックは失敗します。

### `unused-license-exception` {#unused-license-exception}

[`unused-allowed-license`](#unused-allowed-license) と同じですが、[`exceptions`](#exceptions) が対象です。
{% endraw %}
