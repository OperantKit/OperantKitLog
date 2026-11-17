# OperantKitLog (OKL)

:gb: [English README](README.md)

OperantKitLog (OKL) は、[OperantKit](https://github.com/OperantKit) がオペラント条件づけやレスポンデント条件づけのセッションを記録するファイル形式です。反応・強化子の提示・刺激の始まりと終わり・状態の変化・フェイズのすべてを、1 つの出来事につき 1 行で記録します。このリポジトリは、OperantKit のセッションログの正典の wire format 仕様です。これらのログを読む人・書く人・変換する人、R や MATLAB などの言語で読み取りプログラムを書く人のためのものです。

- **プレーンテキストです。** 短いヘッダの後に、1 つの出来事につきタブ区切りの 1 行が続きます。どのテキストエディタでも開け、`grep`・`awk`・表計算ソフトへの読み込みがそのまま使えます。
- **ファイルが自分自身を説明します。** ヘッダが出来事の種類ごとに各列の型を宣言するため、外部のスキーマは要りません。
- **確かめられます。** 読み取りプログラムが受け付けるべきファイルと拒否すべきファイルを、例のファイルで定めています。

このリポジトリには**仕様だけ**があり、実行できるコードは含みません。AI コーディングエージェント向けの案内は [AGENTS.md](AGENTS.md) にあります。

## インストール

インストールするものはありません。Python から OKL ファイルを読み書きするには [session-recorder](https://github.com/OperantKit/session-recorder) を使ってください。インストールの手順はその README にあります。

## 例

完全な OKL v1 ファイル [`spec/conformance/valid/01-minimal.txt`](spec/conformance/valid/01-minimal.txt) です。

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

`#` で始まる行がヘッダです。1 行目が形式とバージョンを示し、セッションの項目と、出来事の種類ごとに型付きの列を宣言する codebook が続きます。`?` は省略可能な列を示し、空のときは `-` と書きます。ここでは `operandum` (オペランダム：反応が行われたレバーやキー) が 3 回目の反応にだけ記録されています。`# ---` の後は 1 行が 1 つの出来事で、秒単位の時刻・出来事の種類・宣言された列がタブ区切りで並びます。

session-recorder で読む場合:

```python
from session_recorder import read_log

log = read_log("spec/conformance/valid/01-minimal.txt")
print(log.meta.session_name, [(e.timestamp, e.operandum) for e in log.events])
# minimal [(0.0, None), (1.5, None), (3.0, 0)]
```

標準のコマンドで読む場合:

```bash
grep -v '^#' spec/conformance/valid/01-minimal.txt | awk -F'\t' '$2 == "response"' | wc -l   # 3
```

## 中身

| ファイル / ディレクトリ | 役割 |
|---|---|
| [`spec/wire-format.md`](spec/wire-format.md) | 概要：ファイルの構造、magic 行、ヘッダ、本体、TSV のエスケープ、安定性の約束 |
| [`spec/header.md`](spec/header.md) | ヘッダの文法：必須 / 任意の項目、セクション、終端の行 |
| [`spec/codebook.md`](spec/codebook.md) | 正典の 9 種類の出来事とその項目、codebook の宣言の書き方 |
| [`spec/grammar.ebnf`](spec/grammar.ebnf) | EBNF による形式文法 |
| [`spec/conformance/valid/`](spec/conformance/valid/) | 仕様に適合する読み取りプログラムが必ず受け付ける 6 つの例のファイル |
| [`spec/conformance/invalid/`](spec/conformance/invalid/) | 仕様に適合する読み取りプログラムが必ず拒否する 5 つの例のファイル。各ファイルの 2 行目に、破っている規則が書かれています |

OKL v1 は、セッションのログを作るプログラム (記録プログラム、ハードウェアとの橋渡し) と、ログを使うプログラム (分析、ダッシュボード) の間の約束です。仕様をどの実装からも切り離しておくことで、形式が特定の言語に依存しなくなります。

実装:

- Python の書き込み / 読み取り: [session-recorder](https://github.com/OperantKit/session-recorder)。そのテストが `spec/conformance/` のファイルをすべて実行します。
- Python の利用側: [session-analyzer](https://github.com/OperantKit/session-analyzer) (session-recorder のパーサを使います)。

### 安定性の約束

OKL v1 の wire format は安定しています。magic 行・ヘッダの文法・正典の codebook を壊す変更には、新しい形式のバージョン (`# OKL v2`) と、両方のバージョンを読める移行期間が必要です。

正典の codebook に**新しい種類の出来事**を足すことは、壊す変更ではありません。既存の種類に**省略可能な項目**を足すことも、壊す変更ではありません。どの利用側も、決め打ちのスキーマではなく、ファイル自身のヘッダにある codebook を読むからです。

### 適合性

実装は、[`spec/conformance/valid/`](spec/conformance/valid/) のすべてのファイルを受け付け、[`spec/conformance/invalid/`](spec/conformance/invalid/) のすべてのファイルを拒否しなければなりません。例のファイルの足し方は [`spec/conformance/README.md`](spec/conformance/README.md) にあります。

## 制限

- 実装は Python ([session-recorder](https://github.com/OperantKit/session-recorder)) だけです。他の言語の読み取りプログラムは、この仕様に沿って書く必要があります。
- 仕様が定めるファイルの拡張子は `.txt` です。magic 行 `# OKL v1` があれば、読み取りプログラムは他の拡張子を受け付けてもかまいません。

## ライセンス

MIT です。[LICENSE](LICENSE) を参照してください。
