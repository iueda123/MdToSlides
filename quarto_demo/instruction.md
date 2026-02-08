# Markdown スライド生成 総合まとめ

note_1〜note_4 の内容を統合・整理したドキュメント。

---

## 1. ツール選定の指針

| ゴール | 推奨ツール | 理由 |
|---|---|---|
| 学会・研究室発表（HTML登壇＋PDF配布） | **Quarto (reveal.js)** | 安定性が高く、同一 `.qmd` からHTML/PDF/PPTXを生成可能 |
| PPTX納品・共同編集が前提 | **Pandoc + reference.pptx** | 編集可能pptxの生成に最適。テンプレ（スライドマスター）で体裁を制御 |
| 軽量に回したい（VSCode完結） | **Marp** | CSSテーマ運用で手軽。日本語フォント指定に注意 |
| ライブコーディング・デモ中心 | **Slidev** | インタラクティブ機能が豊富。登壇はWeb版、配布はPDFで使い分け |
| 最小構成のHTMLスライド | **remark** | HTML/CSS知識があれば自由度が高い |

## 2. 共通ベストプラクティス（ツール非依存）

- **1スライド＝1メッセージ**。箇条書きは5〜7行以内。
- 見出し階層を先に設計してから本文を書く（並べ替え・削除が容易）。
- スタイルはテーマ／テンプレートに集約し、文章内にハードコードしない。
- 図表・画像は相対パスで同梱（リンク切れ防止）。
- `Makefile` / `justfile` / CI で**ワンコマンドビルド**を確保。
- **スピーカーノート**を原稿側に書く（台本とスライドの乖離防止）。
- PDF配布時はアニメーション前提の作りを避けるか、PDF用の別ビルドを用意。

## 3. Quarto × reveal.js の基本構造

### 役割分担

- **Quarto** = オーサリング基盤。`.qmd`（Markdown＋YAML）を解析し、Pandoc経由で多形式に変換。コード実行・図表挿入・テンプレ管理を担う。
- **reveal.js** = プレゼンエンジン。ブラウザ上でのスライド描画・アニメーション・ナビゲーションを提供。

Quartoが「Markdown → reveal.js 構造化HTML」への変換を行い、reveal.jsが表示を担当する。

### `.qmd` と `.md` の違い

- **YAML front matter** (`--- … ---`) で `format` 設定、タイトル、言語などを定義。
- **拡張ブロック**: `::: notes`、`::: {.columns}` などPandoc fenced divs が使える。
- **コードチャンク**: `` ```{python} `` 等の実行ブロックや `code-line-numbers` 等のオプション制御。

### スライド区切りとサブスライド

- `##` = 水平スライド（セクション区切り）、左右キーで移動。
- `###` = 垂直サブスライド（同セクション内の掘り下げ）、上下キーで移動。

## 4. 主要オプション

  * qmdファイルの冒頭に生成物の挙動を制御する各種設定を記述する。
  * YAML front matter で記述する。

| オプション | 記述場所 | 効果 |
|-----------|----------|------|
| `incremental: true` | YAML `format.revealjs` 配下 | リスト要素を1クリックごとに段階表示。スライド単位なら `:::{.incremental}` ブロックで制御 |
| `embed-resources: true` | YAML `format.revealjs` 配下 | CSS/JS/画像をBase64化しHTML内にインライン展開。単一HTMLで配布可能（サイズ増に注意） |
| `layout: true` | スライド属性 | そのスライドをテンプレート化し、後続サブスライドへ背景・共通要素を継承 |

----------------------------------

## 5. スピーカーノート

```markdown
## スライドタイトル

本文...

::: notes
ここに話し手用のノートを記述する。
このノートは reveal.js の `s` キーでプレゼンタービューに表示される。
`--to pptx` 出力時はPowerPointのノート欄に転記される。
:::
```

特定の読み上げ規格に依存せず、各ターゲット（reveal.js / PowerPoint）のノートUIへマッピングされる。

----------------------------------

## 6. ビルドコマンド

### HTML スライド生成

```bash
quarto render slides.qmd --to revealjs
# -> slides.html
```

### PDF 出力（Chrome ヘッドレス印刷）

reveal.js の `?print-pdf` モードを利用し、登壇画面と同じ状態をPDF化する。

```bash
CHROME_BIN=${CHROME_BIN:-google-chrome}
"$CHROME_BIN" \
  --headless --disable-gpu --no-sandbox --disable-dev-shm-usage \
  --print-to-pdf="$(pwd)/slides.pdf" \
  "file://$(pwd)/slides.html?print-pdf"
```

- `--no-sandbox`: CIや制限環境でサンドボックス制限によるクラッシュを回避。
- `--disable-dev-shm-usage`: `/dev/shm` が小さいCI環境での共有メモリ不足を回避。

### LaTeX ベース PDF（別ルート）

```bash
quarto render slides.qmd --to pdf
```

TeX Live / TinyTeX が必要。reveal.js版とは異なるデザイン。厳密なPDF体裁が求められる場合向け。

### Makefile の役割（quarto_demo/Makefile）

- `make html` / `make pdf` / `make` で HTML と PDF の生成を一括化する。
- `make pdf` は `slides.html` の生成後に Chrome のヘッドレス印刷で `slides.pdf` を作る（reveal.js の `?print-pdf` を使用）。
- `make preview` は `quarto preview` によるローカルサーバを起動する。
- `make clean` は生成物（`slides.html` と `slides.pdf`）だけを削除する。
- `CHROME_BIN` を環境変数で差し替え可能（例: `CHROME_BIN=chromium`）。

#### Makefile とは何か

- bash スクリプトではなく、`make`（GNU Make など）用の設定ファイル。独自の文法で「ターゲット」「依存関係」「レシピ」を記述する。
- レシピ部分は通常シェルで実行されるため、見た目はシェルコマンドに近い。

#### 使い方の例

```bash
# HTMLとPDFをまとめて生成
make

# HTMLのみ生成
make html

# PDFのみ生成（HTMLを先に生成）
make pdf

# プレビュー用サーバ起動
make preview

# 生成物を削除
make clean

# Chrome実行ファイルを差し替えてPDF生成
CHROME_BIN=chromium make pdf
```

#### 「行うべき事はありません」について

- `make: 'all' に対して行うべき事はありません.` や `make: 'html' に対して行うべき事はありません.` はエラーではなく、依存関係が最新で再ビルド不要という意味。
- 再生成したい場合は `make clean && make`、または `make -B html` のように強制ビルドする。

## 7. 図の配置パターン

### 右半分

```markdown
::: {.columns}
::: {.column width="50%"}
テキスト
:::
::: {.column width="50%"}
![キャプション](img/figure.svg){width=100%}
:::
:::
```

### 上半分

```markdown
::: {.layout-split}
![](img/figure.svg){width=90%}

テキスト（下段）
:::
```

CSS で `display: grid; grid-template-rows: 1fr 1fr` を指定。

### 右下 1/4

```markdown
::: {.absolute style="position:absolute; right:5%; bottom:5%; width:25%;"}
![](img/figure.svg)
:::
```

## 8. テーマの共有と再利用

`_extensions/<org>/<name>/` に `_extension.yml` + SCSS/JS を配置。

| 例 | 内容 |
|---|---|
| `_extensions/lab/minimal-light/` | ラボのブランドカラー・フォントを定義する `theme.scss` |
| `_extensions/lab/dark-grid/` | コード向け配色＋グリッド背景。カスタムキーボードショートカット追加 |
| `_extensions/lab/poster-layout/` | `layout: true` 向けのレイアウトスニペット |

Gitリポジトリに含めれば、別プロジェクトから以下で再利用可能:

```bash
quarto use template lab/minimal-light
```

（Gitリポジトリを取得し `_extensions` を含むスターター構成を展開するコマンド）

## 9. CI 自動化（GitHub Actions 例）

```yaml
name: build-slides
on: [push]
jobs:
  render:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: quarto-dev/quarto-actions/setup@v2
      - name: Render reveal.js
        run: quarto render slides.qmd --to revealjs
      - name: Export PDF via Chrome
        run: |
          sudo apt-get update && sudo apt-get install -y google-chrome-stable
          google-chrome --headless --no-sandbox --disable-dev-shm-usage \
            --print-to-pdf="slides.pdf" \
            "file://$(pwd)/slides.html?print-pdf"
      - uses: actions/upload-artifact@v4
        with:
          name: slides
          path: |
            slides.html
            slides.pdf
```

`upload-artifact` を `pages` デプロイに差し替えれば自動公開も可能。GitLab CI なら `artifacts: paths:`、Jenkins なら `archiveArtifacts` に置き換える。

## 10. reveal.js と PowerPoint の関係

- 目的は近い（スライド表示）が、reveal.js はブラウザ上のWebスライド、PowerPoint はデスクトップアプリ＋pptxフォーマット。
- Quarto なら同一 `.qmd` から `--to revealjs`（HTML）と `--to pptx`（PowerPoint）を両方生成できる。
- reveal.js を"完全な代替"と捉えるより、**ブラウザ登壇に特化した形式**と理解し、必要に応じて pptx/PDF を併産するのが現実的。

## 11. よくあるハマりどころ

- **フォント警告**: ネットワーク遮断時にCDNフォントが取得不可。ローカルCSSでフォント指定を固定する。
- **Chrome sandbox エラー**: `--no-sandbox --disable-dev-shm-usage` で対処。
- **PDF ページ崩れ**: `?print-pdf` は reveal.js 公式ルートなので基本は安定。段階表示の一括表示には `pdfSeparateFragments: true` を組み合わせる。
- **Marp の日本語フォント混在**: CSSでフォント指定を明示的に固定する。

## 12. 最小ディレクトリ構成

```
project/
├── slides.qmd          # 原稿
├── slides.html          # reveal.js 版（生成物）
├── slides.pdf           # Chrome ヘッドレス印刷（生成物）
├── img/                 # 図表
├── _extensions/         # カスタムテーマ（任意）
├── Makefile             # ワンコマンドビルド（推奨）
└── .github/workflows/   # CI 設定（任意）
```
