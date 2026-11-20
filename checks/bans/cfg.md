{% raw %}
# `[bans]` セクション {#the-bans-section}

`cargo deny check bans` の設定がすべて入ります。

## 設定例 {#example-config}

```toml
[bans]
multiple-versions = "deny"
wildcards = "deny"
allow-wildcard-paths = true
highlight = "simplest-path"
workspace-default-features = "warn"
external-default-features = "deny"
allow = [
    { name = "all-versionsa" },
    "version-rangea:<0.1.1",
    "specific-versionb@0.1.2",
    "any-version",
]
deny = [
    "specific-versiond@0.1.9",
    { name = "all-versionsd", wrappers = [
        "specific-versiona",
    ], reason = "we want to get rid of this crate but there is still one user of it" },
]
skip-tree = [{ name = "blah", depth = 20 }]

[bans.workspace-dependencies]
duplicates = "allow"
include-path-dependencies = false
unused = "allow"

[[bans.skip]]
name = "rand"
version = "=0.6.5"

[[bans.features]]
name = "featured-krate"
version = "1.0"
deny = ["bad-feature"]
allow = ["good-feature"]
exact = true
reason = "`bad-feature` is bad"

[bans.build]
allow-build-scripts = [{ name = "all-versionsa" }]
executables = "warn"
interpreted = "deny"
script-extensions = ["cs"]
enable-builtin-globs = true
include-dependencies = true
include-workspace = true
include-archives = true

[[bans.build.bypass]]
name = "allversionsa"
build-script = "5392f0e58ad06e089462d93304dfe82337acbbefb87a0749a7dc2ed32af04af7"
required-features = ["feature-used-at-build-time"]
allow-globs = ["scripts/*.cs"]
allow = [
    { path = "bin/x86_64-linux", checksum = "5392f0e58ad06e089462d93304dfe82337acbbefb87a0749a7dc2ed32af04af7" },
]

[bans.std-replacements]
scope = "workspace"
level = "deny"
rust-version = "1.0"
```

### `multiple-versions` {#multiple-versions}

同じクレートの複数のバージョンが見つかったときに、どうするかを決めます。

- `deny` - 重複があるクレートごとにエラーを出し、チェックを失敗させます。
- `warn`（既定値） - 重複があるクレートごとに警告を表示しますが、チェックは失敗させません。
- `allow` - 同じクレートの重複したバージョンを無視します。

### `multiple-versions-include-dev` {#multiple-versions-include-dev}

`true` にすると、クレートの複数のバージョンをチェックするときに、`dev-dependencies` も含めます。既定では false で、dev の依存の辺を通じてしか到達しないクレートは、複数のバージョンのチェックで無視されます。この場合、`skip` と `skip` tree も使われないことに注意してください。そのため、使われていない設定についての警告が出ることがあります。

### `wildcards` {#wildcards}

依存が `*`（ワイルドカード）のバージョンで指定されているときに、どうするかを決めます。

- `deny` - ワイルドカードのバージョンで指定されたクレートごとに、エラーを出します。
- `warn`（既定値） - ワイルドカードのバージョンを持つクレートごとに警告を表示しますが、チェックは失敗させません。
- `allow` - ワイルドカードのバージョン指定をすべて無視します。

### `allow-wildcard-paths` {#allow-wildcard-paths}

指定すると、`wildcard` フィールドの動作が次のように変わります。

- **非公開**のクレートにある [path](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html#specifying-path-dependencies) や [git](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html#specifying-dependencies-from-git-repositories) の `dependencies` について、警告やエラーを出さなくなります。
- 公開・非公開どちらのクレートでも、path や git の `dev-dependencies` について、警告やエラーを出さなくなります。
- **公開**されているクレートにある path や git の `dependencies` と `build-dependencies` については、これまでどおり警告やエラーを出します。

非公開のクレートに限られているのは、crates.io では、`dev-dependencies` を除いて、`path` や `git` の依存を持つパッケージを公開できないからです。

### `workspace-dependencies` {#workspace-dependencies}

[`[workspace.dependencies]`](https://doc.rust-lang.org/cargo/reference/workspaces.html#the-dependencies-table) をどう扱うかを設定します。

```toml
[bans.workspace-dependencies]
duplicates = 'deny'
include-path-dependencies = true
unused = 'deny'
```

#### `duplicates` {#duplicates}

ワークスペースの 2 つ以上の直接の依存が同じクレートに解決され、しかも 1 つ以上の宣言で `workspace = true` が使われていないときに、どうするかを決めます。

- `deny`（既定値） - `workspace = true` を使っていない依存の宣言ごとに、エラーを出します
- `warn` - `workspace = true` を使っていない依存の宣言ごとに警告を出しますが、チェックは失敗させません。
- `allow` - ワークスペースのクレートの依存について、`workspace = true` のチェックを行いません

#### `include-path-dependencies` {#include-path-dependencies}

`true` にすると、path の依存も重複のチェックに含めます。そうでない場合、path の依存は完全に無視されます。

#### `unused` {#unused}

[`[workspace.dependencies]`](https://doc.rust-lang.org/cargo/reference/workspaces.html#the-dependencies-table) にある依存が、ワークスペースで使われていないときに、どうするかを決めます。

- `deny`（既定値） - ワークスペースで実際には使われていない依存ごとに、エラーを出します。
- `warn` - ワークスペースで実際には使われていない依存ごとに警告を出しますが、チェックは失敗させません。
- `allow` - 使われていないワークスペースの依存のチェックを行いません。

### `highlight` {#highlight}

同じクレートの複数のバージョンが見つかり、`multiple-versions` が `warn` か `deny` に設定されている場合、`-g <dir>` オプションを使うと、各バージョンと、それがどのようにグラフに取り込まれたかを示す [dotgraph](https://www.graphviz.org/) を出力します。このフィールドは、グラフの色の付け方を決めます。これにより、取り除いたり更新したりするのによい候補を、すばやく見つけやすくなります。

- `lowest-version` - 最も低い重複バージョンへの経路を強調表示します。強調表示の色は ![赤](https://placehold.it/15/ff0000/000000?text=+) です
- `simplest-path` - グラフのルートまでの辺の総数が最も少ない重複バージョンへの経路を強調表示します。これは多くの場合、取り除いたりアップグレードしたりするのに最もよい候補です。強調表示の色は ![青](https://placehold.it/15/0000FF/000000?text=+) です。
- `all` - `lowest-version` と `simplest-path` の両方を強調表示します。両者が同じ場合は、![赤](https://placehold.it/15/ff0000/000000?text=+) だけで強調表示します。

![Imgur](https://i.imgur.com/xtarzeU.png)

### `deny` {#deny}

```toml
deny = ["package-spec"]
```

禁止する特定のクレートを決めます。各エントリには、cargo-deny の設定のほかの部分と同じ [PackageSpec](../cfg.html#package-specs) を使います。

#### `wrappers` {#wrappers}

```toml
deny = [{ crate = "crate-you-don't-want:<=0.7.0", wrappers = ["this-can-use-it"] }]
```

このフィールドを使うと、特定のクレートが禁止されたクレートに直接依存することを許し、それ以外の推移的な依存はすべて禁止できます。

#### `deny-multiple-versions` {#deny-multiple-versions}

```toml
multiple-versions = 'allow'
deny = [{ crate = "crate-you-want-only-one-version-of", deny-multiple-versions = true }]
```

このフィールドを使うと、ほかのすべてのクレートについては複数のバージョンを許可したり警告にしたりしつつ、特定のクレートについてだけ、そのクレート自身の複数のバージョンを禁止できます。このフィールドは、`wrappers` と同時には設定できません。

#### `deny.reason` {#denyreason}

```toml
deny = [{ crate = "package-spec", reason = "the reason this crate is banned"}]
```

このフィールドには、そのクレートを禁止する理由を文字列（簡単なメッセージや URL など）で書きます。この理由は診断の出力に表示されるので、利用者は、なぜそうなっているのかを調べるために履歴を掘り返したり、メンテナーに尋ねたりして時間をむだにせずに済みます。

#### `deny.use-instead` {#denyuse-instead}

```toml
deny = [{ crate = "openssl", use-instead = "rustls"}]
```

これは、特定のクレートを禁止する、最もよくあるケースのための略記です。つまり、その機能について、プロジェクトでは別のクレートを使うことにした、というケースです。

### `allow` {#allow}

```toml
allow = ["package-spec"]
```

許可する特定のクレートを決めます。`allow` のリストに 1 つ以上のエントリがあると、そのリストにないクレートはすべて禁止されます。注意して使ってください。各エントリには、cargo-deny の設定のほかの部分と同じ [PackageSpec](../cfg.html#package-specs) を使います。

### `allow-workspace` {#allow-workspace}

```toml
allow-workspace = false
```

`true` にすると、既定ですべてを禁止するポリシー（つまり `deny = [{ name = "*" }]` を指定している場合）を使っていても、ワークスペースのメンバーをすべて自動的に許可します。外部のクレートには厳しい依存の許可リストを適用しつつ、自分たちのワークスペースのクレートは、`allow` の設定に明示的に書かなくても自動的に許可したい組織にとって便利です。

`allow-workspace = true` のときは、次のようになります。

- ワークスペースのメンバーはすべて、自動的に `allow` のリストにあるものとして扱われます
- ワークスペースのメンバーは、明示的な `deny` のエントリよりも優先されます
- 外部の依存は、これまでどおり許可リストに明示的に入れる必要があります
- 単一のクレートのワークスペースでも、複数のクレートのワークスペースでも使えます

**既定値**: `false`

#### `allow.reason` {#allowreason}

```toml
allow = [{ crate = "package-spec", reason = "the reason this crate is allowed"}]
```

このフィールドには、そのクレートを許可する理由を文字列（簡単なメッセージや URL など）で書きます。この理由は診断の出力に表示されるので、利用者は、なぜそうなっているのかを調べるために履歴を掘り返したり、メンテナーに尋ねたりして時間をむだにせずに済みます。

### `external-default-features` {#external-default-features}

ワークスペースにないクレートで `default` フィーチャが有効になっているときに使う、リントレベルを決めます。必要なら、この リントレベルをクレートごとに上書きできます。

たとえば、`an-external-crate` で `default` フィーチャが有効になっている場合、それを明示的に許可できます。

```toml
[bans]
external-default-features = "deny"

[[bans.features]]
crate = "an-external-crate"
allow = ["default"]
```

### `workspace-default-features` {#workspace-default-features}

`external-default-features` のワークスペース版です。

```toml
[bans]
external-default-features = "allow"

[[bans.features]]
crate = "a-workspace-crate"
deny = ["default"]
```

### `features` {#features}

```toml
[[bans.features]]
crate = "featured-krate:1.0"
deny = ["bad-feature"]
allow = ["good-feature"]
exact = true
```

クレートごとに、フィーチャの allow/deny のリストを指定できます。各エントリには、cargo-deny の設定のほかの部分と同じ [PackageSpec](../cfg.html#package-specs) を使います。

#### `features.deny` {#featuresdeny}

そのクレートの特定のフィーチャを禁止します。

#### `features.allow` {#featuresallow}

そのクレートの特定のフィーチャを許可します。有効になっているフィーチャのうち、このリストにないものは禁止されます。

#### `features.exact` {#featuresexact}

指定すると、`allow` にあるフィーチャが、クレートで有効になっているフィーチャと完全に一致することを求めます。許可されているのに有効になっていないフィーチャがあると、失敗します。

### `skip` {#skip}

```toml
skip = [
    "package-spec",
    { crate = "package-spec", reason = "an old version is used by crate-x, see <PR link> for updating it" },
]
```

重複したバージョンを禁止していると、1 つ以上の重複がなくなるまで、たとえば PR が受け入れられて新しいバージョンが公開されるのを、しばらく待たなければならないことがよくあります。`skip` フィールドを使うと、必要がなくなるまでのあいだ、重複の検出であるクレートを一時的に無視して、エラーを出さないようにできます。

`skip` のリストにあるクレートには、特定のバージョン制約を使うことをおすすめします。`skip` のリストにあるエントリが、グラフの中のどのクレートにも一致しなくなると、cargo-deny が警告を出すので、設定を整理できるからです。

各エントリには、cargo-deny の設定のほかの部分と同じ [PackageSpec](../cfg.html#package-specs) を使います。

### `skip-tree` {#skip-tree}

```toml
skip-tree = [
    "windows-sys<=0.52", # このクレートと、その直接・推移的な依存の _すべて_ をスキップします
    { crate = "windows-sys<=0.52", reason = "several crates use the outdated 0.42 and 0.45 versions" },
    { crate = "windows-sys<=0.52", depth = 3, reason = "several crates use the outdated 0.42 and 0.45 versions" },
]
```

重複したバージョンに対処していると、ある特定のクレートが、連鎖的な影響の中心になっていることがよくあります。たとえば、alpha や beta のあいだ、特定のクレートの最先端のバージョンを使っている場合があります。反対に、エコシステムのほかの大部分がより新しいバージョンに移ったのに、あるクレートがひどく古い依存を使っている場合もあります。どちらの場合も、そのクレートが取り込む推移的な依存のうち、ほかの依存と衝突するものを 1 つずつ明示的に `skip` するのは、かなり面倒です。そこで `skip-tree` の出番です。

`skip-tree` のエントリは、スキップするクレートの名前とバージョンの範囲を指定するという点では、`skip` と似ています。ただし、`depth` フィールドもあり、そのクレートから何階層下までを一緒にスキップするかを指定できます。depth が `0` なら、そのクレートを `skip` フィールドに指定するのと同じです。

既定では、`depth` は無限であることに注意してください。

各エントリには、cargo-deny の設定のほかの部分と同じ [PackageSpec](../cfg.html#package-specs) を使います。

**注:** `skip-tree` は非常に大ざっぱな手段なので、注意して使ってください。

### `std-replacements` {#std-replacements}

`std-replacements` フィールドでは、crates.io から取得したクレートを [`std-replacement-data`] と照らし合わせてチェックするかどうか、またどのようにチェックするかを設定します。[`std-replacement-data`] には、`std` や `core` に一部または全部が実装されたクレートについての情報が入っています。

既定では、このフィールドがなければ、このチェックはまったく行われません。

#### `std-replacements.scope` {#std-replacementsscope}

どのクレートを対象にするかの範囲です。この範囲で絞り込まれるのは、[`std-replacement-data`] に載っているクレートに依存しているクレートであって、載っているクレートそのものではありません。

- `workspace`（既定値） - ワークスペースの 1 つ以上のクレートから依存されているクレートだけを対象にします。
- `all` - データにあるクレートは、グラフの中にあればすべて対象にします。
- `transitive` - ワークスペースの外にあるクレートから依存されているクレートだけを対象にします。
- `none` - どのクレートも対象にしません。これは、このチェックを無効にするもう 1 つの方法です。

#### `std-replacements.ignore-rust-version` {#std-replacementsignore-rust-version}

クレートの `rust-version` を無視する範囲です。既定では `none` で、`rust-version` は尊重されます。つまり、[`std-replacement-data`] にあるクレートに依存しているクレートが lint を発生させるのは、そのうち少なくとも 1 つが宣言している `rust-version` が、std による置き換えの API が安定化されたバージョンの少なくとも 1 つ以上（>=）である場合だけです。

- `none`（既定値） - `rust-version` を常に考慮します
- `workspace` - ワークスペースのメンバーから依存されているクレートについて、`rust-version` を無視します
- `all` - `rust-version` を一切考慮しません
- `transitive` - ワークスペースにないクレートから依存されているクレートについて、`rust-version` を無視します

#### `std-replacements.rust-version` {#std-replacementsrust-version}

自分で `rust-version` を指定していないクレートに使う `rust-version` です。指定しない場合、[`std-replacement-data`] にあるバージョンは考慮されません。この値は semver に従っている必要はなく、単純な `<major>.<minor>` でも構いません。Rust がパッチリリースで機能を追加することはほとんど（あるいはまったく）なく、メジャーバージョンも 1 つしかないので、照合のときにはマイナーバージョンだけが考慮されることに注意してください。

#### `std-replacements.ignore` {#std-replacementsignore}

本来なら lint を発生させるクレートを、[PackageSpec](../cfg.html#package-specs) で指定して無視します。

#### `std-replacements.level` {#std-replacementslevel}

クレートが [`std-replacement-data`] にあり、必要な条件をすべて満たしたときに出る診断の、リントレベルです。

- `deny`（既定値）
- `warn`
- `allow`

### `build` {#build}

`build` フィールドには、コンパイル時に実行されるクレートについて診断を出すための設定が入ります。コンパイル時に実行されるのは、[ビルドスクリプト](https://doc.rust-lang.org/cargo/reference/build-scripts.html)を持っているクレートか、[手続き型マクロ](https://doc.rust-lang.org/reference/procedural-macros.html)のクレートです。この設定は（今のところ）、拡張子の glob パターンで設定した特定の種類のファイルと、実行ファイルについての診断に重点を置いています。実行ファイルには、ネイティブのものと、[shebang 付きのインタプリタ向けスクリプト](<https://en.wikipedia.org/wiki/Shebang_(Unix)>)の形のものがあります。

この設定のねらいは、ビルド済みのバイナリやスクリプトを持っている、または使っているクレートや、そのほか気をつけておきたい種類のファイルを含んでいるクレートに、注意を向けてもらうことです。しかし、cargo-deny が提供するコンパイル時のクレートの lint は、積極的に悪意のあるコードからあなたを守るもの**ではありません**。

cargo-deny が**検出しない**ものを、ざっと挙げます。

- クレートが、ふつうの Rust のコードを使って、SSH の鍵をリモートのサーバーにアップロードするなど、あからさまに悪いことをする場合
- クレートに、圧縮されたり、そのほかの方法で難読化されたりした実行可能なバイナリが含まれている場合
- ビルドスクリプトが `include!()` を使っていて、あるバージョンでは無害なコードを取り込み、その後、ビルドスクリプトの内容自体のチェックサムの不一致を起こさずに、それを悪意のあるものに置き換える場合
- 悪意のないクレートの、ビルド時の依存が、上のどれかを行う場合
- 私はセキュリティの専門家ではないので思いつかない、そのほかの数多くのこと

つまり、`cargo-deny` が（今のところ）本当に役立つのは、クレートがネイティブの実行ファイルを持っている場合や、クレートのメンテナーが、CI やリリース管理などのための補助スクリプトを、忘れたり意図的に残したりしてクレートのソースに入れている場合（実際には自動で実行されることのないもの）を分析するときだけです。

#### `allow-build-scripts` {#allow-build-scripts}

ビルドスクリプトを持つことが許可されるクレートを、すべて指定します。このオプションを省略すると、すべてのクレートがビルドスクリプトを持つことを許可されます。このオプションを空のリストにすると、どのクレートもビルドスクリプトを持つことを許可されません。

#### `executables` {#executables}

ネイティブの実行ファイルをどう扱うかを制御します。このチェックは、実際にディスクからファイルのヘッダーを読み込んで行うことに注意してください。そのため、このチェックは Windows でも動作します。つまり、実行ビットは関係ありません。

- `deny`（既定値） - ネイティブの実行ファイルが見つかると、エラーを出します。
- `warn` - ネイティブの実行ファイルが見つかると警告を表示しますが、チェックは失敗させません。
- `allow` - ネイティブの実行ファイルが見つかると注記を表示しますが、チェックは失敗させません。

このチェックが今のところ扱うのは、主な実行ファイルの形式だけです。

- [ELF](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format)
- [PE](https://en.wikipedia.org/wiki/Portable_Executable)
- [Mach-O](https://en.wikipedia.org/wiki/Mach-O)

#### `interpreted` {#interpreted}

インタプリタ向けのスクリプトをどう扱うかを制御します。このチェックは、実際にディスクからファイルのヘッダーを読み込んで行うことに注意してください。そのため、このチェックは Windows でも動作します。つまり、実行ビットは関係ありません。

- `deny` - インタプリタ向けのスクリプトが見つかると、エラーを出します。
- `warn` - インタプリタ向けのスクリプトが見つかると警告を表示しますが、チェックは失敗させません。
- `allow`（既定値） - インタプリタ向けのスクリプトが見つかると注記を表示しますが、チェックは失敗させません。

#### `script-extensions` {#script-extensions}

指定すると、コンパイル時に実行されるクレートをスキャンして、指定した拡張子のファイルを探し、一致したファイルごとにエラーを出します。

#### `enable-builtin-globs` {#enable-builtin-globs}

`true` にすると、python など、たいていの開発者のマシンにインストールされていることが多い、一般的な言語のための組み込みの glob パターンを有効にします。

```toml
# スクリプト言語や、手軽に実行できるコンパイル言語のための「一般的な」パターンの一覧です。
# これらは、ビルドスクリプトや proc macro でローカルに実行するのに使われる可能性があります。
# 言うまでもなく、この一覧は完全ではなく、完全になることもありえません。
# ほとんどの場合、拡張子は人間のためのものにすぎず、ほとんどのツールは拡張子に関係なく、
# 実行できるスクリプトやコードを平気で実行します（シェルスクリプトなど）。
# それに加えて、_本当に_ 悪意のあるクレートなら、必要に応じてファイルを生成したり、
# リモートの場所からダウンロードしたり、実のところ、何でもできます
globs = [
    "*.bat", "*.cmd", # batch
    "*.go", # Go `go run <file.go>`
    "*.java", # Java `java <file.java>`
    "*.js", "*._js", "*.es", "*.es6", # javascript
    "*.pl", "*.perl", "*.pm", # perl
    "*.6pl", "*.6pm", "*.p6", "*.p6l", "*.p6m", "*.pl6", "*.pm6", "*.t", # perl 6
    "*.ps1", "*.psd1", "*.psm1", # powershell
    "*.py", # python
    "*.rb", # ruby
    "*.sh", "*.bash", "*.zsh", # shell
]
```

#### `include-dependencies` {#include-dependencies}

既定では、コンパイル時に実行されるクレートだけをスキャンします。`true` にすると、このフィールドにより、そのクレートに加えて、その依存もすべてチェックします。このオプションが既定で無効になっているのは、有効にしても、公開されたクレートに残された CI のスクリプトが見つかるくらいのことが多いからです。

#### `include-workspace` {#include-workspace}

`true` にすると、ワークスペースのクレートもスキャンします。既定では false です。自分のコードは、ある程度信頼しているはずだからです。

#### `include-archives` {#include-archives}

`true` にすると、アーカイブファイル（Windows の .lib、Unix の .a、C++ の .o オブジェクトファイルなど）も、ネイティブのコードとして数えます。既定では false です。これらは、実行できるようになる前にリンクが必要なことが多いからです。

#### `bypass` {#bypass}

ここまでの設定はすべて、コンパイル時のクレートに対して実行されるグローバルなチェックを設定するためのものでした。一方、`allow` フィールドは、それらの lint をクレートごとに抑えるための手段です。

各エントリには、cargo-deny の設定のほかの部分と同じ [PackageSpec](../cfg.html#package-specs) を使います。

```toml
[build.bypass]
crate = "crate-name"
```

##### `build-script` と `required-features` {#build-script-and-required-features}

`build-script` フィールドに、64 文字の 16 進数による有効な [SHA-256](https://en.wikipedia.org/wiki/SHA-2) を設定すると、クレートのビルドスクリプトのチェックサムがユーザーの指定したチェックサムと一致し、**かつ** `required-features` フィールドで指定したフィーチャがどれも有効になっていない _場合に_、残りのスキャンが省略されます。チェックサムが一致しない場合は、計算したチェックサムが警告として出され、そのクレートは、チェックサムが指定されていない場合と同じようにスキャンされます。

**注:** これらのオプションが適用されるのは、ビルドスクリプトを持つクレートだけで、proc macro には適用されません。proc macro には、簡単にチェックサムを計算できる単一のエントリポイントがないからです。

```toml
[[build.bypass]]
name = "crate-name"
build-script = "5392f0e58ad06e089462d93304dfe82337acbbefb87a0749a7dc2ed32af04af7"
```

##### `allow-globs` {#allow-globs}

指定した 1 つ以上の glob パターンに一致するファイルの、スキャンを省略します。すべてのクレートに適用される [`script-extensions`](#the-script-extensions-field-optional) フィールドとは違い、これらの glob は拡張子だけでなく、何にでも一致させられることに注意してください。

```toml
[build]
script-extensions = ["cs"]

[[build.bypass]]
crate = "crate-name"
allow-globs = [
    "scripts/*.cs",
]
```

##### `bypass.allow` {#bypassallow}

1 つのファイルのスキャンを省略します。

```toml
[build]
executables = "deny"

[[build.bypass]]
crate = "crate-name"
allow = [
    { path = "bin/x86_64-linux", checksum = "5392f0e58ad06e089462d93304dfe82337acbbefb87a0749a7dc2ed32af04af7" }
]
```

###### `path` {#path}

スキャンを省略するファイルの、クレートのルートからの相対パスです。

###### `checksum` {#checksum}

ファイルの、64 文字の 16 進数による [SHA-256](https://en.wikipedia.org/wiki/SHA-2) チェックサムです。チェックサムが一致しない場合は、エラーが出ます。

[`std-replacement-data`]: https://github.com/rust-lang/std-replacement-data
{% endraw %}
