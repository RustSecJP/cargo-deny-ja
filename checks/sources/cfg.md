{% raw %}
# `[sources]` セクション {#the-sources-section}

`cargo deny check sources` のすべての設定が入ります

## 設定例 {#example-config}

```toml
[sources]
unknown-registry = "allow"
unknown-git = "deny"
required-git-spec = "tag"
allow-registry = [
    "https://sekretz.com/registry/index",
    "sparse+https://fake.sparse.com",
]
allow-git = [
    "https://notgithub.com/orgname/reponame.git",
]
private = [
    "https://internal-host/repos",
]
unused-allowed-source = "warn"
[sources.allow-org]
github = [
    "yourghid",
    "YourOrg",
]
gitlab = [
    "gitlab-org",
]
bitbucket = [
    "atlassian",
]
```

### `unknown-registry` {#unknown-registry}

`allow-registry` リストにないクレートレジストリのクレートが見つかったときに、どうするかを決めます。

- `deny` - 取得元の URL を含むエラーを出し、チェックを失敗させます。
- `warn`(既定) - クレートごとに警告を表示しますが、チェックは失敗させません。
- `allow` - クレートごとに注記を表示しますが、チェックは失敗させません。

### `unknown-git` {#unknown-git}

`allow-git` リストにない git リポジトリのクレートが見つかったときに、どうするかを決めます。

- `deny` - リポジトリの URL を含むエラーを出し、チェックを失敗させます。
- `warn`(既定) - クレートごとに警告を表示しますが、チェックは失敗させません。
- `allow` - クレートごとに注記を表示しますが、チェックは失敗させません。

### `required-git-spec` {#required-git-spec}

git の取得元にどの[指定子](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html#specifying-dependencies-from-git-repositories)を必須とするかを決めます。git の取得元は、パッチを当てたコードを一時的に使うには便利な方法ですが、長期の保守には欠点があります。取得元に使う指定子によって `cargo update` を実行したときの動きが決まり、既定の場合、これは実質的にそのリポジトリへのワイルドカード依存を持つことになるからです。

この設定値を使うと、git の取得元にどの指定子を許可するかを制御でき、予想外の動きを減らせます。次の値は、具体性の低いものから高いものの順に並んでいます。具体性の低い指定子を使うと、それより具体性の高い指定子もすべて許可されます。

- `any`(既定) - すべての git 指定を許可します。指定子をまったく付けない既定の書き方も含みます。この場合、リポジトリの `master` ブランチの最新のコミットを追いかけます
- `branch` - `branch = "<branch_name>"` 指定子を許可します。
- `tag` - `tag = "<tag_name>"` 指定子を許可します。
- `rev` - `rev = "<commit_sha>"` 指定子を許可します。

### `allow-git` {#allow-git}

クレートの取得元として許可する git の URL を設定します。クレートの取得元がリストにある URL のどれにも当てはまらない場合、その扱いは `unknown-git` の設定で決まります。URL は完全に一致する必要があることに注意してください。ただし、cargo の処理に合わせるため、末尾に `.git` があれば取り除かれます。

```toml
[sources]
allow-git = [
    "https://github.com/EmbarkStudios/cargo-deny",
]
```

### `private` {#private}

`allow-git` と同じように URL を設定できます。ただし、`allow-git` が 1 つの URL に完全に一致させるためのものなのに対して、`private` の URL は、ホストが一致し、同じパスで始まる git リポジトリの URL なら _どれでも_ 許可します。このフィールドは主に、社内や非公開の git ホスト(ふつうは VPN 上)を、リポジトリを 1 つずつ指定しなくても使えるようにするためのものです。もちろん、これを使って Github 上のすべてのリポジトリを許可することもできますが、おすすめしません。😉

```toml
[sources]
private = [
    "https://super-duper-sekret",
]
```

### `allow-registry` {#allow-registry}

許可するレジストリのリストです。クレートがリストにあるレジストリのどれにも見つからない場合、その扱いは `unknown-registry` の設定で決まります。

指定しない場合、このリストには既定で [crates.io](http://crates.io) のレジストリが入ります。これは次の設定と同じです。

```toml
[sources]
allow-registry = [
    "https://github.com/rust-lang/crates.io-index",
]
```

クレートのレジストリをまったく許可しない場合は、空にしてください。

```toml
[sources]
unknown-registry = "deny"
allow-registry = []
```

### `allow-org` {#allow-org}

Rust の世界のほとんどのプロジェクトは、依存の 1 つにある不具合を直したり、フィーチャを追加したりしたいとき、おそらく私たちと同じような手順をとっていると思います。だいたい次のような手順です。

1. 変更を加えるために、クレートをフォークします
1. ローカルで作業します。たぶん、クローンしたフォークへの `path` 依存を使うように、自分のプロジェクトにパッチを当てるだけです
1. 変更をフォークにプッシュします。満足できたら、`path` 依存を `git` 依存に変えて自分のフォークを指すようにし、ほかの人や CI が同じ変更を簡単に使えるようにします
1. いずれ(できれば!)、変更を入れた PR を元のリポジトリに出します
1. できれば、変更が元のリポジトリにマージされます
1. 変更を取り込んだリリースが出るまで待ちます。その間に、`git` の取得元を元のリポジトリに向け直すこともあります
1. `git` の取得元を外し、代わりに変更を含んだクレートの新しいバージョンを指すようにします
1. 一件落着!

会社や組織で作業していると、すべてのクレートを個人の Github アカウントではなく、共有の組織アカウントにフォークすることがよくあります。しかし、git の取得元をリントしていると、フォークを作ったり消したりするたびに、そのリストを更新し続けなければならず、面倒です。すべてのフォークが同じ組織(Github の用語での organization)の下にあり、おそらくそこでフォークを作る権限があるのは信頼できる人だけで、その組織の下のリポジトリはまとめて信頼したいと考えている場合でも、そうしなければなりません。

`allow-org` オブジェクトを使うと、いくつかの VCS プロバイダーの組織やユーザーを複数指定でき、プロジェクトの git の取得元をもっと簡単に設定できます。

#### `github` {#github}

git の取得元として許可する `github.com` の組織を複数指定できます。

```toml
[sources.allow-org]
github = ["YourCoolOrgGoesHere"]
```

#### `gitlab` {#gitlab}

git の取得元として許可する `gitlab.com` の組織を複数指定できます。

```toml
[sources.allow-org]
gitlab = ["YourCoolOrgGoesHere"]
```

#### `bitbucket` {#bitbucket}

git の取得元として許可する `bitbucket.org` の組織を複数指定できます。

```toml
[sources.allow-org]
bitbucket = ["YourCoolOrgGoesHere"]
```

### `unused-allowed-source` {#unused-allowed-source}

`allow` リストにある取得元のどれかが依存グラフの中で見つからなかったときに、どうするかを決めます。

- `warn`(既定) - `sources.allow` にあるが、どのクレートでも使われていない取得元ごとに警告を出します。
- `allow` - `sources.allow` リストにある使われていない取得元は無視されます。
- `deny` - `sources.allow` リストに使われていない取得元があるとエラーになり、取得元のチェックは失敗します。

### `unused-allowed-org` {#unused-allowed-org}

`allow-org` リストにある組織のどれかが依存グラフの中で見つからなかったときに、どうするかを決めます。

- `warn`(既定) - `sources.allow-org` にあるが、どのクレートでも使われていない組織ごとに警告を出します。
- `allow` - `sources.allow-org` リストにある使われていない組織は無視されます。
- `deny` - `sources.allow-org` リストに使われていない組織があるとエラーになり、取得元のチェックは失敗します。
{% endraw %}
