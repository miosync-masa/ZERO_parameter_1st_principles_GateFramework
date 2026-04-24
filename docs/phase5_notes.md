# Phase 5 作業メモ

**日付**: 2026-04-24
**担当**: ClaudeCode (メカタマキ)
**結果**: ✅ **98 passed + 5 xfailed (87.5 秒)**

## テスト結果

### Phase 1-4 継続 (74 pass + 5 xfail)
- Phase 1: 15/15
- Phase 2:  9/9
- Phase 3: 27/27
- Phase 4: 23 pass + 5 xfail (期待通り)

### Phase 5 新規 CLI テスト (24/24 pass)
- Basic invocation: 3/3 (`--version`, `--help`, no-subcommand)
- list-proteins:    3/3 (table/json/compact)
- predict:          9/9 (compact/json/table/split-format/kras/tdp43/invalid/nonexistent/missing-arg)
- predict-batch:    4/4 (csv/json/stdin/csv-output)
- explain:          4/4 (default/idr/channels-only/verbose)
- API parity:       1/1 (CLI output == direct Predictor call)

## 実装した CLI サブコマンド

| Command | Purpose | 主要 flags |
|---------|---------|-----------|
| `predict`        | 単一変異予測 | `--protein` / `--annotation` / `--mutation` / `--format` / `--mode` |
| `predict-batch`  | 複数変異バッチ (CSV/JSON/TSV/stdin) | `--input` / `--output` / `--format` |
| `explain`        | 詳細診断 (channel states + residue context) | `--verbose` / `--channels-only` |
| `list-proteins`  | 同梱タンパク一覧 | `--format` |

## 実装した出力フォーマット

- `json`    - 機械可読 (pipeline-friendly)
- `table`   - ASCII box drawing (人間可読)
- `compact` - 1行要約
- `csv`     - predict-batch 専用 (flat row)

## pip install 後の動作確認

```
$ pip install -e .
Successfully installed pathogenicity-gates-0.5.0.dev0

$ which pathogenicity-gates
/Users/iizumimamichi/.pyenv/shims/pathogenicity-gates

$ pathogenicity-gates --version
pathogenicity-gates 0.5.0-dev

$ pathogenicity-gates list-proteins
Bundled proteins in pathogenicity-gates:
  brca1    P38398
           Breast cancer type 1 susceptibility protein (tumor suppressor)
  kras     P01116
           GTPase KRas (proto-oncogene)
  p53      P04637
           Cellular tumor antigen p53 (tumor suppressor)
  tdp43    Q13148
           TAR DNA-binding protein 43 (ALS/FTLD associated)

$ pathogenicity-gates predict --protein p53 --mutation R175H
* R175H  (p53)  Pathogenic   n_closed=3

$ pathogenicity-gates predict --protein tdp43 --mutation G348C --format json
{
  "prediction": "Pathogenic",
  "n_closed": 1,
  "channels": { "Ch07_PTM": "O", "Ch10_SLiM": "O",
                "Ch11_IDR_Pro": "O", "Ch12_IDR_Gly": "C" },
  "regime": "idr", ...
}
```

すべて正常動作。

## 依存性: ゼロ追加

Python 標準ライブラリのみ使用:
- `argparse` (CLI parser)
- `json`, `csv`, `io`, `tempfile` (data I/O)
- `re`, `os`, `sys`, `subprocess` (utility)

pyproject.toml の dependencies に追加なし。

## 実装中に気づいたこと

### 1. ProteinAnnotation に description フィールドが未存在
Phase 2 の loader では `protein.description` を読んでいなかったため、
CLI の `list-proteins` で description が欠落。追加 (default None で後方互換):

```python
# loader.py
description: Optional[str] = None  # dataclass の default 有り field は後置
```

YAML 側の `protein.description` は Phase 2 時点で書かれているので
値はそのまま拾える。

### 2. dataclass の field 順序制約
`description: Optional[str] = None` は default 有り field なので、
default なし field の前に置くと `non-default argument follows default argument` エラー。
`benign_pro_poly` の default 有り field の後に配置して解決。

### 3. CLI test 時間
subprocess 起動 + Predictor setup が 1 テストあたり 2-3 秒 × 24 テスト =
約 60 秒 追加。Phase 4 までの 36 秒から 87 秒へ増加。許容範囲。

## 論文 Revision への申し送り

### Methods セクション追記案
> The package is available via `pip install pathogenicity-gates` (PyPI) and
> provides a command-line interface (`pathogenicity-gates`) with four
> subcommands (`predict`, `predict-batch`, `explain`, `list-proteins`).
> A non-expert user can evaluate any variant with a single command, e.g.
> `pathogenicity-gates predict --protein p53 --mutation R175H`. This
> directly addresses Reviewer #1's concern about framework accessibility.

### Data Availability / Code Availability
- GitHub: (Phase 6 で作成)
- PyPI: `pathogenicity-gates` (Phase 6 で公開)
- Zenodo DOI: (Phase 6 で取得)
- CLI docs: `docs/cli_usage.md`

## 次ステップ (Phase 6: PyPI 公開) への申し送り

- [ ] GitHub repo セットアップ (本番用 README)
- [ ] GitHub Actions CI (pytest on push)
- [ ] Zenodo DOI 取得
- [ ] PyPI パッケージ公開 (`twine upload`)
- [ ] 論文本文に CLI 紹介追加

## Phase 5 完了判定 (DoD)

- [x] `pip install -e .` で `pathogenicity-gates` コマンドが PATH に入る
- [x] `pathogenicity-gates --version` が `0.5.0-dev` 表示
- [x] 4 サブコマンド (predict, predict-batch, explain, list-proteins) 全動作
- [x] 3 出力フォーマット (json/table/compact) + csv 全動作
- [x] Phase 1-4 の 79 テスト継続 pass (74 + 5 xfail)
- [x] Phase 5 新規 24 CLI テスト全 pass
- [x] `docs/cli_usage.md` 作成
- [x] `README.md` に CLI セクション追加
- [x] `phase5_notes.md` 作成

**✨ Phase 5 完了。環 に「Phase 5 完了」を報告します。**
研究者が `pip install pathogenicity-gates` 一発で使える状態が完成。
Reviewer #1 の使用容易性指摘への最終的な回答を実装レベルで達成。
