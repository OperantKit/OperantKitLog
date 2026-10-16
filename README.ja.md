# OperantKitLog (OKL)

:gb: [English README](README.md)

OperantKit セッションログの正典 wire format **仕様** ディレクトリ。

ここには **仕様のみ** が存在し、実行可能コードは含まない。Python の
writer/reader 実装は [`session-recorder`](https://github.com/OperantKit/session-recorder) 配下に
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

## 30 秒で動く例

完全な OKL v1 ファイル（[`spec/conformance/valid/01-minimal.txt`](spec/conformance/valid/01-minimal.txt)）。`#` で始まる行がヘッダで、magic 行・セッションのフィールド・各イベント種別を型つきの列で宣言するコードブックからなる（`?` は省略可能な列で、値が無いときは `-` と書く）。`# ---` の後は 1 行 1 イベントで、秒単位の時刻・イベント種別・宣言された列をタブで区切って並べる。

```text
# OKL v1
# session_name = "minimal"
# clock_type = "ManualClock"
# session_start = 0.0
# events:
#   response          : id:int operandum:int?
# ---
0.0	response	0	-
1.5	response	1	-
3.0	response	2	0
```

Python 実装でも、標準のテキストツールでも読める:

```python
from session_recorder import read_log  # pip install session-recorder

log = read_log("spec/conformance/valid/01-minimal.txt")
print(log.meta.session_name, [(e.timestamp, e.operandum) for e in log.events])
# minimal [(0.0, None), (1.5, None), (3.0, 0)]
```

```bash
grep -v '^#' spec/conformance/valid/01-minimal.txt | awk -F'\t' '$2 == "response"' | wc -l   # 3
```

## 場当たりの CSV や JSON ではなく、仕様のある形式を使う理由

- **自己記述的.** 各ファイルのコードブックがイベント種別と列の型を宣言するので、読み手は外部のスキーマを要さず、新しいイベント種別が古い読み手を壊さない。
- **型とエスケープ.** 列は `int`・`float`・`str`・`bool`（`?` で省略可）。文字列中のタブ・改行・バックスラッシュはエスケープし、値の欠損は `-` で表すので、どの行も列数が決まっている。
- **プレーンテキスト.** UTF-8 のタブ区切りなので、どのエディタでも開け、`grep`・`awk`・`column`・表計算ソフトの取り込みがそのまま使える。
- **検査できる適合性.** 正常系 5 個と異常系 5 個の fixture が、読み手が受理すべきものと拒否すべきものを定める。異常系の各 fixture は 2 行目に違反している規則を書いている。
- **形式文法.** [`spec/grammar.ebnf`](spec/grammar.ebnf) が構文を定めるので、他言語の読み手は 1 つの実装ではなく仕様に照らして書ける。

## なぜ仕様のみか

OKL v1 は producer（recorder、ハードウェアブリッジ）と consumer
（analyzer、dashboard）の間のデータ層契約。仕様を特定の実装から
切り離してここに集約することで、スキーマドリフトを防ぎ、フォーマット
を言語非依存に保つ。

実装：

- Python writer/reader: [`session-recorder`](https://github.com/OperantKit/session-recorder)
- Python consumer: [`session-analyzer`](https://github.com/OperantKit/session-analyzer)
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
