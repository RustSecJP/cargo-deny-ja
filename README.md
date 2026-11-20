# cargo-deny の本（日本語訳）

[![原文](https://img.shields.io/badge/原文-EmbarkStudios%2Fcargo--deny-blue)](https://github.com/EmbarkStudios/cargo-deny)
[![日本語訳](https://img.shields.io/badge/日本語訳-非公式-orange)](https://rustsecjp.github.io/cargo-deny-ja/)

> [!NOTE]
> これは [EmbarkStudios/cargo-deny](https://github.com/EmbarkStudios/cargo-deny) の公式ドキュメント（The cargo-deny Book）の**非公式な日本語訳**です。Embark Studios による確認や公認を受けたものではありません。このリポジトリにツール本体のコードはありません。
> This is an unofficial Japanese translation of the cargo-deny book. It does not contain the tool itself.

`cargo-deny` は、プロジェクトの依存グラフを調べる cargo のプラグインです。既知の脆弱性の助言（RustSec）、使ってはいけないクレート、ライセンス、クレートの取得元を確かめます。

**読むのはこちら: https://rustsecjp.github.io/cargo-deny-ja/** （原文: [The cargo-deny Book](https://embarkstudios.github.io/cargo-deny/)）

## 訳した版

| | |
|---|---|
| 原文の版 | 2026-11-19 時点（コミット [`6d51289`](https://github.com/EmbarkStudios/cargo-deny/commit/6d512893fcfea934e8e69c5df44132a4970c42d0)） |
| 翻訳日 | 2026-11-21 |

原文がこの版より後に更新されている場合、訳は古い内容のままです。

## ライセンス

原文と同じく、次のどちらかを選んで利用できます。

* Apache License, Version 2.0（[LICENSE-APACHE](LICENSE-APACHE.txt)）
* MIT license（[LICENSE-MIT](LICENSE-MIT.txt)）

誤訳の指摘や訳語の提案は、このリポジトリの Issue か Pull Request でお願いします。出典の詳細は [NOTICE.md](NOTICE.md) にあります。

RustSecJP は非公式のコミュニティです。Embark Studios、RustSec、Rust Project、Rust Foundation とは提携していません。
