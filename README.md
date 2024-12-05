# gptscript-tutorial

GPT系のAPIを使って処理をする [gptscript](https://github.com/gptscript-ai/gptscript) のチュートリアルです。

## インストール
- 研究室のサーバには導入済みです
- Macであれば [Homebrew](https://brew.sh/ja/) を使って簡単にインストールできます
```
brew install gptscript
```

## 使い方
### プロンプトを書いたファイルを用意する
例：韓国語を日本語に翻訳して、原文と訳文の各行をタブ記号で連結したTSVファイルとして出力してもらうためのスクリプト (`translate-Korean-to-Japanese.gpt` とする)
```
model: gpt-4o-mini
tools: sys.read, sys.write

You are a Japanese professional translator and also fluent in Korean.
Please read the Korean text file "source/file001.txt", translate their contents into Japanese following the guidelines below, and store the translations into the tab-separated text (TSV) file "target/file001.tsv".

Guidelines:
- Put the attribute of the output file first.
- Each line must be translated in order and into EXACTLY one line to keep the line-wise correspondence between Korean and Japanese text.
- Please maintain the meaning of the original text.
- Each output line consists of the original Korean text line and the translated Japanese text line separated with the tab character, terminated with the newline characater.
```

いくつかのコツ
- できたら英語で書く
- GPTに役割を与え、どういうスキルを持っているかを示す
- 指示は具体的に
- タスク遂行上の補足事項がある場合は箇条書き等を使って列挙する
  - ただし、ファイルの出力形式の指定は厳密に守られる保証がないので注意が必要（処理結果のチェックをすること）
- ファイル名を相対パスで書く場合はスクリプト実行時のカレントディレクトリを基準にすること

### API Keyを設定する
```
export OPENAI_API_KEY=...
```
API Keyの値には自分のAPI Keyを入れてください。

### スクリプトを実行する
読み込むファイル `source/file001.txt` を用意すること（出力ファイルを格納するディレクトリの作成も必要）
```
gptscript --no-trunc -o gptscript.log gptscript.gpt
```
実行後 `target/file001.tsv` というファイルができるはず。
