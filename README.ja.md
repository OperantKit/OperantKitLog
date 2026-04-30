# OperantKitLog (OKL)

:gb: [English README](README.md)

OperantKit セッションログの正典 wire format **仕様** ディレクトリ。

ここには **仕様のみ** が存在し、実行可能コードは含まない。Python の
writer/reader 実装は [`session-recorder`](../session-recorder) 配下に
あり、将来の Rust / TS / JS 実装はここの仕様に対して書かれることを
想定する。

## 構成

| ファイル / ディレクトリ | 役割 |
|---|---|
| [`spec/wire-format.md`](spec/wire-format.md) | 概要：ファイル構造、magic 行、ヘッダ、本文、TSV エスケープ、安定性契約 |
| [`spec/header.md`](spec/header.md) | ヘッダ文法：必須 / 任意フィールド、セクション、終端子 |
| [`spec/codebook.md`](spec/codebook.md) | 正典イベント型とそのフィールド；codebook 宣言構文 |
| [`spec/grammar.ebnf`](spec/grammar.ebnf) | 形式 EBNF 文法 |
| [`spec/conformance/valid/`](spec/conformance/valid/) | conformant な reader が受理すべき golden fixture |
| [`spec/conformance/invalid/`](spec/conformance/invalid/) | conformant な reader が拒否すべき golden fixture |

## なぜ仕様のみか

OKL v1 は producer（recorder、ハードウェアブリッジ）と consumer
（analyzer、dashboard）の間のデータ層契約。仕様を特定の実装から
切り離してここに集約することで、スキーマドリフトを防ぎ、フォーマット
を言語非依存に保つ。

実装：

- Python writer/reader: [`session-recorder`](../session-recorder)
- Python consumer: [`session-analyzer`](../../analysis/session-analyzer)
  （session-recorder のパーサを使用）

## 安定性契約

OKL v1 wire format は安定。magic 行・ヘッダ文法・正典 codebook の
破壊的変更には新フォーマットバージョン（`# OKL v2`）と両バージョンが
パース可能な移行期間が必要。

正典 codebook への **新イベント型の追加** は非破壊的変更。既存
イベント型への **新オプションフィールドの追加** も非破壊的（各
consumer はハードコードされたスキーマではなくファイル自身のヘッダ
から codebook を読むため）。

## Conformance

実装は [`spec/conformance/valid/`](spec/conformance/valid/) 配下の
全 fixture を受理し、[`spec/conformance/invalid/`](spec/conformance/invalid/)
配下の全 fixture を拒否しなければならない。Python 実装は
`session-recorder/tests/test_okl_conformance.py` を介してこれらの
fixture を通している。
