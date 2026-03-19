# コーディング規約（総論）

## 1. はじめに

このドキュメントは **RikiMasuda 配下の全リポジトリに共通適用するコーディング規約の総論** です。  
個別リポジトリの README や言語固有のガイドラインはこの規約を上書き・補完できますが、本規約に反する場合は理由をコメントに明記してください。

対象言語は主に **Python** と **MATLAB** で、用途は研究目的のシミュレーションプログラムです。  
本規約は「読みやすさ・再現性・保守性・レビュー容易性」を高め、数か月後の自分やチームが迷わず読めるコードを目指します。

---

## 2. 基本原則

### 2-1. 明確さ優先（Clear over Clever）

- 巧妙な一行より、平凡でも読める三行を選ぶ。
- コードは書いた回数より読まれる回数の方が多い。

| ✅ 良い例 | ❌ 悪い例 |
|---|---|
| `result = [x * 2 for x in values if x > 0]` | `r=[x*2 for x in v if x>0]` |

### 2-2. 再現性

研究コードにおける再現性は最重要事項の一つです。

- **乱数シードを必ず固定**し、コード中またはコンフィグに明示する。
- **実行環境**（OS、Python/MATLAB バージョン、主要パッケージバージョン）を README または requirements ファイルに記録する。
- **入力データのバージョン**（ファイルハッシュ、取得日、URL）を記録する。
- 結果保存時に **git commit hash** を一緒に記録する（後述）。

```python
# ✅ 良い例
import numpy as np
RANDOM_SEED = 42
rng = np.random.default_rng(RANDOM_SEED)
```

```python
# ❌ 悪い例（シード未設定）
import numpy as np
data = np.random.randn(100)
```

### 2-3. 追跡可能性（Traceability）

- パラメータ・実験条件・結果を紐付けて保存する（詳細は § 5）。
- マジックナンバーを使わず、名前付き定数または設定ファイルで管理する。

```python
# ✅ 良い例
TIME_STEP_S = 0.01        # 積分ステップ [s]
SIMULATION_DURATION_S = 10.0
```

```python
# ❌ 悪い例
for i in range(1000):     # 1000 の根拠が不明
    ...
```

### 2-4. 小さな関数・単一責任・純粋関数

- 1 関数 = 1 つのことをする（Single Responsibility）。
- 副作用（ファイル書き込み、グローバル状態変更）を持つ処理と計算処理を混在させない。
- 可能な範囲で純粋関数（入力 → 出力が決まる）を使うとテストが容易になる。

---

## 3. 命名規則（総論）

### 3-1. 基本方針

| 対象 | Python | MATLAB |
|---|---|---|
| 変数・関数 | `snake_case` | `camelCase` または `snake_case`（プロジェクト内統一） |
| クラス | `PascalCase` | `PascalCase` |
| 定数 | `UPPER_SNAKE_CASE` | `UPPER_SNAKE_CASE` |
| ファイル | `snake_case.py` | `snake_case.m` または `PascalCase.m`（クラス） |
| ディレクトリ | `snake_case/` | `snake_case/` |

### 3-2. 物理量・単位を名前に含める

単位を名前に埋め込み、次元エラーを防ぐ。

```python
# ✅ 良い例
dt_s = 0.01          # 時間刻み [s]
mass_kg = 1.5        # 質量 [kg]
velocity_mps = 3.0   # 速度 [m/s]
angle_rad = 0.5      # 角度 [rad]
```

```python
# ❌ 悪い例（単位不明）
dt = 0.01
m = 1.5
v = 3.0
```

推奨サフィックス例：`_s`（秒）、`_ms`（ミリ秒）、`_m`（メートル）、`_kg`（キログラム）、`_mps`（m/s）、`_rad`（ラジアン）、`_deg`（度）、`_n`（個数・カウント）

### 3-3. 曖昧な短縮を避ける

- `tmp`, `data2`, `val`, `foo`, `buf` などは使わない。
- **例外**：短いループ添字（`i`, `j`, `k`）や慣習的な数学記号（`x`, `y`, `t` など）は文脈が明確なら許容する。その場合はコメントで意味を明示する。

```python
# ✅ 良い例
filtered_positions = [p for p in positions if p > threshold_m]

# 許容例（コメントで意味を示す）
for i, t in enumerate(time_array_s):  # i: インデックス, t: 時刻 [s]
    ...
```

### 3-4. 真偽値の命名

真偽値は `is_` / `has_` / `can_` / `should_` などの接頭辞を使う。

```python
is_converged = True
has_boundary_condition = False
```

---

## 4. コメント規約

### 4-1. 「なぜ」「前提」「注意点」を書く

コードを読めば分かる「何を」ではなく、背景・理由・制約を書く。

```python
# ✅ 良い例
# Euler 法では dt が大きいと発散するため 0.01 s 以下に設定（Ref: § 3.2 の安定条件）
dt_s = 0.01

# ❌ 悪い例（コードの繰り返し）
dt_s = 0.01  # dt_s に 0.01 を代入
```

### 4-2. 数式・出典・論文参照

数式や理論を使う場合は出典を必ず示す。

```python
# 運動方程式（Newton の第2法則）
# F = m * a
# 参考: Smith et al. (2020), DOI: 10.xxxx/xxxxx, Eq. (3.4)
acceleration_mps2 = force_n / mass_kg
```

- DOI は `DOI: 10.xxxx/xxxxx` 形式で記載する。
- 書籍は著者・タイトル・章・式番号を記載する。
- URL は変更されうるので DOI や arXiv ID を優先する。

### 4-3. TODO / FIXME の使い分け

| タグ | 用途 |
|---|---|
| `TODO:` | 将来やるべき改善・機能追加 |
| `FIXME:` | 既知のバグ・不正確な実装（動くが問題あり） |
| `HACK:` | 一時的な回避策（後で直す前提） |
| `NOTE:` | 注意事項・前提条件の共有 |

```python
# TODO: 高精度化のために RK4 に切り替える
# FIXME: 負の質量が入力されたとき例外を出さず計算が続く
```

---

## 5. ドキュメント規約（研究コード向け）

### 5-1. README の最低限記載事項

各リポジトリの README には以下を含める。

- [ ] **目的**：何を解くシミュレーションか（1〜3 行）
- [ ] **実行手順**：コマンド例（コピペで動くレベル）
- [ ] **依存関係**：言語バージョン、主要パッケージとバージョン
- [ ] **再現手順**：論文の図を再現するためのコマンドや手順
- [ ] **データ入手**：入力データの場所・取得方法（URLまたはパス）
- [ ] **出力**：生成されるファイルと形式の説明
- [ ] **主要パラメータ**：変更頻度の高いパラメータの一覧と意味

### 5-2. 実験・シミュレーション結果の保存

結果ディレクトリやログファイルに以下を必ず含める。

```
results/
  20240318_152300/
    params.json          # 実験条件・パラメータ
    git_hash.txt         # git rev-parse HEAD の出力
    output_data.csv      # 計算結果
    figures/
```

`params.json` の例：

```json
{
  "random_seed": 42,
  "dt_s": 0.01,
  "simulation_duration_s": 10.0,
  "git_commit": "a1b2c3d",
  "timestamp": "2024-03-18T15:23:00+09:00"
}
```

### 5-3. 乱数・初期条件・設定ファイル（Config）の運用

- 設定値はコード中にハードコードせず、`config.yaml` / `config.json` / `params.py` などに切り出す。
- 設定ファイルのサンプル（`config.example.yaml`）をリポジトリに含め、実際の設定ファイルは `.gitignore` してよい（秘密情報がある場合）。
- 乱数シードは設定ファイルで管理し、実験ごとに記録する。

---

## 6. エラーハンドリング / 入力検証

### 6-1. `assert` の使い所

`assert` は**開発中の前提条件チェック**に限定する。プロダクションコードや入力検証には使わない（`-O` フラグで無効化されるため）。

```python
# ✅ 良い例：内部前提条件（デバッグ用）
assert len(time_array_s) == len(state_array), "時系列の長さが一致しない"

# ❌ 悪い例：ユーザー入力検証に assert を使う
assert dt_s > 0  # → ValueError を使う
```

### 6-2. 例外の方針

- 想定外の入力は `ValueError` / `TypeError` などの具体的な例外を送出する。
- エラーメッセージには**何が問題で、何を期待していたか**を含める。

```python
# ✅ 良い例
if dt_s <= 0:
    raise ValueError(f"dt_s は正の値が必要です。受け取った値: {dt_s}")
```

### 6-3. 前提条件チェック（Guard Clause）

関数の先頭で前提条件を検証し、早期リターンまたは例外で処理を打ち切る。

```python
def simulate(dt_s: float, duration_s: float) -> np.ndarray:
    if dt_s <= 0:
        raise ValueError(f"dt_s > 0 が必要: {dt_s}")
    if duration_s <= dt_s:
        raise ValueError("duration_s > dt_s が必要")
    # ... 本処理
```

---

## 7. テストと検証

### 7-1. 単体テストの考え方

数値計算のテストでは絶対一致より許容誤差（tolerance）を使う。

```python
# ✅ 良い例（許容誤差付き比較）
import numpy as np
np.testing.assert_allclose(result, expected, rtol=1e-5, atol=1e-8)

# ❌ 悪い例（浮動小数点の厳密一致）
assert result == expected
```

- **固定シード**でランダム要素を排除してからテストする。
- **回帰テスト**：既知の正解出力を保存し、リファクタ後も同じ結果が出ることを確認する。

### 7-2. 再現性テスト

- 同一設定・同一シードで 2 回実行し、結果が一致することを確認するテストを推奨する。
- CI/CD 環境でも実行できるよう、重いシミュレーションは小さいパラメータで簡略化したテスト用設定を用意する。

```python
def test_reproducibility():
    result1 = run_simulation(seed=42, duration_s=1.0)
    result2 = run_simulation(seed=42, duration_s=1.0)
    np.testing.assert_array_equal(result1, result2)
```

---

## 8. Python 向け推奨

### 8-1. スタイルガイド

- **PEP 8**（スタイル）: https://peps.python.org/pep-0008/
- **PEP 257**（docstring 規約）: https://peps.python.org/pep-0257/
- **Google スタイル docstring** を推奨: https://google.github.io/styleguide/pyguide.html#38-comments-and-docstrings
- **NumPy docstring** スタイル（数値計算向け）: https://numpydoc.readthedocs.io/en/latest/format.html

### 8-2. Docstring の例（Google スタイル）

```python
def integrate_euler(state: np.ndarray, dt_s: float, derivative_fn) -> np.ndarray:
    """Euler 法で状態を1ステップ積分する。

    Args:
        state: 現在の状態ベクトル。
        dt_s: 積分ステップ幅 [s]。正の値が必要。
        derivative_fn: 状態の時間微分を返す関数。

    Returns:
        次の時刻の状態ベクトル。

    Raises:
        ValueError: dt_s が 0 以下の場合。
    """
    if dt_s <= 0:
        raise ValueError(f"dt_s > 0 が必要: {dt_s}")
    return state + dt_s * derivative_fn(state)
```

### 8-3. 型ヒント

可能な範囲で型ヒントを付ける（必須ではないが強く推奨）。

```python
from typing import Callable
import numpy as np

def run_simulation(duration_s: float, dt_s: float, seed: int = 42) -> np.ndarray:
    ...
```

### 8-4. フォーマッタ / リンタ（導入推奨）

必須ではないが、プロジェクト単位での導入を推奨する。

- **ruff**（高速リンタ＋フォーマッタ）: https://docs.astral.sh/ruff/
- **black**（フォーマッタ）: https://black.readthedocs.io/
- 設定例（`pyproject.toml`）:
  ```toml
  [tool.ruff]
  line-length = 99
  ```

---

## 9. MATLAB 向け推奨

### 9-1. スタイルガイド・参考リンク

- MathWorks 公式ベストプラクティス: https://mathworks.com/products/matlab/matlab-and-best-practices.html
- MATLAB Style Guidelines 2.0（Richard Johnson）: https://www.mathworks.com/matlabcentral/fileexchange/46056

### 9-2. スクリプトと関数の使い分け

| 用途 | 推奨 |
|---|---|
| 再利用する計算・処理 | 関数ファイル（`.m` ファイル、1 ファイル 1 関数）|
| 実験のエントリポイント・プロット | スクリプト（`run_simulation.m` など）|
| 共通ユーティリティ | `utils/` ディレクトリにまとめ `addpath` で読み込む |

### 9-3. パス管理

- `addpath` はエントリポイントのスクリプト冒頭でまとめて実行する。
- `addpath(genpath(...))` の乱用は避け、必要なディレクトリのみ追加する。

### 9-4. 命名・ベクトル化・可読性

```matlab
% ✅ 良い例：ベクトル化
force_n = mass_kg .* acceleration_mps2;

% ❌ 悪い例：不要なループ
for i = 1:length(mass_kg)
    force_n(i) = mass_kg(i) * acceleration_mps2(i);
end
```

- **過度なワンライナーを避ける**：1 行に詰め込むより読みやすさを優先する。
- 物理量のサフィックス（`_s`, `_kg`, `_mps` など）は Python と同様に適用する。

---

## 10. 例外規定（プロトタイプ・探索的実装）

探索的実装やアイデア検証フェーズでは以下を許容する。

- 命名の簡略化（ただし後で整える前提でコメントを残す）
- テストなし
- ハードコードされたパラメータ

ただし、**プロトタイプをそのまま本実装にしない**。  
探索フェーズが終わったら以下を実施する。

- [ ] 命名を本規約に合わせてリネームする
- [ ] マジックナンバーを定数または設定ファイルに切り出す
- [ ] 最低限の再現性チェック（シード固定・結果保存）を追加する
- [ ] README または関数 docstring に実験条件を記録する

---

## 11. PR 前セルフチェックリスト

PR を出す前に以下を確認する。

### コード品質
- [ ] 曖昧な変数名・関数名を使っていない
- [ ] 物理量の変数名に単位サフィックスを付けている
- [ ] マジックナンバーを使っていない（定数または設定ファイルに切り出している）
- [ ] コメントは「なぜ」「前提」「注意点」を説明している
- [ ] 数式・論文参照が必要な箇所に出典を記載している

### 再現性・追跡可能性
- [ ] 乱数シードを固定している（または意図的に未固定の場合はコメントに明記）
- [ ] 実験設定をハードコードせず設定ファイルまたは定数で管理している
- [ ] 結果保存時に git commit hash とパラメータを記録する仕組みがある

### ドキュメント
- [ ] 新しい関数に docstring を書いている
- [ ] README に変更を反映している（必要な場合）
- [ ] 破壊的変更がある場合は PR 本文に明記している

### テスト
- [ ] 追加・変更した関数に対するテストを書いている（または既存テストが通る）
- [ ] 数値比較は許容誤差付きで行っている
