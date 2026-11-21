{% raw %}
# 設定 {#config}

cargo-deny の最上位の設定ファイルは、ふつう `deny.toml` という名前です。cargo-deny は、マニフェストがあるディレクトリから始めて親ディレクトリへとさかのぼりながら、各ディレクトリで次のパスを順に調べます。

- `deny.toml`
- `.deny.toml`
- `.cargo/deny.toml`
- `.config/deny.toml`

最初に見つかったファイルが使われます。特定のファイルを使いたいときは、`--config <path>` を渡してください。

## 例 - cargo-deny 自身の設定 {#example---cargo-denys-own-configuration}

```toml
[graph]
# cargo-deny は、実際には「通常の」tier-1 ターゲットで動かすことだけを想定しています
targets = [
    "x86_64-unknown-linux-gnu",
    "aarch64-unknown-linux-gnu",
    "x86_64-unknown-linux-musl",
    "aarch64-apple-darwin",
    "x86_64-apple-darwin",
    "x86_64-pc-windows-msvc",
]
all-features = true

[advisories]
unmaintained = "all"
unsound = "all"
ignore = []

[bans]
multiple-versions = "deny"
wildcards = 'deny'
deny = [
    { crate = "git2", use-instead = "git cli" },
    { crate = "gix", use-instead = "git cli" },
    { crate = "openssl", use-instead = "rustls" },
    { crate = "openssl-sys", use-instead = "rustls" },
    "libssh2-sys",
    { crate = "cmake", use-instead = "cc" },
    { crate = "windows", reason = "bloated and unnecessary", use-instead = "ideally inline bindings, practically, windows-sys" },
]
skip = [
    { crate = "getrandom@0.2.17", reason = "ring uses this old version" },
    { crate = "hashbrown@0.15.5", reason = "petgraph pulls this in directly, while pulling in later versions via indexmap" },
]
skip-tree = [
    { crate = "windows-sys", reason = "a foundational crate for many that bumps far too frequently to ever have a shared version" },
]

[bans.std-replacements]
scope = "workspace"

[sources]
unknown-registry = "deny"
unknown-git = "deny"

[licenses]
# テキストからライセンスを推定するときは、非常に高い確信度を求めます
confidence-threshold = 0.93
allow = [
    "Apache-2.0",
    "Apache-2.0 WITH LLVM-exception",
    "BSD-3-Clause",
    "MIT",
    "ISC",
    "Unicode-3.0",
    "Zlib",
]
exceptions = [
    { crate = "zstd-sys", allow = ["BSD-2-Clause"] }
]
```

## `graph` {#graph}

graph テーブルでは、各チェックの対象となる依存グラフをどのように組み立てるかを設定します。

```toml
[graph]
targets = [
    "x86_64-unknown-linux-gnu",
    { triple = "aarch64-apple-darwin" },
    { triple = "x86_64-pc-windows-msvc", features = ["sse2"] },
]
exclude = ["some-crate@0.1.0"]
all-features = true
no-default-features = false
features = ["some-feature"]
exclude-dev = true
```

### `targets` {#targets}

既定では、cargo-deny は cargo が解決したすべてのクレートを対象にします。これには、次のようなターゲット固有の依存も含まれます。

```toml
[target.x86_64-pc-windows-msvc.dependencies]
winapi = "0.3.8"

[target.'cfg(target_os = "fuchsia")'.dependencies]
fuchsia-cprng = "0.1.1"
```

しかし、実際に `x86_64-fuchsia` や `aarch64-fuchsia` 向けにビルドしているのでなければ、`fuchsia-cprng` がプロジェクトにコンパイルされたりリンクされたりすることはありません。そのため、これをチェックしても意味がありません。

`targets` フィールドでは、**実際に**ビルドする 1 つ以上のターゲットを指定できます。クレートへの依存リンクはそれぞれこのリストと照らし合わされ、リストのどのターゲットもターゲット制約を満たさない場合、その依存リンクは無視されます。あるクレートへの依存リンクが 1 つもなくなった場合、そのクレートはチェックの対象となるクレートグラフに含まれません。

#### `targets.triple` または `"<triple_string>"` {#targetstriple-or-triple_string}

ターゲット固有の依存を絞り込むのに使うターゲットの [ターゲットトリプル](https://forge.rust-lang.org/release/platform-support.html) です。指定したターゲットトリプルが `rustc` に組み込まれたターゲットの 1 つで**ない**場合、そのターゲットの設定チェックは、`[target.<target-triple>.dependencies]` という生の形式のターゲット設定だけに限られます。`cfg()` 式を評価するには、ターゲットの詳細を知っている必要があるためです。

#### `targets.features` {#targetsfeatures}

Rust の `cfg()` 式は [`target_feature = "feature-name"`](https://doc.rust-lang.org/reference/attributes/codegen.html#the-target_feature-attribute) という述語に対応していますが、今のところ、コンパイル時にこれを実際に渡す方法は `RUSTFLAGS` 環境変数を使うことだけです。`features` フィールドでは、特定のターゲットトリプルについて、ビルドで使う予定の `target_feature` を 1 つ以上指定できます。この文書を書いた時点では、cargo-deny は、指定したフィーチャがそのターゲットトリプルで実際に有効かどうかを検証しようとはしません。ただし、検証は[予定されています](https://github.com/EmbarkStudios/cfg-expr/issues/1)。

### `exclude` {#exclude}

コマンドラインオプションの [`--exclude`](../cli/common.html#--exclude) と同じように、このフィールドでは 1 つ以上の [Package ID 指定](https://doc.rust-lang.org/cargo/commands/cargo-pkgid.html) を指定できます。指定したクレートは、実行する操作で使うクレートグラフから除外されます。

クレートの除外は再帰的に働くことに注意してください。あるクレートの推移的な依存が、除外したクレートを通してしか参照されていない場合、その依存もクレートグラフから除外されます。

### `all-features` {#all-features}

`true` にすると、メタデータを集めるときに `--all-features` が使われます。

### `no-default-features` {#no-default-features}

`true` にすると、メタデータを集めるときに `--no-default-features` が使われます。

### `features` {#features}

これを設定していて、コマンドラインで `--features` を指定していない場合、メタデータを集めるときにこれらのフィーチャが使われます。

### `exclude-dev` {#exclude-dev}

`true` にすると、すべての `dev-dependencies` が、ワークスペースのクレートのものも含めて、どのチェックで使うクレートグラフにも含まれなくなります。このオプションは、コマンドラインで `check` サブコマンドの[前](../cli/common.html#--exclude-dev)または[後](../cli/check.html#--exclude-dev)に `--exclude-dev` を付けても有効にできます。

### `exclude-unpublished` {#exclude-unpublished}

`true` にすると、`publish = false` が指定されたワークスペースのクレートは、依存グラフのルートとして使われなくなります。つまり、そのクレート自体と、_公開されている_ワークスペースのクレートから直接にも間接にも参照されていないその依存は、チェックの対象となる依存グラフから除外されます。

## `output` {#output}

### `feature-depth` {#feature-depth}

診断に包含グラフを含めるとき、フィーチャを表示する最大の深さです。コマンドラインで `--feature-depth` を指定した場合はそちらが使われます。実際にフィーチャを表示する診断にだけ適用されます。指定しない場合の既定値は `1` です。

## パッケージ指定 {#package-specs}

多くの設定オプションでは、少なくともパッケージ指定が必要です。ここではそれについて説明します。パッケージ指定を使うオプションは、それぞれのドキュメントで明記します。以下の例では [`bans.deny`](bans/cfg.html#deny) オプションを使います。

### 文字列形式 {#string-format}

そのオプションが最低限パッケージ指定だけを必要とする場合は、文字列形式を使えます。文字列形式には 3 つの書き方があります。

#### 単純な形式 {#simple}

```toml
# simple クレートのどのバージョンにも一致します
deny = ["simple"]
```

いちばん単純な文字列は、クレート名だけのものです。この場合、チェックで使うバージョン要件は `*` になり、グラフにあるそのクレートのすべてのバージョンに一致します。

#### バージョン要件付き {#with-version-requirements}

```toml
# simple クレートのうち、述語に合うバージョンだけに一致します
deny = ["simple:<=0.1,>0.2"]
```

クレートにバージョン要件(述語)を適用したいときは、`:` 区切りの後ろにそれを付け加えるだけです。

#### 完全一致 {#exact}

```toml
# simple クレートのこのバージョンだけに一致します
deny = [
    "simple@0.1.0",
    # これは意味的には上と同じです
    "simple:=0.1.0",
]
```

完全一致の形式は、バージョン要件の特別な場合です。`@` の後ろの semver は [= (Exact)](https://docs.rs/semver/latest/semver/enum.Op.html#opexact) に変換されます。

### テーブル形式 {#table-format}

#### crate 形式 {#crate-format}

```toml
deny = [
    { crate = "simple@0.1.0" }, # "simple@0.1.0" と同じです
    { crate = "simple", wrappers = ["example"] },
]
```

crate 形式は、古い `name` と `version` のテーブル形式に代わるものです。上で説明した文字列形式を、1 つの `crate` キーに書きます。

#### 古い形式 {#old-format}

```toml
deny = [
    { name = "simple" },
    { name = "simple", version = "*" }
    { name = "simple", wrappers = ["example"] }
]
```

古い形式では、必須の `name` キーと、省略できる `version` キーを使います。この形式は非推奨なので、使わないでください。

## `[licenses]` セクション {#the-licenses-section}

詳しくは [licenses の設定](licenses/cfg.html) を見てください。

## `[bans]` セクション {#the-bans-section}

詳しくは [bans の設定](bans/cfg.html) を見てください。

## `[advisories]` セクション {#the-advisories-section}

詳しくは [advisories の設定](advisories/cfg.html) を見てください。

## `[sources]` セクション {#the-sources-section}

詳しくは [sources の設定](sources/cfg.html) を見てください。
{% endraw %}
