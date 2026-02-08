# MdToSlides

Markdown からスライドを生成するためのプロジェクト。Quarto + reveal.js を使用。

## ドキュメント

| ファイル | 説明 |
|---|---|
| [instruction.md](instruction.md) | ツール選定・オプション・ビルド手順の総合まとめ |
| [how-to-setup-quarto.md](how-to-setup-quarto.md) | Quarto CLI のセットアップ手順（Ubuntu） |

## デモ（quarto_demo/）

| ファイル | 説明 |
|---|---|
| [slides.qmd](quarto_demo/slides.qmd) | スライド原稿（Markdown + YAML front matter） |
| [Makefile](quarto_demo/Makefile) | ビルド自動化（`make html` / `make pdf` / `make preview`） |
| `quarto_demo/img/` | スライドで使用する図表（SVG） |

## クイックスタート

```bash
cd quarto_demo

# HTML スライド生成
make html

# ライブプレビュー
make preview

# PDF 出力（Chrome ヘッドレス印刷）
make pdf

# HTML + PDF を一括生成
make
```
