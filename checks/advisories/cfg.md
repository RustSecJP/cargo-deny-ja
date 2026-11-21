{% raw %}
# `[advisories]` セクション {#the-advisories-section}

`cargo deny check advisories` の設定がすべて入ります。

## 既定の設定のリファレンス {#default-config-reference}

```toml
[advisories]
db-urls = ["https://github.com/RustSec/advisory-db"]
db-path = "~/.cargo/advisory-dbs" # または $CARGO_HOME/advisory-dbs が解決される場所
yanked = "warn"
unmaintained = "workspace"
unsound = "workspace"
maximum-db-staleness = "P90D" # 助言データベースの取得が無効になっているときだけチェックされます
unused-ignored-advisory = "warn"
```

以下のフィールドは、すべて省略できます。`[advisories]` セクションを丸ごと省いた場合も、advisories チェックは各フィールドに記載の既定値を使って実行されます。たとえば、既定では [RustSec Advisory DB](https://github.com/RustSec/advisory-db) が取得され、チェックに使われます。

## 設定例 {#example-config}

```toml
[advisories]
db-path = "~/.cargo/advisory-dbs"
db-urls = ["https://github.com/RustSec/advisory-db"]
ignore = [
    "RUSTSEC-0000-0000",
    "crate@0.1",
    { id = "RUSTSEC-0000-0001", reason = "has a deadline", expiry = "2026-12-31" },
    { crate = "yanked", reason = "a new version has not been released" },
]
unmaintained = "workspace"
unused-ignored-advisory = "warn"
```

### `db-urls` {#db-urls}

1 つ以上の助言データベースの URL です。

既定値: [RustSec Advisory DB](https://github.com/RustSec/advisory-db)

### `db-path` {#db-path}

1 つ以上の助言データベースをクローンする先の、ルートディレクトリのパスです。

この値では、基本的なシェル展開が使えます。

- `~` - [`home::home_dir`](https://docs.rs/home/latest/home/fn.home_dir.html) に展開されます
- `$VARNAME` - [`std::env::var("VARNAME")`](https://doc.rust-lang.org/std/env/fn.var.html) に展開されます
- `${VARNAME}` - [`std::env::var("VARNAME")`](https://doc.rust-lang.org/std/env/fn.var.html) に展開されます
- `${VARNAME:-fallback}` - [`std::env::var("VARNAME")`](https://doc.rust-lang.org/std/env/fn.var.html) に展開されます。存在しない場合は、フォールバックの値（`:-` と `}` のあいだにあるすべて）に展開されます
- `$CARGO_HOME` - [`std::env::var("CARGO_HOME")`](https://doc.rust-lang.org/std/env/fn.var.html) が存在すればそれに展開され、存在しなければ `$(home::home_dir())/.cargo` に展開されます

展開したあとのパスは、有効な utf-8 でなければならないことに注意してください。

既定値: `$CARGO_HOME/advisory-dbs`

### `version` {#version}

```toml
version = 2
```

version フィールドは（この文書を書いている時点では）もう使われていません。次のフィールドは削除され、今では指定するとエラーになります。

- `vulnerability` - 削除されました。脆弱性の助言は、今ではすべてエラーになります。
- `unsound` - 削除されました。unsound の助言は、今ではすべてエラーになります。
- `notice` - 削除されました。notice の助言は、今ではすべてエラーになります。
- `severity-threshold` - 削除されました。脆弱性の助言は、今ではすべてエラーになります。

これまでと同じく、特定の助言を無視したい場合は、`ignore` フィールドに追加してください。

### `yanked` {#yanked}

ソースのレジストリから yank されたバージョンのクレートが見つかったときに、どうするかを決めます。

- `deny` - yank されたクレートの名前とバージョンを示すエラーを出し、チェックを失敗させます。
- `warn`（既定値） - yank されたクレートの名前とバージョンを示す警告を表示しますが、チェックは失敗させません。
- `allow` - yank されたクレートについての注記を表示しますが、チェックは失敗させません。

### `disable-yank-checking` {#disable-yank-checking}

ローカルのレジストリインデックスを調べて、yank されたクレートをチェックするかどうかを制御します（[`yanked`](#yanked) を参照してください）。

- `false`（既定値） - ローカルのレジストリインデックスを調べて、yank されたクレートをチェックします。
- `true` - yank のチェックを無効にします。ローカルのレジストリインデックスは調べません。

### `ignore` {#ignore}

```toml
ignore = [
   "RUSTSEC-0000-0000",
   { id = "RUSTSEC-0000-0000", reason = "this vulnerability does not affect us as we don't use the particular code path" },
   { id = "RUSTSEC-0000-0001", reason = "waiting on an upstream fix", expiry = "2026-12-31" },
   "yanked@0.1.1",
   { crate = "yanked-crate@0.1.1", reason = "a semver compatible version hasn't been published yet" },
]
```

助言データベースの各助言には、`RUSTSEC-2019-0001` のような一意の識別子があります。この配列に識別子を入れると、その助言は警告やエラーではなく、注記として扱われます。

さらに、[PackageSpec](../cfg.html#package-specs) を、省略可能な `reason` とともに指定すると、yank されたクレートのバージョンを無視できます。

既定値: 空（助言も yank されたクレートも無視しません）。

#### `expiry` {#expiry}

```toml
ignore = [
   { id = "RUSTSEC-0000-0000", expiry = "P30D" },
]
```

期限が来るまでのあいだだけ、助言を無視できるようにします。期限が過ぎると、その助言はふだんどおり報告され、`expiry` の値を指す `advisory-ignore-expired` 診断が出ます。

- 助言の公開日を基準とした[期間](#durations)。
- ISO-8601 形式、つまり `YYYY-MM-DD` の絶対的な日付。

個々の ignore に期限がない場合は、[`ignore-expiry`](#ignore-expiry) があればそれを使います。

#### `allow` {#allow}

```toml
ignore = [
   { id = "RUSTSEC-0000-0000", allow = ['some-crate@0.1.0'] },
]
```

助言の対象となるクレートに直接依存してよいクレートの、[PackageSpec](../cfg.html#package-specs) のリストです。これにより、助言を無視しつつも、新しいクレートや更新によって、助言の対象のクレートへの依存がさらに増えることは許さないようにできます。

### `ignore-expiry` {#ignore-expiry}

```toml
ignore-expiry = "P90D"
# または絶対的な日付
# ignore-expiry = "2026-12-31"
```

特定の助言に [`expiry`](#expiry) がない場合に使われる、既定の期限です。

### `unmaintained` {#unmaintained}

```toml
unmaintained = 'workspace'
```

保守されていないことを示す助言を、エラーにするかどうかを決めます。保守されていないことによるエラーも、[`ignore`](#ignore) オプションで個別に無視できます。

- `all` - 保守されていないことを示す助言に当てはまるクレートは、すべて失敗になります
- `workspace`（既定値） - 保守されていないことを示す助言は、1 つ以上のワークスペースのクレートが直接依存しているクレートに当てはまる場合だけ失敗になります。
- `transitive` - 保守されていないことを示す助言は、1 つ以上のワークスペースのクレートが直接依存して**いない**クレートに当てはまる場合だけ失敗になります。
- `none` - 保守されていないことを示す助言は、完全に無視されます。

### `unsound` {#unsound}

```toml
unsound = 'workspace'
```

unsound の助言を、エラーにするかどうかを決めます。unsound によるエラーも、[`ignore`](#ignore) オプションで個別に無視できます。

- `all` - unsound の助言に当てはまるクレートは、すべて失敗になります
- `workspace`（既定値） - unsound の助言は、1 つ以上のワークスペースのクレートが直接依存しているクレートに当てはまる場合だけ失敗になります。
- `transitive` - unsound の助言は、1 つ以上のワークスペースのクレートが直接依存して**いない**クレートに当てはまる場合だけ失敗になります。
- `none` - unsound の助言は、完全に無視されます。

### `git-fetch-with-cli` {#git-fetch-with-cli}

非推奨です。`cargo-deny` は今では常に git の CLI を使って取得します。このフィールドを使うと非推奨の警告が出ますが、効果はありません。

### `maximum-db-staleness` {#maximum-db-staleness}

データベースが古くなったとみなされ、エラーが出るまでの、最大の[期間](#durations)です。これがチェックされるのは、`--offline` フラグか `check --disable-fetch` フラグで、助言データベースの取得が無効になっているときだけです。そうでない場合は、データベースは常にクローンまたは取得され、リモートの git リポジトリと同じ最新の状態になるからです。

指定しない場合の既定値は、`cargo-audit` が使っていて、`cargo-deny` もこれまで使ってきたのと同じ値、つまり `P90D`（90 日）です。

### `unused-ignored-advisory` {#unused-ignored-advisory}

`ignore` のリストにある助言のどれかが、依存グラフの中で見つからなかったときに、どうするかを決めます。

- `warn`（既定値） - `advisories.ignore` にあるものの、どのクレートでも使われなかった助言ごとに、警告を出します。
- `allow` - `advisories.ignore` のリストにある使われなかった助言を、無視します。
- `deny` - `advisories.ignore` のリストに使われなかった助言があると、エラーになり、advisories チェックが失敗します。

## 期間 {#durations}

期間には [RFC3339](https://www.rfc-editor.org/info/rfc3339/) の形式を使います。これは、それなりに標準的な形式だからです……ただし、ドキュメントはあまり整っていません。公式の文法は次のとおりです。

```txt
   dur-second        = 1*DIGIT "S"
   dur-minute        = 1*DIGIT "M" [dur-second]
   dur-hour          = 1*DIGIT "H" [dur-minute]
   dur-time          = "T" (dur-hour / dur-minute / dur-second)
   dur-day           = 1*DIGIT "D"
   dur-week          = 1*DIGIT "W"
   dur-month         = 1*DIGIT "M" [dur-day]
   dur-year          = 1*DIGIT "Y" [dur-month]
   dur-date          = (dur-day / dur-month / dur-year) [dur-time]

   duration          = "P" (dur-date / dur-time / dur-week)
```

ただし、私の知るかぎり、この期間の形式について、実行できる公式の仕様準拠テストはありません。また、ほかの言語で書かれたパーサーをいくつか見てみましたが、どれも……実際には文法にきちんと従っていないようでした。そのため、cargo-deny の実装は仕様に照らすと間違っている _かもしれません_。それでも、少なくとも一貫して間違っています。

仕様では小数点として `,` も使えますが、cargo-deny では簡単にするため、小数点として `.` だけに対応していることに注意してください。

最後にもう 1 つ。この形式には、正確でない単位が 2 つあります。年の 'Y' と、月の 'M' です。そのため、どちらも使うことはおすすめしません。もし使う場合は、次のように計算されます。

- 1 年 = 365 日
- 1 か月 = 30.43 日
{% endraw %}
