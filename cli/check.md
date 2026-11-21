{% raw %}
# `check` コマンド {#the-check-command}

check コマンドは cargo-deny の主要なサブコマンドです。プロジェクトのすべてのクレートを実際に調べ、設定と照らし合わせてチェックするのがこのコマンドだからです。

## 引数 {#args}

### `[WHICH...]` {#which}

実行するチェックです。ここで 1 つ以上のチェックを指定しないかぎり、既定では**すべての**チェックが実行されます。

使えるチェックの一覧は [checks](../checks/index.html) を見てください。

## オプション {#options}

### `-A, --allow <ALLOW>` {#-a---allow-allow}

リントを allow に設定します。

```shell
$ cargo run -- --manifest-path tests/test_data/duplicates/Cargo.toml check bans
error[duplicate]: found 2 duplicate entries for crate 'webpki-roots'
    ┌─ /home/jake/code/cargo-deny/tests/test_data/duplicates/Cargo.lock:138:1
    │
138 │ ╭ webpki-roots 0.25.4 registry+https://github.com/rust-lang/crates.io-index
139 │ │ webpki-roots 0.26.11 registry+https://github.com/rust-lang/crates.io-index
    │ ╰──────────────────────────────────────────────────────────────────────────┘ lock entries
    │
    ├ webpki-roots v0.25.4
      └── minreq v2.13.4
          └── duplicates v0.1.0
    ├ webpki-roots v0.26.11
      └── duplicates v0.1.0

bans FAILED
$ cargo run -- --manifest-path tests/test_data/duplicates/Cargo.toml check -A duplicate bans
bans ok
```

### `--audit-compatible-output` {#--audit-compatible-output}

cargo-audit から cargo-deny へ移行しやすくするためのフラグです。このフラグを付けると、cargo-deny は cargo-audit とまったく同じ内容を出力します。出力先も cargo-audit と同じく、`stderr` ではなく `stdout` です。

このフラグは出力形式が JSON のときだけ適用されることに注意してください。また、cargo-deny は複数の助言のデータベースに対応しているので、JSON オブジェクトは 1 つではなく、助言のデータベースごとに 1 つずつ出力されることにも注意してください。

### `-D, --deny <DENY>` {#-d---deny-deny}

リントを deny に設定します。

```shell
$ cargo run -- --manifest-path tests/test_data/duplicates/Cargo.toml check -A duplicate bans
warning[unmatched-skip-root]: skip tree root was not found in the dependency graph
   ┌─ deny.toml:35:16
   │
35 │     { crate = "windows-sys", reason = "a foundational crate for many that bumps far too frequently to ever have a shared version" },
   │                ━━━━━━━━━━━ no crate matched these criteria

warning[unmatched-skip]: skipped crate 'getrandom = =0.2.17' was not encountered
   ┌─ deny.toml:31:16
   │
31 │     { crate = "getrandom@0.2.17", reason = "ring uses this old version" },
   │                ━━━━━━━━━━━━━━━━             ────────────────────────── reason
   │                │
   │                unmatched skip configuration

warning[unmatched-skip]: skipped crate 'hashbrown = =0.15.5' was not encountered
   ┌─ deny.toml:32:16
   │
32 │     { crate = "hashbrown@0.15.5", reason = "petgraph uses this old version" },
   │                ━━━━━━━━━━━━━━━━             ────────────────────────────── reason
   │                │
   │                unmatched skip configuration

bans ok
$ cargo run -- --manifest-path tests/test_data/duplicates/Cargo.toml check -A duplicate -D unmatched-skip bans
warning[unmatched-skip-root]: skip tree root was not found in the dependency graph
   ┌─ deny.toml:35:16
   │
35 │     { crate = "windows-sys", reason = "a foundational crate for many that bumps far too frequently to ever have a shared version" },
   │                ━━━━━━━━━━━ no crate matched these criteria

error[unmatched-skip]: skipped crate 'getrandom = =0.2.17' was not encountered
   ┌─ deny.toml:31:16
   │
31 │     { crate = "getrandom@0.2.17", reason = "ring uses this old version" },
   │                ━━━━━━━━━━━━━━━━             ────────────────────────── reason
   │                │
   │                unmatched skip configuration

error[unmatched-skip]: skipped crate 'hashbrown = =0.15.5' was not encountered
   ┌─ deny.toml:32:16
   │
32 │     { crate = "hashbrown@0.15.5", reason = "petgraph uses this old version" },
   │                ━━━━━━━━━━━━━━━━             ────────────────────────────── reason
   │                │
   │                unmatched skip configuration

bans FAILED
```

### `--feature-depth <FEATURE_DEPTH>` {#--feature-depth-feature_depth}

包含グラフで、フィーチャの辺を追加する深さを指定します

### `-g, --graph <GRAPH>` {#-g---graph-graph}

graph_output のルートディレクトリのパスです

指定すると、同じクレートの複数のバージョンが検出されるたびに、dotviz のグラフを作ります。

各ファイルは `<dir>/graph_output/<crate_name>.dot` に作られます。`<dir>/graph_output/*` は実行のたびに削除され、作り直されます。

### `--hide-inclusion-graph` {#--hide-inclusion-graph}

クレートの情報を表示するときに、包含グラフを表示しません

既定では、診断メッセージが特定のクレートに関するものである場合、cargo-deny は診断に逆向きの依存グラフを付け加えて、そのクレートがどのようにプロジェクトに取り込まれたかを示します。

```text
some diagnostic message

the-crate
├── a-crate
└── b-crate
    └── c-crate
```

### `-s, --show-stats` {#-s---show-stats}

ログレベルに関係なく、すべてのチェックの統計を表示します

### `-W, --warn <WARN>` {#-w---warn-warn}

リントを warn に設定します。

```shell
$ cargo run -- --manifest-path tests/test_data/duplicates/Cargo.toml check bans
error[duplicate]: found 2 duplicate entries for crate 'webpki-roots'
    ┌─ /home/jake/code/cargo-deny/tests/test_data/duplicates/Cargo.lock:138:1
    │
138 │ ╭ webpki-roots 0.25.4 registry+https://github.com/rust-lang/crates.io-index
139 │ │ webpki-roots 0.26.11 registry+https://github.com/rust-lang/crates.io-index
    │ ╰──────────────────────────────────────────────────────────────────────────┘ lock entries
    │
    ├ webpki-roots v0.25.4
      └── minreq v2.13.4
          └── duplicates v0.1.0
    ├ webpki-roots v0.26.11
      └── duplicates v0.1.0

bans FAILED
$ cargo run -- --manifest-path tests/test_data/duplicates/Cargo.toml check -W duplicate bans
warning[duplicate]: found 2 duplicate entries for crate 'webpki-roots'
    ┌─ /home/jake/code/cargo-deny/tests/test_data/duplicates/Cargo.lock:138:1
    │
138 │ ╭ webpki-roots 0.25.4 registry+https://github.com/rust-lang/crates.io-index
139 │ │ webpki-roots 0.26.11 registry+https://github.com/rust-lang/crates.io-index
    │ ╰──────────────────────────────────────────────────────────────────────────┘ lock entries
    │
    ├ webpki-roots v0.25.4
      └── minreq v2.13.4
          └── duplicates v0.1.0
    ├ webpki-roots v0.26.11
      └── duplicates v0.1.0

bans ok
```

## 終了コード {#exit-codes}

[0.14.1](https://github.com/EmbarkStudios/cargo-deny/releases/tag/0.14.1) から、check コマンドの終了コードは、実行したチェックのうち 1 つ以上のエラーがあったもののビットセットになっています。

スクリプトやプログラムは、次の値を使って、どのチェックが失敗したかを正確に判断できます。

- `advisories` - `0x1`
- `bans` - `0x2`
- `licenses` - `0x4`
- `sources` - `0x8`
{% endraw %}
