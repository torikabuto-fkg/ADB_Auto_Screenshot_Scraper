# Medical Edge Scraper & OCR Pipeline

![Python](https://img.shields.io/badge/Python-3.8+-blue?logo=python&logoColor=white)
![Platform](https://img.shields.io/badge/Platform-Windows%20%7C%20WSL-lightgrey)
![Tools](https://img.shields.io/badge/Tools-ADB%20%7C%20PaddleOCR-green)

**Android アプリ（Hokuto など）から病院口コミを自動収集し、AI-OCR でテキスト化・構造化するツールセット**

数年分の口コミを数分で収集 → LLM（ChatGPT/Claude）で一括分析可能！

---

## 🚀 クイックスタート

### 1. 環境準備

```bash
# リポジトリをクローン
git clone https://github.com/torikabuto-fkg/medical-edge-scraper.git
cd medical-edge-scraper

# ライブラリをインストール
pip install -r requirements.txt

# GPU 対応の場合（推奨）
python -m pip install paddlepaddle-gpu==2.6.1
```

### 2. BlueStacks 設定

1. [BlueStacks 5](https://www.bluestacks.com/) をインストール
2. **設定 → 詳細設定 → ADB デバッグ** を有効化
3. ADB ポート番号を確認（デフォルト: `5556`）

### 3. データ収集

**高精度テキスト抽出（推奨）：**

```bash
python hokuto_scrape_xml_dump.py
```

**画像キャプチャ（PDF 化用）：**

```bash
python hokuto_scrape.py
```

### 4. OCR 実行（画像を取得した場合）

```bash
python ocr_pipeline.py
```

### 5. データマージ（両方取得した場合）

```bash
python hokuto_marge_ocr_xml_reviews.py <OCR結果.jsonl> <XML結果.txt>
```

---

## 📁 ツール構成

| スクリプト | 機能 | 出力 |
|-----------|------|------|
| `hokuto_scrape_xml_dump.py` | UI ツリーから**高精度テキスト**を直接抽出 | `.txt` |
| `hokuto_scrape.py` | 画面を自動スクロール＆キャプチャ | `.png` |
| `ocr_pipeline.py` | 画像 → PDF + OCR テキスト抽出 | `.pdf`, `.docx`, `.txt` |
| `hokuto_marge_ocr_xml_reviews.py` | OCR と XML を類似度マッチングでマージ | `.xlsx` |

---

## 💡 推奨ワークフロー

```
① hokuto_scrape_xml_dump.py  → 高精度テキスト取得
         ↓
② hokuto_scrape.py          → 画像も取得（PDF 化用）
         ↓
③ ocr_pipeline.py           → OCR 実行
         ↓
④ hokuto_marge_ocr_xml_reviews.py → 最高精度の構造化 Excel を生成
         ↓
⑤ ChatGPT / Claude          → テキストを投入して分析
```

---

## ⚙️ 設定のカスタマイズ

各スクリプト冒頭の設定エリアを編集：

```python
# ADB 接続（WSL の場合）
ADB_PATH = "/mnt/c/Program Files/BlueStacks_nxt/HD-Adb.exe"
ADB_HOST = "127.0.0.1"
ADB_PORT = "5556"

# 出力先
OUTPUT_DIR = "hokuto_reviews"
```

---

## 📊 出力例

### XML テキスト抽出（hokuto_scrape_xml_dump.py）

```
[037] UIダンプ中...
  -> 新規テキスト: 0件
  -> [Stop Check] 新しいテキストがありません
終了: これ以上スクロールしても新しいテキストが見つかりません

✅ 完了
抽出されたテキスト: 523件
保存先: ./hokuto_scrapes_xml/東大病院.txt
```

### マージ実行（hokuto_marge_ocr_xml_reviews.py）

```
🔗 レビューをマッチング中...
   OCRレビュー: 32件
   XMLレビュー: 30件
   ✓ マッチ: 2024年度 5年 (類似度: 87.32%)

📊 統計:
   総レビュー数: 32
   OCR structure + XML text: 27件
   OCR only: 5件

✨ 完了!
```

---

## 🐳 Docker で実行（オプション）

<details>
<summary>GPU 対応 OCR を Docker で実行する場合（クリックで詳細表示）</summary>

### 前提条件

- Docker Desktop (Windows)
- WSL2 backend 有効化
- NVIDIA GPU + ドライバ

### 実行手順

```bash
# イメージをビルド
docker build -t paddleocr-gpu .

# OCR 実行
docker run --rm -it \
  --gpus all \
  -v "${PWD}:/workspace" \
  paddleocr-gpu \
  python ocr_pipeline.py
```

</details>

---

## 🛠️ 主な機能

### ✅ hokuto_scrape_xml_dump.py の特徴

- **OCR 不要の高精度テキスト**（UI ツリーから直接読み取り）
- 3 回連続で新規テキスト 0 件なら自動停止
- 重複除去・順序保持

### ✅ hokuto_scrape.py の特徴

- 自動スクロール＆キャプチャ
- 重複検知で自動停止
- 中央領域のみクロップ（上下 10% カット）
- BAN 対策（ランダム待機時間）

### ✅ hokuto_marge_ocr_xml_reviews.py の特徴

- OCR の見出し構造 + XML の高精度本文を結合
- 分裂レビューの自動結合
- 重複レビュー除去（類似度 92% 以上）
- ノイズフィルター搭載
- Excel 出力（ヘッダー装飾・オートフィルター付き）

---

## 📌 注意事項

- 個人の学習・情報収集目的で使用してください
- 対象アプリの利用規約を確認してください
- 短時間の大量アクセスはアカウント制限の可能性があります
- `ADB_PATH` はご自身の BlueStacks インストール先に合わせて変更してください

---

## 📖 詳細ドキュメント

各スクリプトの詳細な使い方・パラメータ調整については、[Wiki](https://github.com/torikabuto-fkg/medical-edge-scraper/wiki) を参照してください。

---

## License

MIT
