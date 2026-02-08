# Quarto CLI セットアップ手順 (Ubuntu)

## 1. インストール（.deb パッケージ）

公式推奨の方法。Pandoc・Typst・Deno・Dart Sass が同梱されるため、別途インストール不要。

```bash
# /usr/local/bin が無い場合は先に作成（dpkg が失敗する原因になる）
sudo mkdir -p /usr/local/bin

# 最新安定版をダウンロード（バージョンは適宜変更）
wget https://github.com/quarto-dev/quarto-cli/releases/download/v1.8.27/quarto-1.8.27-linux-amd64.deb

# インストール
sudo dpkg -i quarto-1.8.27-linux-amd64.deb

# 依存パッケージの不足があれば解消
sudo apt-get install -f
```

  * 最新バージョンの確認: <https://github.com/quarto-dev/quarto-cli/releases>
  * 初回動作確認に使ったバージョン： 1.5.57 
  
## 2. インストール確認

```bash
quarto --version
quarto check
```

`quarto check` は以下を検証する:

- Quarto / Pandoc / Typst / Deno / Dart Sass のバージョン
- TinyTeX の有無
- Chromium の有無
- Python 3 / Jupyter の検出
- R / knitr の検出（R がインストールされている場合）

## 3. オプション: PDF 生成環境

### reveal.js → Chrome 印刷ルート（LaTeX 不要）

```bash
# Chrome / Chromium がなければインストール
sudo apt-get install -y google-chrome-stable
# または
sudo apt-get install -y chromium-browser
```

### Beamer / LaTeX ルート

```bash
quarto install tinytex
```

TinyTeX は `~/.TinyTeX` に配置される軽量 TeX ディストリビューション。

## 4. オプション: Mermaid / Graphviz 図の PDF 埋め込み

PDF や DOCX 内で Mermaid / Graphviz を画像レンダリングするには Chromium が必要。

```bash
quarto install chromium
```

## 5. PATH の確認

`.deb` インストール後は `/usr/local/bin/quarto` に置かれる。PATH に含まれていない場合:

```bash
echo '' >> ~/.bashrc
echo '#--------------------------------"' >> ~/.bashrc
echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.bashrc
echo '#--------------------------------"' >> ~/.bashrc
echo '' >> ~/.bashrc

source ~/.bashrc
```

## 6. tarball で任意ディレクトリに配置する場合

```bash
# ダウンロード・展開
wget https://github.com/quarto-dev/quarto-cli/releases/download/v1.8.27/quarto-1.8.27-linux-amd64.tar.gz
tar -xzf quarto-1.8.27-linux-amd64.tar.gz

# シンボリックリンクで PATH に通す
ln -s "$(pwd)/quarto-1.8.27/bin/quarto" ~/.local/bin/quarto
```

root 権限なしでインストールしたい場合や、プロジェクト内にバージョン固定したい場合に便利。

## 7. よくあるトラブル

| 症状 | 原因 | 対処 |
|---|---|---|
| `dpkg -i` で失敗 | `/usr/local/bin` が存在しない | `sudo mkdir -p /usr/local/bin` を先に実行 |
| `quarto: command not found` | PATH に `/usr/local/bin` がない | `.bashrc` に追記して `source` |
| ダウングレードで壊れる | 新旧パッケージの競合 | `sudo dpkg -r quarto` で削除してから再インストール |
| VSCode 拡張が検出しない | conda 環境内にインストールしている | システムワイドに `.deb` でインストールし直す |
| knitr エラー | apt で入れた R パッケージとの不整合 | R 内から `install.packages()` で CRAN 版を使う |
| Chrome PDF 化でクラッシュ | CI 環境のサンドボックス制限 | `--no-sandbox --disable-dev-shm-usage` を付ける |

## 8. 動作確認（最小テスト）

```bash
# テスト用 qmd を作成
cat <<'EOF' > /tmp/test.qmd
---
title: "Test"
format: html
---

## Hello

Quarto is working.
EOF

# レンダリング
quarto render /tmp/test.qmd

# ブラウザで確認
google-chrome /tmp/test.html
```
