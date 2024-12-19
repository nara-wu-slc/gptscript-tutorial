# gptscript-tutorial

GPT系のAPIを使って処理をする [gptscript](https://github.com/gptscript-ai/gptscript) のチュートリアルです。

## 目次
- [インストール](#インストール)
- [使い方1: 基本（直接型）](#使い方1-基本直接型)
- [使い方2: 応用（テンプレート型）](#使い方2-応用テンプレート型)

## インストール
- 研究室のサーバには導入済みです
- Macであれば [Homebrew](https://brew.sh/ja/) を使って簡単にインストールできます
```
brew install gptscript
```

## 使い方1: 基本（直接型）
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

>[!NOTE]
>出力ファイル名を指定しないと適当なファイル名で出力されてしまうので注意

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


## 使い方2: 応用（テンプレート型）
### プロンプトのテンプレートを用意する
特定のファイル名を書いてしまうと大量のファイルを順番に処理したいときに困るので、テンプレートに基づいてプロンプトファイルを自動生成してAPIに投げるようにします。

例えばテキストの要約をしたいとしたとき、以下のようなファイルを `prompt_template.gpt` という名前で作り、入力ファイル名を入れる箇所を `{INPUT}`、出力ファイル名を入れる箇所を `{OUTPUT}`で表します。
```
model: gpt-4o-mini
tools: sys.read, sys.write

You are a helpful text analyst.
Please read the Japanese text file {INPUT} and write its summary into the file {OUTPUT}, following the guidelines below.

Guidelines:
- Please keep the original expressions as much as possible.
- Please do not put newline symbols inside the output.
```
>[!NOTE]
>後の処理の都合で、テンプレートファイルの拡張子は `.gpt` にしておいてください。

### API Keyを設定する
```
export OPENAI_API_KEY=...
```
API Keyの値には自分のAPI Keyを入れてください。

### テンプレートからプロンプトファイルを自動生成して gptscript を実行する
その上で、`input/*.txt` に入力ファイル群が存在するとします（異なる場合は適宜コマンドを書き換えてください）。

まず最初にプロンプトテンプレートのファイル名を指定します。
相対パスでも絶対パスでもかまいません。
```
TEMPLATE=prompt_template.gpt
```

続いて、以下のコマンドを実行します。
出力はプロンプトによって異なるはずなので、プロンプトテンプレートのファイル名を利用して、出力ファイル名は `output/prompt_template/*.txt` とします。
```
if test -n "${OPENAI_API_KEY}" && test -n "${TEMPLATE}" && test -s "${TEMPLATE}"; then \
  name=`basename ${TEMPLATE} .gpt` ;\
  mkdir -p output/${name} ;\
  for f in input/*.txt ; do \
    base=`basename ${f} .txt` ;\
    temp=`mktemp -d /tmp` ;\
    sed -e "s,{INPUT},${f},; s,{OUTPUT},output/${name}/${base}.txt," ${TEMPLATE} > ${temp} ;\
    gptscript --no-trunc --output output/${name}/${base}.log ${temp} ;\
    /bin/rm ${temp} ;\
  done ;\
fi
```

>[!NOTE]
>上記のコマンドは安全のため 環境変数 `OPENAI_API_KEY` と 変数 `TEMPLATE` が設定されていないと何もしないようにしてあります。
