# Medical Edge Scraper & OCR Pipeline

![Python](https://img.shields.io/badge/Python-3.8+-blue?logo=python&logoColor=white)
![Platform](https://img.shields.io/badge/Platform-Windows%20%7C%20WSL-lightgrey)
![Tools](https://img.shields.io/badge/Tools-ADB%20%7C%20PaddleOCR-green)

---

## 概要 (Overview)

医学部の学習・マッチング活動を効率化するための **「エッジ攻略」** ツールセットです。

Android エミュレータ上のアプリ画面（Hokuto など）を **自動でスクロール＆キャプチャ** し、  
**AI-OCR** を用いてテキストデータ化・PDF 化を一括で行います。

取得したデータは **LLM（ChatGPT / Claude）** に投入することで、  
**数年分の口コミを数秒で分析・要約** することが可能です。

---

## 機能 (Features)

### 🤖 画像キャプチャ — Scraper (`hokuto_scrape.py`)

| 機能 | 説明 |
|------|------|
| **ADB 自動スクロール** | BlueStacks（Android エミュレータ）と ADB 接続し、自動スワイプ＆スクリーンショット撮影 |
| **重複検知・自動停止** | 画像ファイルサイズの比較により、リストの最後まで到達すると自動停止 |
| **中央領域クロップ** | 上下 10% をカットし、コンテンツ部分（中央 80%）のみを保存 |
| **BAN 対策** | 人間らしいランダムな待機時間（1.5〜3.0 秒）を実装済み |
| **最大 300 ページ** | 自動停止しない場合でも 300 ページで安全に終了 |

### 🌳 XML テキスト抽出 — Scraper (`hokuto_scrape_xml_dump.py`)

| 機能 | 説明 |
|------|------|
| **uiautomator dump** | `uiautomator dump` で画面の UI 階層を XML として取得し、`text` 属性を抽出 |
| **OCR 不要の高精度テキスト** | UI ツリーから直接テキストを読むため、OCR 誤字がゼロ |
| **重複除去・順序保持** | 既出テキストをスキップしつつ、出現順を維持して保存 |
| **自動停止** | 3 回連続で新規テキストが 0 件なら自動終了（最大 100 スクロール） |
| **リアルタイムプレビュー** | 各スクロールで新規テキストの冒頭 3 行をターミナルに表示 |

> 💡 **使い分け**: `hokuto_scrape.py`（画像）は見た目の保存・PDF 化に、`hokuto_scrape_xml_dump.py`（XML）はテキスト精度重視のデータ取得に最適です。両方を取得して `hokuto_marge_ocr_xml_reviews.py` でマージすると最高精度になります。

### 📝 OCR Pipeline (`ocr_pipeline.py`)

| 機能 | 説明 |
|------|------|
| **画像結合 → PDF** | 収集したスクリーンショットを結合して PDF 化（Goodnotes などで閲覧用） |
| **PaddleOCR（GPU 対応）** | 高精度に日本語テキストを抽出 |
| **複数形式出力** | Word (`.docx`) および Text (`.txt`) 形式で出力し、LLM 分析に即座に利用可能 |

---

## ワークフロー

```
BlueStacks (Hokuto アプリ)
  │
  ├─ hokuto_scrape.py ──────── ADB でスクロール＆キャプチャ（画像）
  │                             └─ review_page_001.png ~ _300.png
  │
  ├─ hokuto_scrape_xml_dump.py ─ ADB + uiautomator で UI テキスト抽出
  │                             └─ 病院名.txt（高精度テキスト）
  │
  ├─ ocr_pipeline.py ──────── 画像 → PDF + OCR テキスト抽出
  │                             ├─ combined.pdf   (閲覧用)
  │                             ├─ output.docx    (Word)
  │                             └─ output.txt     (テキスト)
  │
  └─ ChatGPT / Claude ──────── テキストを投入して分析・要約
```

---

## 必要要件 (Requirements)

- **Python 3.8+**
- **Android Emulator** — [BlueStacks 5](https://www.bluestacks.com/) 推奨
- **ADB** — BlueStacks 同梱のものを使用（別途インストール不要）
- **Pillow** — スクリーンショットのクロップに使用

---

## 🚀 セットアップ

### 1. リポジトリをクローン

```bash
git clone https://github.com/torikabuto-fkg/medical-edge-scraper.git
cd medical-edge-scraper
```

### 2. ライブラリのインストール

**GPU を使用する場合（推奨）:**

```bash
python -m pip install paddlepaddle-gpu==2.6.1
pip install -r requirements.txt
```

**CPU のみの場合:**

```bash
python -m pip install paddlepaddle
pip install -r requirements.txt
```

### 3. BlueStacks の設定

1. BlueStacks の **設定 → 詳細設定** で **ADB デバッグ** を有効にする
2. ADB ポート番号を確認（デフォルト: `5556`）

---

## ▶️ 使い方 (Usage)

### Step 1: スクレイピング

1. BlueStacks で対象アプリ（Hokuto など）の **口コミ画面** を開く
2. スクリプト内の設定を環境に合わせて編集:

```python
# ADB パス（WSL の場合）
ADB_PATH = "/mnt/c/Program Files/BlueStacks_nxt/HD-Adb.exe"

# ADB 接続先
ADB_HOST = "127.0.0.1"
ADB_PORT = "5556"

# 画像の保存先ディレクトリ
OUTPUT_DIR = "hokuto_reviews"
```

3. スクリプトを実行:

```bash
python hokuto_scrape.py
```

4. 3 秒のカウントダウン後、自動スクロール＆キャプチャが開始
5. リストの最後に到達すると **自動で停止**

#### 実行ログの例

```
Connecting to 127.0.0.1:5556...
[Success] Connected to BlueStacks.
----------------------------------------
スクレイピングを開始します。
Hokutoの口コミ画面を開いておいてください。
※画像は中央部分のみ保存します（上下20%カット）
3秒後に開始します...
----------------------------------------
[001] Processing...
  -> Waiting 2.3s...
[002] Processing...
  -> Waiting 1.8s...
...
[047] Processing...
  -> [Stop Check] 画面に変化がありません。
[048] Processing...
  -> [Stop Check] 画面に変化がありません。
終了: これ以上スクロールできません。
完了しました。
```

### Step 2: XML テキスト抽出（高精度・OCR 不要）

画像キャプチャと**並行して**、または**代わりに**、UI ツリーから直接テキストを抽出できます。

1. スクリプト内の設定を編集:

```python
# 出力先
OUTPUT_DIR = "./hokuto_scrapes_xml"
OUTPUT_FILE = "東大病院.txt"
```

2. 実行:

```bash
python hokuto_scrape_xml_dump.py
```

#### 実行ログの例

```
Connecting to 127.0.0.1:5556...
[Success] Connected
----------------------------------------
UI XMLダンプによるスクレイピングを開始します
Hokutoの口コミ画面を開いておいてください
3秒後に開始...
----------------------------------------

[001] UIダンプ中...
  -> 新規テキスト: 24件
  -> 累計: 24件
  --- 新規テキストプレビュー ---
    1. 東京大学 5年 男性 見学した 2024年度
    2. 指導医の先生方が非常に丁寧で、研修医の裁量も大き...
    3. 当直の回数がやや多い印象を受けました。ただし...

[002] UIダンプ中...
  -> 新規テキスト: 18件
  -> 累計: 42件
...

[035] UIダンプ中...
  -> 新規テキスト: 0件
  -> [Stop Check] 新しいテキストがありません
[036] UIダンプ中...
  -> 新規テキスト: 0件
  -> [Stop Check] 新しいテキストがありません
[037] UIダンプ中...
  -> 新規テキスト: 0件
  -> [Stop Check] 新しいテキストがありません
終了: これ以上スクロールしても新しいテキストが見つかりません

✅ 完了
抽出されたテキスト: 523件
保存先: ./hokuto_scrapes_xml/東大病院.txt
```

> 💡 **ヒント**: XML 抽出テキストは OCR テキストよりも正確です。両方を取得して `hokuto_marge_ocr_xml_reviews.py`（別リポジトリ）でマージすると、**OCR の見出し構造 + XML の高精度本文** を組み合わせた最高精度の Excel が生成できます。

### Step 3: OCR テキスト抽出

```bash
python ocr_pipeline.py
```

画像が PDF に結合され、PaddleOCR でテキスト抽出 → `.docx` / `.txt` に出力されます。

### Step 4: LLM で分析

出力された `.txt` を ChatGPT や Claude に投入して、口コミの傾向分析・要約を行います。

---

## ⚙️ カスタマイズ

### スワイプ設定（1080×1920 解像度用）

```python
SWIPE_START_X = 540    # 画面横幅の中央
SWIPE_START_Y = 1600   # 下から掴んで...
SWIPE_END_X = 540      # 真上に引き上げる
SWIPE_END_Y = 600      # 移動距離: 1000px
SWIPE_DURATION = 600    # スワイプ速度 (ms)。小さいほど速い
```

> エミュレータの画面解像度が異なる場合は、座標を調整してください。

### 重複検知の感度

連続で同じファイルサイズが検出された回数で停止判定を行います（デフォルト: **2 回連続** で停止）。

---

## 📁 出力ファイル構成

### hokuto_scrape.py（画像キャプチャ）

```
hokuto_reviews/
├── review_page_001.png   # 口コミ画面キャプチャ（中央クロップ済み）
├── review_page_002.png
├── review_page_003.png
├── ...
└── review_page_047.png
```

### hokuto_scrape_xml_dump.py（XML テキスト抽出）

```
hokuto_scrapes_xml/
└── 東大病院.txt           # UI ツリーから抽出した高精度テキスト（1行1要素）
```

### OCR Pipeline 実行後

```
output/
├── combined.pdf          # 全画像を結合した PDF
├── output.docx           # OCR テキスト (Word)
└── output.txt            # OCR テキスト (プレーンテキスト)
```

---

## 🛠️ 技術スタック

| ツール / ライブラリ | 用途 |
|-------------------|------|
| [ADB](https://developer.android.com/tools/adb) | Android エミュレータとの通信・操作 |
| [BlueStacks 5](https://www.bluestacks.com/) | Android エミュレータ |
| [uiautomator](https://developer.android.com/training/testing/other-components/ui-automator) | UI 階層の XML ダンプ・テキスト抽出 |
| [Pillow](https://python-pillow.org/) | スクリーンショットのクロップ処理 |
| [PaddleOCR](https://github.com/PaddlePaddle/PaddleOCR) | GPU 対応 日本語 OCR エンジン |

---

## 📌 注意事項

- このツールは **個人の学習・情報収集目的** で使用してください
- 対象アプリの **利用規約** を確認の上ご利用ください
- 短時間に大量のキャプチャを取得すると **アカウント制限** の可能性があります。`SWIPE_DURATION` や `wait_time` を適切に設定してください
- `ADB_PATH` は **ご自身の BlueStacks インストール先** に合わせて書き換えてください
- WSL から実行する場合、パスは `/mnt/c/...` 形式で指定してください

---

## License

MIT
