# Quarto × reveal.js → PDF デモ

学会/研究室の発表を Markdown で管理し、Quarto から reveal.js スライドと PDF 配布資料を同時に用意するまでの流れを最小構成で記録しています。

## つくったもの

```
demo_quarto/
├── README.md       # この説明
├── slides.qmd      # 原稿（Quarto）
├── slides.html     # reveal.js 版（ブラウザ登壇用）
└── slides.pdf      # print-pdf で書き出した配布用PDF
```

## 1. 前提ツール

- [Quarto CLI](https://quarto.org/docs/download/) 1.5 以上
- Google Chrome または Chromium（`--headless --print-to-pdf` が利用できるもの）
- LaTeXは不要（reveal.js版をブラウザでPDF化する運用）

※今回の環境では `quarto-1.5.57/bin/quarto` を直接叩いています。PATHに追加済みの方は単に `quarto` とだけ書けばOKです。

## 2. HTML スライドを生成

```bash
cd demo_quarto
quarto render slides.qmd --to revealjs
# -> slides.html を生成（埋め込みリソースで単一HTML化）
```

`slides.qmd` は `##` 見出しでページを区切り、`::: notes` でスピーカーノートを入れるという最低限の作法だけを使ったテンプレです。必要に応じて `format.revealjs` 配下にテーマや遷移のオプションを足してください。

## 3. reveal.js 版を PDF に出力

reveal.js の `?print-pdf` モードを Chrome のヘッドレス印刷でPDF化しています。CIでも再現しやすい方法です。

```bash
cd demo_quarto
CHROME_BIN=${CHROME_BIN:-google-chrome}
"$CHROME_BIN" \
  --headless --disable-gpu --no-sandbox --disable-dev-shm-usage \
  --print-to-pdf="$(pwd)/slides.pdf" \
  "file://$(pwd)/slides.html?print-pdf"
# -> slides.pdf
```

Chromeが利用できない場合は、`quarto render slides.qmd --to pdf` でLaTeXベースのPDFを作る方法もあります（TeX環境の整備が必要）。

## 4. よくあるハマりどころ

- **フォントの警告**: ネットワークが遮断されているとCDNフォントが取得できず警告が出ます。`format.revealjs.theme` でローカルCSSを指定するか、`embed-resources: true` と合わせて自前のフォント設定を行ってください。
- **Chromeのsandboxエラー**: CIや制限環境では `--no-sandbox --disable-dev-shm-usage` オプションを付けると安定します。
- **PDF化したときのページ崩れ**: `?print-pdf` はreveal.jsの公式ルートなので、基本的にブラウザ表示と同じ段階表示を保ったままPDFが得られます。段階表示を一括表示したい場合は `--print-to-pdf-no-header` と `pdfSeparateFragments: true` を組み合わせます。

## 5. 次のアクション

- `quarto preview slides.qmd` でライブプレビューを行いながら推敲。`./quarto-1.5.57/bin/quarto preview slides.qmd `
- GitHub Actions / GitLab CI 等で `quarto render` → `google-chrome --print-to-pdf` を自動化し、`slides.html` / `slides.pdf` をアーティファクト化
- reveal.js テーマを研究室用にカスタムしたい場合は `_extensions/` ディレクトリに専用テーマを追加
