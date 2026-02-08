更に以下の点に関する記述を加えて、README_2.md とこれらに関する実例が分かるデモ demo_quarto_2/ を追加してください。

  * Quartoとreveal.jsの役割分担がよくわからない。各々元々どういう目的のものなのかの説明を入れる。
  * `quarto render --to revealjs` というコマンドの意味。
  * `?print-pdf` をChromeで吐き出し という手順をもう少し丁寧に説明。
  * qmdファイルの記述ルール。普通のmdと違う点。
  * google-chrome --headless --print-to-pdf で配布用PDFを同梱 という手順をもう少し丁寧に説明。
  * ### でサブスライドを重ねるとはどういうことか？
  * incremental: true はどこに記述するべきものか？段階表示とはどういう意味か。
  * embed-resources: true で単一HTMLにバンドルとはどういう意味か。
  * スピーカーノートである ::: notes ブロックの記述作法をもう少し詳しく。またどのように活用されるものなのか（pptx変換した時にノートとして使われるのか？何らかの読み上げアプリケーション規格に則ったものなのか？）
  * layout: true スライドで背景やレイアウトを共有可能とはどういうことか。
  * reveal.js の ?print-pdf モードを使うと登壇画面と同じ状態をそのままPDF化できるとはどういうことか。
  * Chromeヘッドレス実行時は --no-sandbox --disable-dev-shm-usage がCIでも安定とはどういう意味か。
  * LaTeX派生のPDFが必要な場合は format.pdf を追加して quarto render --to pdf とはどういう意味か。
  * GitHub Actions / GitLab CI 等で quarto render → google-chrome --print-to-pdf を自動化し、slides.html / slides.pdf をアーティファクト化 という方法論についてもう少し具体的に説明してほしい。
  * _extensions/ ディレクトリに専用テーマを追加について３個ほど例を示してほしい。
  * reveal.js が PowerPoint の代わりという認識で正しいか？
  * CIでHTML/PDFを自動化する例 についての説明をもう少し加えてください。
  * 「テーマを共有できます」とあるが、何と何の間で共有できるのか？
  * 「これらをGitリポジトリに含めれば、project 配下から quarto use template lab/minimal-light のように再利用できます。」という説明がよくわからない。もう少し説明を加えてほしい。
  * 何らかのFigureをスライド右半分に表示する例、上半分に表示する例、右下1/4に表示する例に関する説明も加えてください。



