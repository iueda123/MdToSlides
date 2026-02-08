# Quarto × reveal.js 実践ガイド (README_2)

`demo_quarto_2/` に、学会発表スライドをHTML+PDFで共通管理するための具体例をまとめました。以下では、note002.md で挙がった疑問を順番に整理します。

## 1. Quartoとreveal.jsの役割分担

- **Quarto**: Markdownベースの `.qmd` を解析し、reveal.js/Beamer/PDF/Wordなど多様な形式へレンダリングする統合ビルドツール。PandocやJupyterを内包し、コード実行・図表挿入・テンプレ管理を担う "オーサリング基盤"。
- **reveal.js**: HTML/CSS/JSでスライドを描画するオープンソースプレゼンエンジン。ブラウザ上での登壇、プレゼンタービュー、ショートカットなど表示面を受け持つ。
- **分担イメージ**: Quartoが「Markdown → reveal.js構造化HTML」への変換とオプション注入を行い、reveal.jsが実際のアニメーション・ナビゲーションを提供する。

## 2. `quarto render --to revealjs` の意味

`quarto render slide_deck.qmd --to revealjs` は、`.qmd` のYAMLを読み取り、Pandoc経由で「reveal.js互換HTML＋サポートファイル（CSS/JS）」を生成するコマンドです。`--to` でターゲット書式を切り替えられ、`--to pdf` `--to gfm` など同じ原稿から別形式を作成できます。

## 3. `.qmd` の記述ルール（Markdownとの違い）

`demo_quarto_2/slide_deck.qmd` では以下の差異を利用しています。

- 冒頭の **YAML front matter** (`--- … ---`) でタイトル、`format` 設定、言語などを定義。
- **拡張ブロック**: `::: notes` や `::: {.columns}` のように属性を持つdivを記述可能（Pandoc fenced divs）。
- **コードチャンク**: `{python}` 等の実行ブロックや `code-line-numbers: true` 等の細かいオプションをYAML/本文で制御。
- `.md` より「構造＋設定」を厳密に書ける代わりに、YAML記法やfenced divを覚える必要がある。

## 4. サブスライド (`###`) と `layout: true`

- reveal.jsでは `##` 見出しを水平スライド、直後の `###` を同じ列の **垂直サブスライド** として扱います。`demo_quarto_2/slide_deck.qmd:18` 以降の「サブスライドA/B」が例です。登壇時は左右キーでセクション移動、上下キーでサブスライド移動。
- `layout: true`（`demo_quarto_2/slide_deck.qmd:30`）は特別なスライド属性で、「このスライドをテンプレート化し、後続のサブスライドへ背景や共通要素を継承せよ」という意味です。研究ロゴや枠線を複数枚に跨らせたい時に使います。

## 5. `incremental: true` と「段階表示」

- `format.revealjs.incremental: true`（YAML）に置くのが基本。スライド内のリスト要素が1クリックごとに表示されるモードで、聴衆の視線コントロールに使います。
- スライド単位で切り替えたい場合は本文に `:::{.incremental}` ブロックを置き、その中の要素だけ段階表示にできます。

## 6. `embed-resources: true`

- 通常reveal.jsは外部CSS/JS/画像へのリンクを含みますが、`embed-resources: true` にするとQuartoがそれらをBase64化してHTML内にインライン展開します。`slides.html` を単体配布できる反面、ファイルサイズが増える点に注意。

## 7. スピーカーノート (`::: notes`) の作法と活用

- 各スライド直後に `::: notes … :::` でMarkdownを記述。
- reveal.jsのプレゼンタービュー（`s`キー）でノートが別ウィンドウに表示され、登壇者だけの台本として利用できます。Chrome印刷でも脚注扱いでPDFに残せます。
- QuartoでPowerPoint（`--to pptx`）出力した場合は、各スライドの「ノート」欄に転記されます。特定の読み上げ規格に依存しているわけではなく、reveal.js/PowerPointなど各ターゲットがサポートするノートUIへマッピングされるイメージです。

## 8. `?print-pdf` と Chrome ヘッドレス印刷

- reveal.js は `index.html?print-pdf` にアクセスするとPDFレイアウト用CSSを適用します。
- これをヘッドレスChromeで開き、`--print-to-pdf=slides.pdf` を指定すると登壇画面と同じ段階表示・縦横スタックを保ったPDFが出力されます。
- `demo_quarto_2` では以下のコマンドで自動化しています。

```bash
cd demo_quarto_2
CHROME_BIN=${CHROME_BIN:-google-chrome}
"$CHROME_BIN" \
  --headless --disable-gpu --no-sandbox --disable-dev-shm-usage \
  --print-to-pdf="$(pwd)/slide_deck.pdf" \
  "file://$(pwd)/slide_deck.html?print-pdf"
```

- `--no-sandbox` と `--disable-dev-shm-usage` は、CI（継続的インテグレーション）や制限付きLinuxでChromeのサンドボックス/共有メモリ制限が原因のクラッシュを避けるための定番フラグです。

## 9. LaTeX派生PDF (`format.pdf` + `quarto render --to pdf`)

- `.qmd` のYAMLに `format.pdf` セクションを追加すると、`quarto render slide_deck.qmd --to pdf` でBeamer文書が生成されます。
- TeX Live/ TinyTeX などのLaTeX環境が必要で、reveal.js版とは異なるデザインになります。厳密なPDF体裁が求められる学会や冊子向けに有効です。

## 10. CIでHTML/PDFを自動化する例

ローカルと同じコマンドをCIに移すだけで、pushのたびにHTML/PDFが再生成されます。典型的な構成は以下の3段階です。

1. リポジトリをチェックアウトし、Quarto CLIとChromeをセットアップ
2. `quarto render ... --to revealjs` でHTML化
3. `google-chrome --headless ... --print-to-pdf` でPDF化し、成果物をアーティファクト化

GitHub Actionsの最小例:

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
        run: quarto render demo_quarto_2/slide_deck.qmd --to revealjs
      - name: Export PDF via Chrome
        run: |
          sudo apt-get update && sudo apt-get install -y google-chrome-stable
          google-chrome --headless --no-sandbox --disable-dev-shm-usage \
            --print-to-pdf="demo_quarto_2/slide_deck.pdf" \
            "file://$(pwd)/demo_quarto_2/slide_deck.html?print-pdf"
      - uses: actions/upload-artifact@v4
        with:
          name: slides
          path: |
            demo_quarto_2/slide_deck.html
            demo_quarto_2/slide_deck.pdf
```

`actions/upload-artifact` を `pages` デプロイに差し替えれば、CI完了時にHTML/PDFを自動公開できます。同じ構成をGitLab CIなら `artifacts: paths:`、Jenkinsなら `archiveArtifacts` に置き換えるだけで済みます。

## 11. `_extensions/` にテーマを追加する例

Quartoでは `_extensions/<org>/<name>/` に `_extension.yml` とSCSS/JSを置くと、「同じリポジトリ内の別プロジェクト間」「チームメンバー間」「社内標準テンプレート」といった単位でテーマを共有できます。例えば:

1. `_extensions/lab/minimal-light/` : `theme.scss` でラボのブランドカラーとフォントを定義し、`_extension.yml` で `title-slide-attributes` を設定。
2. `_extensions/lab/dark-grid/` : 背景にグリッド模様やコード向け配色を加えたSCSSと、`revealjs-plugins.js` でカスタムキーボードショートカットを追加。
3. `_extensions/lab/poster-layout/` : `layout: true` と合わせるためのレイアウトスニペットを `partials/` に用意し、`qmd` 側で `format.revealjs.theme: lab/poster-layout` と指定。

これらをGitリポジトリに含めれば、別プロジェクトが `quarto use template lab/minimal-light` を実行するだけでテンプレをコピーできます（`quarto use template` は Git リポジトリを取得して `_extensions` を含むスターター構成を展開するコマンドです）。つまり、研究室全体で統一テーマをメンテしつつ、各自の発表プロジェクトに素早く配布できる仕組みになります。

## 12. 図の配置例

`demo_quarto_2/slide_deck.qmd` 末尾に3種類のレイアウトを追加しました。

1. **右半分**: `::: {.columns}` で左右2カラムを作り、右カラムに `![キャプション](img/figure-half.svg){width=100%}` を置く。
2. **上半分**: `::: {.layout-split}` というカスタムdivを用意し、CSSで `display: grid` + `grid-template-rows: 1fr 1fr` とすると、上段に `![](){width=90%}`、下段に文章を配置できる。
3. **右下1/4**: `::: {.absolute}` ブロックに `style="position:absolute; right:5%; bottom:5%; width:25%;"` のような属性を指定し、その中に `![]()` を入れると、スライド右下に小さな図を固定可能。

QuartoはPandocの属性付きdiv/画像属性をそのままreveal.jsへ渡すので、HTML/CSSの知識があれば柔軟に配置できます。詳細は `slide_deck.qmd:120-190` を参照してください。

## 13. reveal.js は PowerPoint の代わり？

- 目的は近い（スライド表示）もののアーキテクチャが異なり、reveal.js はブラウザで動くWebスライド、PowerPointはデスクトップアプリ＋pptxフォーマット。
- Quartoなら `.qmd` から `--to revealjs` でHTMLスライド、`--to pptx` でPowerPointも生成できるため、「ブラウザ登壇＋pptx納品」の両立が可能。
- つまり、reveal.jsを“完全な代替”と捉えるより、ブラウザ登壇に特化した形式と理解し、必要に応じてpptx/PDFを併産する、という使い分けが現実的です。

## 14. CI説明の補足

- Chromeが必要なため、CIイメージに含まれない場合は `apt-get install google-chrome-stable` などのインストール手順を追加します。
- 共有メモリが64MBの`/dev/shm`しかないCIでは `--disable-dev-shm-usage` を必ず付与し、`--no-sandbox` で名前空間制限を解除しないとクラッシュするケースがあります。
- 成果物を配布したい場合はGitHub ActionsのArtifacts、GitLab Pages、S3アップロードなど “置き場所” を決めておくと利用者が迷いません。

## 15. 付属デモ `demo_quarto_2`

- `slide_deck.qmd`: 上記の設定・サブスライド・ノート等をすべて含んだ原稿
- `slide_deck.html`: `quarto render --to revealjs` の成果物
- `slide_deck.pdf`: Chromeヘッドレス印刷で出力したPDF

同デモには「図を右半分/上半分/右下1/4に配置する例」も追加してあります（`slide_deck.qmd:120` 以降）。`::: {.columns}`, `style="position:absolute"` などPandocの属性を使って実現しています。

実際のコマンドログや生成物は `demo_quarto_2/` 配下を参照してください。
