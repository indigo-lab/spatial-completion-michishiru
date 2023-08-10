# About

「JSON-LD 穴埋め方式」による JSON-LD テンプレートを用いて、地理的範囲の補完をテストするためのデータセットです。
論文は [こちら](http://example.org/) で公開されています。

このテストは [Japan Search](https://jpsearch.go.jp/) に収録されている
[NHK みちしるデータセット](https://jpsearch.go.jp/database/michi) の 4,001 件のメタデータを加工して作成されています。

# Data

## reference

正解となる JSON ファイルを収録したフォルダです。

以下は [reference/michi-D0004010001_00000.json](https://github.com/indigo-lab/spatial-completion-michishiru/blob/main/reference/michi-D0004010001_00000.json) の抜粋です。

```
{
  "@context": "http://schema.org/",
  "@id": "https://jpsearch.go.jp/data/michi-D0004010001_00000",
  "@type": "Clip",
  "name": "ＪＲ宗谷本線",
  "description": "詳細: 宗谷本線は北海道の旭川駅から名寄駅を経て稚内駅を結ぶＪＲ北海道の路線です。稚内駅は日本最北端に位置する駅です。急行礼文は旭川と稚内を結んでいましたが、2000年（平成12年）3月11日に特急「スーパー宗谷」に編入されて廃止されました。\\n\\n（この動画は、２００８年に放送したものです。）",
  "spatial": [
    {
      "@id": "https://jpsearch.go.jp/entity/place/北海道"
    },
    "都道府県: 北海道",
    "住所: （塩狩駅）上川郡和寒町"
  ]
}
```

- [JapanSearch 利活用フォーマット(JSON-LD)](https://jpsearch.go.jp/data/michi-D0004010001_00000.json) を取得・加工して作成しています
- @context は Schema.org だけを使うように設定
- @id はソースの値を継承
- @type は `Clip` を独自に付与
- name は `rdfs:label` から生成
- description は `schema:description` (複数) のうち、`詳細` で始まる最初の一件
- spatial は ソースの `schema:spatial` にセットされている都道府県 URI と <https://jpsearch.go.jp/term/property#spatial> で参照されるリソースの `schema:description` のリテラル値

spatial の値に関しては以下の特徴があり、評価方法によっていずれかの値を選択して使用できます。

- URI : Japan Search の定義する都道府県の URI が設定される、必須
- リテラル(接頭辞 都道府県) : 都道府県の文字列が設定される、カンマ区切りで複数の場合あり、任意
- リテラル(接頭辞 住所) : 任意の場所を表現する文字列が設定される、任意

## template

補完対象を `?` で表現したテンプレート JSON ファイルを収録したフォルダです。
補完手法にバリエーションがあるため、`template/*/*.json` のように、補完手法ごとのフォルダの配下に JSON が収録されています。ファイル名は reference と共通です。

以下は [template/place/michi-D0004010001_00000.json](https://github.com/indigo-lab/spatial-completion-michishiru/blob/main/template/place/michi-D0004010001_00000.json) の抜粋です。

```
{
  "@context": "http://schema.org/",
  "@type": "Clip",
  "name": "ＪＲ宗谷本線",
  "description": "詳細: 宗谷本線は北海道の旭川駅から名寄駅を経て稚内駅を結ぶＪＲ北海道の路線です。稚内駅は日本最北端に位置する駅です。急行礼文は旭川と稚内を結んでいましたが、2000年（平成12年）3月11日に特急「スーパー宗谷」に編入されて廃止されました。\\n\\n（この動画は、２００８年に放送したものです。）",
  "spatial": {
    "@type": "Place",
    "name": "?"
  }
}
```

## request

template をもとに生成した [OpenAI ChatCompletion API (gpt-3.5-turbo-0613)](https://platform.openai.com/docs/api-reference/chat) に対するリクエストボディを収録したフォルダです。
フォルダ・ファイル名の構成は template と共通です。

以下は [request/place/michi-D0004010001_00000.json](https://github.com/indigo-lab/spatial-completion-michishiru/blob/main/request/place/michi-D0004010001_00000.json) の抜粋です。

```
{
  "model": "gpt-3.5-turbo-0613",
  "messages": [
    {
      "role": "system",
      "content": "あなたは与えられた JSON-LD の中の \"?\" の部分に文字列を補完して JSON-LD を返すエージェントです"
    },
    {
      "role": "user",
      "content": "{\"@context\":\"http://schema.org/\",\"@type\":\"Clip\",\"name\":\"ＪＲ宗谷本線\",\"description\":\"詳細: 宗谷本線は北海道の旭川駅から名寄駅を経て稚内駅を結ぶＪＲ北海道の路線です。稚内駅は日本最北端に位置する駅です。急行礼文は旭川と稚内を結んでいましたが、2000年（平成12年）3月11日に特急「スーパー宗谷」に編入されて廃止されました。\\\\n\\\\n （この動画は、２００８年に放送したものです。）\",\"spatial\":{\"@type\":\"Place\",\"name\":\"?\"}}"
    }
  ],
  "temperature": 0
}
```

# Target

template における、各フォルダごとの目的は以下の通りです。詳細は論文を参照してください。

| folder             | note                                     |
| ------------------ | ---------------------------------------- |
| spatial            | schema:spatial へのリテラル補完          |
| place              | schema:Place 制約のある補完              |
| state              | schema:State 制約のある補完              |
| administrativeArea | schema:AdministrativeArea 制約のある補完 |
| address            | 住所のための構造がある場合の補完         |
| containedInPlace   | 上位地名のための構造がある場合の補完     |

# License

本レポジトリのデータは [CC0 1.0 Universal](https://github.com/indigo-lab/metadata-completion-checklist/blob/main/LICENSE) でライセンスされています。
