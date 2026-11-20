{% raw %}
# `init` コマンド {#the-init-command}

cargo-deny の設定は少し複雑なので、テンプレートから設定ファイルを作る `init` コマンドを用意しています。cargo-deny でプロジェクトをどのようにリントするかを設定するときの、出発点として使えます。

`init` コマンドは次のように使います。

```bash
cargo deny init
```

## パスを指定する {#specify-a-path}

ルートの `--config` オプションを使うと、作られる設定ファイルの既定のパス(`<cwd>/deny.toml`)を変更できます。

```bash
cargo deny --config path/to/config.toml init
```

## テンプレート {#template}

現在の作業ディレクトリに `deny.toml` ファイルが作られます。その中身は[このテンプレート](https://github.com/EmbarkStudios/cargo-deny/blob/main/deny.template.toml)をそのままコピーしたものです。

```toml
# このテンプレートには、指定できるすべてのセクションと、その既定値が入っています

# リントレベルを取るフィールドは、すべて次の値を取れることに注意してください。
# * deny - エラーが出て、チェックが失敗します
# * warn - 警告が出ますが、チェックは失敗しません
# * allow - 警告もエラーも出ません。ただし、場合によっては注記が
# 出ます

# このテンプレートに書かれている値は、自分の設定でセクションやフィールドを
# 指定しなかったときに使われる既定値です

# ルートのオプション

# graph テーブルでは、依存グラフの組み立て方、つまり
# どのクレートに対してチェックを行うかを設定します
[graph]
# 1 つ以上のターゲットトリプル(と、必要に応じて target_features)を指定すると、
# `cargo deny check` を実行したときに、指定したターゲットだけがチェックされます。
# たとえば、`nix` クレートが `target_family = "unix"` の設定でだけ使われるように、
# あるパッケージがターゲット固有の依存としてしか使われない場合、
# このリストに windows のターゲットしかなければ、nix クレートと、
# ほかのクレートと共有されていないその依存はすべて無視されます。
# ここのターゲットのリストは、実質的にどのターゲット向けにビルドするかを示しているからです。
targets = [
    # トリプルにはどんな文字列でも書けますが、実際の設定式と照らし合わせて
    # チェックできるのは、rustc に組み込まれたターゲットトリプル(1.40 時点)だけです
    #"x86_64-unknown-linux-musl",
    # 特定のターゲットで有効にすると約束する target_features も指定できます。
    # target_features は今のところ、そのターゲットのアーキテクチャが実際に
    # 対応している有効なフィーチャと照らし合わせた検証は行われません。
    #{ triple = "wasm32-unknown-unknown", features = ["atomics"] },
]
# チェックの実行時に正しい情報の基準として使う依存グラフを作るとき、
# このフィールドを使うとグラフからクレートを刈り取り、cargo-deny から
# 見えないようにできます。これはとても強力な手段です。クレートをグラフから
# 刈り取ると、その依存も、刈り取られていないグラフ内のほかのクレートに
# つながっていないかぎり、すべて刈り取られます。そのため、注意して使ってください。
# 識別子は [Package ID Specifications]
# (https://doc.rust-lang.org/cargo/reference/pkgid-spec.html) です
#exclude = []
# true にすると、`--all-features` を付けてメタデータを集めます。true にした場合、
# これを無効に切り替えることはできないことに注意してください。条件に応じて
# `--all-features` を有効にしたい場合は、代わりにコマンドラインで `--all-features` を
# 渡すことをおすすめします
all-features = false
# true にすると、`--no-default-features` を付けてメタデータを集めます。
# `all-features` と同じ注意点が当てはまります
no-default-features = false
# 設定すると、メタデータを集めるときにこれらのフィーチャが有効になります。
# コマンドラインで `--features` を指定した場合は、このオプションよりそちらが優先されます。
#features = []

# output テーブルでは、診断をどのように出力するか、出力するかどうかを設定します
[output]
# フィーチャを含む包含グラフを診断に出力するとき、このオプションで
# フィーチャの辺を追加する深さを指定できます。
# このオプションがあるのは、グラフがかなり大きくなることがあり、クレートから
# すべてのグラフのルートまでのフィーチャを加えると、冗長になりすぎることがあるためです。
# このオプションは、コマンドラインの `--feature-depth` で上書きできます
feature-depth = 1

# このセクションは、`cargo deny check advisories` を実行したときに使われます
# advisories セクションについてのくわしいドキュメントは、次の場所にあります。
# https://embarkstudios.github.io/cargo-deny/checks/advisories/cfg.html
[advisories]
# 助言のデータベースをクローンまたは取得する先のパス
#db-path = "$CARGO_HOME/advisory-dbs"
# 使う助言のデータベースの URL
#db-urls = ["https://github.com/rustsec/advisory-db"]
# 無視する助言の ID のリストです。無視した助言も、見つかったときには
# 注記が出力されることに注意してください。
ignore = [
    #"RUSTSEC-0000-0000",
    #{ id = "RUSTSEC-0000-0000", reason = "you can specify a reason the advisory is ignored" },
    #"a-crate-that-is-yanked@0.1.1", # 必要なら、yank されたクレートのバージョンも無視できます
    #{ crate = "a-crate-that-is-yanked@0.1.1", reason = "you can specify why you are ignoring the yanked crate" },
]
# true にすると、cargo deny は git の実行ファイルを使って助言のデータベースを取得します。
# false にすると、組み込みの git ライブラリを使います。
# cargo-deny が対応していない特別な認証が必要な場合は、true にすると役に立つことがあります。
# git の認証の設定については、Git Authentication を見てください。
#git-fetch-with-cli = true

# このセクションは、`cargo deny check licenses` を実行したときに使われます
# licenses セクションについてのくわしいドキュメントは、次の場所にあります。
# https://embarkstudios.github.io/cargo-deny/checks/licenses/cfg.html
[licenses]
# 明示的に許可するライセンスのリスト
# 指定できるライセンスの一覧は https://spdx.org/licenses/ を見てください
# [指定できる値: SPDX 3.11 の任意の短い識別子(+ 必要に応じて例外)]。
allow = [
    #"MIT",
    #"Apache-2.0",
    #"Apache-2.0 WITH LLVM-exception",
]
# ライセンスの文章からライセンスを検出するときの確信度のしきい値です。
# 値が大きいほど、ライセンスの文章が、有効な SPDX ライセンスファイルの
# 正規のライセンス文にもっと近くなければなりません。
# [指定できる値: 0.0 から 1.0 までの任意の値]。
confidence-threshold = 0.8
# クレートごとに 1 つ以上のライセンスを許可します。こうすると、通常の allow リストの
# ように、特定のライセンスがあらゆるクレートで受け入れられることはありません
exceptions = [
    # 各エントリは、クレートとバージョンの制約、そしてそれ専用の許可
    # リストです
    #{ allow = ["Zlib"], crate = "adler32" },
]

# 機械で(簡単に)読めるライセンス情報を持たないクレートもあります。
# そのようなクレートに明確化のエントリを追加すると、ライセンス情報を
# 手動で指定できます
#[[licenses.clarify]]
# 明確化を適用するパッケージ指定
#crate = "ring"
# クレートのライセンス要件を表す SPDX 式
#expression = "MIT AND ISC AND OpenSSL"
# ライセンス式の「正しい情報の基準」として使う、クレートのソースにある
# 1 つ以上のファイルです。内容が一致すれば、ライセンスのチェックの実行時に
# 明確化が使われます。一致しなければ明確化は無視され、クレートは通常どおり
# チェックされます。その場合、ほかの設定によっては警告やエラーが
# 出ることがあります
#license-files = [
# 各エントリは、クレートから見た相対パスと、その内容の(中身を意識しなくてよい)ハッシュです
#{ path = "LICENSE", hash = 0xbd0eed23 }
#]

[licenses.private]
# true にすると、公開されていないワークスペースのクレートや、非公開の
# レジストリにだけ公開されているワークスペースのクレートを無視します。
# クレートを(公式のレジストリに)公開しないものとして指定する方法については、
# https://doc.rust-lang.org/cargo/reference/manifest.html#the-publish-field を見てください。
ignore = false
# クレートを公開する可能性のある、1 つ以上の非公開のレジストリです。クレートが
# 非公開のレジストリにだけ公開されていて、ignore が true の場合、そのクレートの
# ライセンスはチェックされません
registries = [
    #"https://sekretz.com/registry
]

# このセクションは、`cargo deny check bans` を実行したときに使われます。
# 'bans' セクションについてのくわしいドキュメントは、次の場所にあります。
# https://embarkstudios.github.io/cargo-deny/checks/bans/cfg.html
[bans]
# 同じクレートの複数のバージョンが検出されたときのリントレベル
multiple-versions = "warn"
# クレートのバージョン要件が `*` のときのリントレベル
wildcards = "allow"
# 複数のバージョンを持つクレートの dot グラフを作るときに使う、
# グラフの強調表示の方法
# * lowest-version - いちばん低いバージョンの重複へのパスを強調表示します
# * simplest-path - 辺がいちばん少ないバージョンへのパスを強調表示します
# * all - lowest-version と simplest-path の両方を使います
highlight = "all"
# チェック対象のワークスペースのメンバーであるクレートの `default` フィーチャに
# 対する既定のリントレベルです。必要なら、クレートごとに `default` を
# allow または deny にして上書きできます。
workspace-default-features = "allow"
# ワークスペースのメンバーでない外部のクレートの `default` フィーチャに対する
# 既定のリントレベルです。必要なら、クレートごとに `default` を allow または deny に
# して上書きできます。
external-default-features = "allow"
# 許可するクレートのリストです。注意して使ってください!
allow = [
    #"ansi_term@0.11.0",
    #{ crate = "ansi_term@0.11.0", reason = "you can specify a reason it is allowed" },
]
# true にすると、既定で deny にしている場合でも、ワークスペースのメンバーは自動的に許可されます
# 既定ですべての外部の依存を deny にしたいが、自分たちのワークスペースのクレートは
# 1 つずつ明示しなくても許可したい組織に便利です
allow-workspace = false
# 拒否するクレートのリスト
deny = [
    #"ansi_term@0.11.0",
    #{ crate = "ansi_term@0.11.0", reason = "you can specify a reason it is banned" },
    # ラッパークレートを指定することもできます。そうすると、本来は禁止されている
    # クレートでも、ラッパークレートの直接の依存であれば許可されます
    #{ crate = "ansi_term@0.11.0", wrappers = ["this-crate-directly-depends-on-ansi_term"] },
]

# 許可または拒否するフィーチャのリスト
# 各エントリはクレートの名前とバージョンの範囲です。バージョンを
# 指定しない場合は、すべてのバージョンに一致します。
#[[bans.features]]
#crate = "reqwest"
# 許可しないフィーチャ
#deny = ["json"]
# 許可するフィーチャ
#allow = [
#    "rustls",
#    "__rustls",
#    "__tls",
#    "hyper-rustls",
#    "rustls",
#    "rustls-pemfile",
#    "rustls-tls-webpki-roots",
#    "tokio-rustls",
#    "webpki-roots",
#]
# true にすると、許可するフィーチャが、有効なフィーチャの集合と完全に
# 一致しなければなりません。これを設定する場合、`deny` を設定する意味はありません
#exact = true

# 重複の検出のときにスキップする、特定のクレートとバージョン。
skip = [
    #"ansi_term@0.11.0",
    #{ crate = "ansi_term@0.11.0", reason = "you can specify a reason why it can't be updated/removed" },
]
# `skip` と同じように、重複の検出のときに特定のクレートをスキップできます。
# skip と違って、指定したクレートから始まる推移的な依存のツリー全体も、
# ある深さまで含めます。深さは既定では無限です。
skip-tree = [
    #"ansi_term@0.11.0", # その直接の依存と推移的な依存の _すべて_ と一緒にスキップされます
    #{ crate = "ansi_term@0.11.0", depth = 20 },
]

# このセクションは、`cargo deny check sources` を実行したときに使われます。
# 'sources' セクションについてのくわしいドキュメントは、次の場所にあります。
# https://embarkstudios.github.io/cargo-deny/checks/sources/cfg.html
[sources]
# allow リストにないクレートレジストリのクレートが見つかったときに、
# どうするかのリントレベル
unknown-registry = "warn"
# allow リストにない git リポジトリのクレートが見つかったときに、
# どうするかのリントレベル
unknown-git = "warn"
# 許可するクレートレジストリの URL のリストです。指定しない場合の既定値は
# crates.io のインデックスです。指定したが空の場合は、どのレジストリも許可されません。
allow-registry = ["https://github.com/rust-lang/crates.io-index"]
# 許可する Git リポジトリの URL のリスト
allow-git = []

[sources.allow-org]
# git の取得元を許可する github.com の組織
github = []
# git の取得元を許可する gitlab.com の組織
gitlab = []
# git の取得元を許可する bitbucket.org の組織
bitbucket = []
```
{% endraw %}
