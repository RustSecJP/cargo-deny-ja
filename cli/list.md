{% raw %}
# `list` コマンド {#the-list-command}

[cargo-license](https://github.com/onur/cargo-license) と同じように、`list` は各クレートのライセンス情報を表示します。

## オプション {#options}

### `-f, --format` {#-f---format}

出力の形式です

- `human`(既定) - 各クレートまたは各ライセンスを 1 行ずつ表示する、簡単な形式です
- `json` - JSON
- `tsv` - タブ区切りの値です。`crate` のレイアウトはこの形式には適用されないことに注意してください

### [`--color`](../cli/common.html#--color) {#--color}

色:

- SPDX 識別子 - ![blue](https://placehold.it/15/5dade2/000000?text=+)
- ライセンスが 1 つのクレート - ![white](https://placehold.it/15/717d7e/000000?text=+)
- ライセンスが 2 つ以上のクレート - ![yellow](https://placehold.it/15/f1c40f/000000?text=+)
- ライセンスが 0 個のクレート - ![yellow](https://placehold.it/15/e74c3c/000000?text=+)

### `-l, --layout` {#-l---layout}

出力のレイアウトです。`tsv` 形式には適用されません。

- `license`(既定) - 各ライセンスがキーになり、値はそのライセンスを使うすべてのクレートです
- `crate` - 各クレートがキーになり、値はそのクレートが使うライセンスのリストです。

### `-t, --threshold` {#-t---threshold}

ライセンス文のファイルにライセンス識別子を割り当てるのに必要な確信度のしきい値です。詳しくは [ライセンスの設定](../checks/licenses/cfg.html#the-confidence-threshold-field-optional) を見てください。

- `layout = license, format = human`(既定)

![Imgur](https://i.imgur.com/Iejfc7h.png)

- `layout = crate, format = human`

![Imgur](https://i.imgur.com/zZdcFXI.png)

- `layout = license, format = json`

![Imgur](https://i.imgur.com/wC2R0ym.png)

- `layout = license, format = tsv`

![Imgur](https://i.imgur.com/14l8a5K.png)
{% endraw %}
