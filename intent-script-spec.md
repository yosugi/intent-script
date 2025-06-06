# IntentScript 仕様書

## 基本コンセプト

IntentScript は、AI に「何をするか」を伝えるための言語で、
従来のプログラミング言語が「どのように実装するか」に焦点を当てるのに対し、
意図や目的の記述に集中する。

また、IntentScript の自然言語的な記述やコメントは、生成AI（LLM）による解釈と実装支援を前提としている。
静的な仕様記述にとどまらず、人間と AI が意図を共有・発展させるための共創的な言語基盤を目指している。

## 言語設計の基本方針

* YAMLベースの構文
  * 専用のパーサが不要で直感的に記述可能
  * 形式言語と自然言語の中間を目指す
* チューリング完全性の回避
  * 再帰やループなどの複雑な制御構造を意図的に制限
  * 意図の構造化に特化し、実装詳細からユーザーを解放
* 漸進的形式性の支援
  * 形式的な厳密さと自然言語的な柔軟性のバランスを取る
  * 同一システム内で異なる抽象度レベルの混在を許容
* 習得コストゼロ
  * 既存の自然言語を知っていれば誰でも使い始められる
  * プログラミング知識を前提としない
* 環境構築コストゼロ
  * 外部ストレージにある仕様書の読み込みに対応
  * IntentScript と指示を LLM のプロンプトに入力するだけで利用可
* AI による柔軟な実行
  * エラー処理や例外的な状況は AI による適切な判断に委ねる

### ファイル形式と拡張子

IntentScript ファイルは `.is.yml` という拡張子を使用する：
* `.is` - IntentScript であることを示す
* `.yml` - YAML ベースの構文であることを示す

この二重拡張子により、エディタやツールは IntentScript ファイルを YAML ファイルとして認識しつつ、
専用の処理や構文強調表示などを適用することができる。

例：
* `order-entity.is.yml` - 注文エンティティの IntentScript 定義
* `shipping-function.is.yml` - 配送料計算関数の IntentScript 定義
* `product-catalog.is.yml` - 商品カタログの IntentScript 定義

このファイル名は一例であり、プロジェクトの要件や好みに応じて柔軟に調整できる。

## 使用仕様の明示: include

IntentScript では、使用可能な構文や定義の範囲を明示するために `include` を使用できる。
これにより、LLM やツールは読み込んだ範囲内で補完・検証を行うことが可能になる。

```yaml
include:
  - raw.githubusercontent.com/yosugi/intent-script/refs/heads/main/intent-script-spec.md
```

* `include` はファイル先頭に記述し、読み込むファイルを列挙する
* 読み込まれた構文・型・演算子等はその IntentScript 内で有効となる

## 漸進的形式性の概念

IntentScriptの「漸進的形式性」は、形式的な厳密さと自然言語表現のバランスを取るための概念である。
同一システム内で異なる抽象度レベルの混在を許容する。

IntentScriptでは、ユーザーの意図や状況に応じて、これらのレベル間を自由に行き来することができる。

### 漸進的形式性の3レベル

* レベル0（自然言語）: 自然言語主体の表現、AIによる意図解析に依存
  ```text
  ユーザーには名前、メールアドレス、年齢の情報がある。
  名前は必須で最大50文字まで入力可能。
  メールアドレスも必須で有効な形式でなければならない。
  また、ユーザーの年齢は18歳以上のみ許容する。
  ```

* レベル1（混合型）: 構造化された形式と自然言語の混合
  ```yaml
  User:
    name: 文字列、必須、最大50文字
    email: 有効なメールアドレス、必須
    age: 数値、18歳以上
  ```

* レベル2（形式的）: 完全に形式的な構文で、厳格なYAML構造として実装
  ```yaml
  User:
    name: string{required, max_length: 50}
    email: string{required, format: "email"}
    age: int{min: 18}
  ```

なお、これらのレベルは便宜的な分類である。
実際には明確な境界があるわけではなく、形式的な表記から自然言語まで連続している。

## 言語構造と基本構文

### 要素定義の基本形式

IntentScriptでは、すべての定義（値、エンティティ、関数など）に一貫した構文を使用する：

```yaml
# 値/定数の定義例
TAX_RATE: 0.1
DEFAULT_CURRENCY: "JPY"
SHIPPING_ZONES: [domestic, asia, worldwide]

# 基本的なエンティティ定義例：ユーザーエンティティ
User:
  name: string{required, max_length: 50}
  email: string{required, format: "email"}
  age: int{min: 18}

# def キーワードとオプション(後述)付きの定義例：商品エンティティ
def Product{kind: entity}:
  name: string{required, max_length: 100}
  price: int{currency: DEFAULT_CURRENCY}  # 定義した定数を使用
```

要素定義は名前、その後にコロン（:）が続く。
定義の中身は、値そのもの、またはインデントされた属性と値のペアで構成される。

必要に応じて、def キーワード、オプション（名前の後の{}）を指定できる。
この 2 つは互いに独立しており、どちらか一方のみを使用することも可能である。
意味は以下の通り。

* def キーワード: `def`
  * 明示的に「これは定義である」ことを示す
  * 形式的な文書や大規模システム向け
* 属性: `User{kind: entity}:`
  * 種類や追加属性を明示的に指定
  * 複雑なシステムや明確な種類区別が必要な場合

### 値/定数の定義

IntentScriptでは、再利用可能な値や定数を定義できる：

```yaml
# シンプルな値の定義
TAX_RATE: 0.1
DEFAULT_CURRENCY: "JPY"
MAX_ORDER_ITEMS: 100

# 複合的な値の定義
SHIPPING_ZONES: [domestic, asia, worldwide]
ERROR_MESSAGES:
  not_found: "商品が見つかりません"
  out_of_stock: "在庫切れです"
```

定義した値は、エンティティや関数の定義内で参照できる：

```yaml
Product:
  price: int{currency: DEFAULT_CURRENCY}
  tax_included_price: int{
    derive: price * (1 + TAX_RATE)
  }
```

### エンティティ定義

IntentScriptにおけるエンティティ定義は、
対象となるデータ構造の意図を簡潔かつ構造的に表現するものであり、
仕様記述やAIによる処理生成の基盤となる。

```yaml
Product:
  name: string{required, max_length: 100}
  price: int{min: 1}
  discount_rate: 0.8
  tag: list<string>{max_items: 5}
  discount_price: int{
    derive: price * (1 - discount_rate)
  }
```

### 関数定義

IntentScriptにおける関数定義は、
処理の「目的」と「構造」を明示的に記述するものであり、
AIによる実装支援を前提とした宣言的なスタイルを採用している。

```yaml
calculate_shipping_cost:
  description: "重量と距離に基づいて送料を計算する"
  inputs:
    weight: int{unit: kg}
    distance: int{unit: km}
  output: int{currency: JPY}
  logic: |
    基本料金は500円。
    重量1kgあたり200円を加算。
    距離100kmごとに100円を加算。
```

### 関数呼び出し

ユーザ定義関数を下記の形式で呼び出すことができる：

```yaml
shipping_cost: int{
  derive: calculate_shipping_cost(weight: 10, distance: 20)
}
```

* 書式: `関数名(引数1, 引数2, ...)`
* 引数にはプリミティブ値・構造型・`if(...)`などの式を指定可能
*  キーワード引数を使用することで、引数の意味を明示的に示すことができる
    * 順序を問わず使用可能
* 関数は副作用を持たない純粋関数として扱われる

### 型システム

IntentScriptでは以下の型をサポートする：

* プリミティブ型: string, int, boolean, date, etc...
* コレクション型: list<T>, map<K,V>
* ユーザー定義型: 他のエンティティへの参照
* Union型: 複数の型の選択を表現。例: string|null, "active"|"inactive"

ジェネリック型には `<>` 記法を使用する：
```yaml
order_items: list<Product>
product_prices: map<string, int>
```

### プロパティとオプション

プロパティにはオプションを `{}` で指定できる：

```yaml
name: string{required, max_length: 100}
price: int{currency: JPY}
```

### YAML記法：コメントと特殊文字

IntentScriptではYAMLのコメント記法を使用する：

```yaml
# これは行全体のコメント
entity:
  name: string  # 行末コメントも可能
  # コメントはインデントに合わせることもできる
```

コメントは漸進的形式性の重要な補完機能となる。
特に、形式的に厳密な記述（レベル2）を使用する場合でも、コメントを通じて自然言語での意図説明を追加できる：

```yaml
# レベル2の形式的記述だが、コメントで意図を自然言語で補足している
User:
  name: string{required, max_length: 50} # 必須で最大50文字まで入力可能
  email: string{required, format: "email"} # 必須で有効な形式でなければならない
  age: int{min: 18} # ユーザーの年齢は成年である18歳以上のみ許容する
```

YAMLの特殊文字（>, |, *, &, -, ?, :, {, } など）を式や条件の中で使用する場合は、
必要に応じて引用符で囲むことを推奨する。

YAMLパーサとの互換性を重視する場合や、構文エラーを防ぎたい場面では、
"items |> filter(_ > 100)" のように式全体を文字列リテラルとして記述すると安全である。

```yaml
filtered_items: list<int>{
  derive: "items |> filter(_ > 100)"  # '>'は特殊文字なので必要に応じ引用符で囲む
}
```

### YAML記法：リテラル表記

IntentScriptでは、YAMLの標準リテラル表記をそのまま使用する：

```yaml
# 文字列リテラル
simple_string: "Hello World"
multiline_string: |
  これは複数行の
  文字列である。
  インデントが保持される。

# 数値リテラル
int: 28
float: 3.14

# 真偽値リテラル
active: true
disabled: false

# 日付時刻リテラル
created_at: 2006-01-02T15:04:05Z

# リストリテラル - ブロック表記
colors:
  - red
  - green
  - blue

# リストリテラル - フロー表記
colors: [red, green, blue]

# マップリテラル - ブロック表記
point:
  x: 10
  y: 20

# マップリテラル - フロー表記
point: {x: 5, y: 15}
```

## データ操作とパイプライン

IntentScript では、処理の意図を直感的かつ宣言的に記述できるよう、パイプライン構文を採用している。
データの流れや変換操作を順に記述することで、構造と意図の可視化を両立する。

### パイプライン方式の基本

パイプライン演算子 `|>` を使用して、データ処理の流れを直感的に表現する：

```yaml
discount_prices: list<int>{
  derive: items
          |> filter(_.in_stock)
          |> map(_.price * (1 - _.discount_rate))
          |> filter("_ > 0")
}
```

### 標準パイプライン操作関数

IntentScriptは以下の基本的なパイプライン操作関数を提供する：

* map: 各要素を変換する
  ```yaml
  # 例: 価格を2倍にする
  doubled_prices: list<int>{
    derive: prices |> map("_ * 2")
  }
  ```

* filter: 条件に一致する要素を選択する
  ```yaml
  # 例: 正の数だけを選択
  positive_numbers: list<int>{
    derive: numbers |> filter("_ > 0")
  }
  ```

* sum: 数値の合計を計算する
  ```yaml
  # 例: 注文の合計金額を計算
  total_amount: int{
    derive: order_items |> map(_.price * _.quantity) |> sum
  }
  ```

* sort: 要素を並べ替える
  ```yaml
  # 例: 数値を昇順にソート
  sorted_values: list<int>{
    derive: values |> sort
  }
  ```

* reduce: 要素を集約して単一の値にまとめる
  ```yaml
  # 例: 数値の掛け合わせ（累積乗算）
  multiplied_number: int{
    derive: numbers |> reduce("_1 * _2", 1)
  }
  ```

### プレースホルダー構文

* 単一引数: `_` で引数全体、`_.property` でプロパティ参照
  ```yaml
  # 単一引数の例 - 価格が100 円以上の商品を抽出
  discounted_prices: list<int>{
    derive: products
            |> filter("_.price >= 100") # 価格が100 円以上の商品のみ抽出
  }
  ```

* 複数引数: `_1`, `_2`, `_3`... で各引数を参照
  ```yaml
  # 複数引数の例 - 数値の降順ソート
  sorted_scores: list<int>{
    derive: scores
            |> sort(_2 - _1)  # 降順にソート（_1と_2は比較される2要素）
  }
  ```

### 条件式: if(...)

IntentScriptでは、式として `if(condition, then_value, else_value)` を使用できる。
これは条件に応じて2つの値のいずれかを返す純粋な式である。

```yaml
final_price: int{
  derive: if(onSale, price * 0.9, price)
}
```

* `then_value` と `else_value` は同じ型であることが望まれるが、詳細は AI の解釈に委ねる
* 一般的な比較・論理演算子をサポート

## 自然言語の使用

IntentScriptでは、構造化された定義やパイプライン処理のどこにでも自然言語を使用できる。
漸進的形式性の概念に基づき、形式的な表現と自然言語を必要に応じて混在させることができる：

この機能により、厳密な構造が必要な部分は形式的に、詳細な説明や複雑なルール、
処理ステップは自然言語で表現するという使い分けが可能になる。
AIはこれらの自然言語表現を解釈し、意図を理解して実装に変換する。

### 属性記述での使用例

下記のように属性を自然言語でも記述できる。

```yaml
Product:
  name: string{required}
  price: int{currency: JPY}
  description: "製品の詳細な説明。マークダウン形式で書くことができる。"

  availability_rule: |
    通常は在庫があれば購入可能。
    ただし、限定商品の場合は会員のみが購入できる。
    予約商品は発売の2週間前から注文を受け付ける。

  discount_strategy: |
    - 定価が5000円以上の商品は10%割引
    - セール期間中はさらに5%追加割引
    - ゴールド会員はいつでも追加で5%割引
```

### パイプラインでの使用例

パイプラインの処理内容も自然言語で記述可。

```yaml
top_products: list<Product>{
  derive: products
          |> 在庫切れの商品は対象外
          |> 売れている順で商品をソート
          |> 先頭5つだけ取得
}
```

### 自然言語による定義

IntentScript は、構造化された定義に加えて、自然言語やコメントによる定義を許容する。
特に未定義の構文や関数については、実装環境（LLM など）がそれらを補完することを想定している。

例えば、次のようなコメントを用いることで、新しい関数を定義できる：

```yaml
# average 関数は数値の平均を計算する
```

このような記述は正式な構文ではないが、柔軟な表現や将来的な拡張を見越した設計として活用できる。

### 入出力に関する補足

IntentScript の仕様は、意図の記述に特化しており、具体的な入出力の形式（ファイル、API、標準入力出力など）は対象外である。
ただし、実行環境に依存する入出力の仕様を明示したい場合には、コメントによる補足記述を推奨する。

```yaml
# 入力: 標準入力から CSV 形式で与えられる
# 出力: 標準出力に1行ずつ出力
```

このようなコメントは、IntentScriptの意図をより明確にし、実装や検証時の補助となる。

## このドキュメントについて

* バージョン: 0.0.1
* ステータス: 本仕様は IntentScript の POC（概念実証）段階における最小限の構成仕様である。
  初期の設計検証および言語としての有効性を確認するためのものであり、今後の拡張や正式な仕様化に向けた基盤として位置づけている。
* 設計方針: 詳細な仕様定義よりも、AIによる柔軟な解釈を重視する。曖昧さは意図的なものであり、実装者とAIの判断に委ねる部分を多く残している。
* 共著について: 本仕様は、生成AIとの対話を通じて作成された。本仕様の形成における生成AIの多大な貢献に感謝する。
* ライセンス: 本仕様は MIT License に基づいて公開する。

このドキュメントは、IntentScript を「AIに意図を伝えるための言語」として発展させていくための出発点であり、今後の議論・実装・活用の土台となることを意図している。
