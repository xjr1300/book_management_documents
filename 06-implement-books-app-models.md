# 書籍アプリのモデルの実装

- [書籍アプリのモデルの実装](#書籍アプリのモデルの実装)
  - [書籍（`books`）アプリの追加](#書籍booksアプリの追加)
  - [書籍アプリモデルの実装](#書籍アプリモデルの実装)
    - [作成日時及び更新日時モデルフィールドの共有](#作成日時及び更新日時モデルフィールドの共有)
    - [書籍分類モデルの実装](#書籍分類モデルの実装)
    - [書籍分類詳細モデルの実装](#書籍分類詳細モデルの実装)
    - [書籍モデルの実装](#書籍モデルの実装)
      - [プライマリーキーが指定されていないモデル](#プライマリーキーが指定されていないモデル)
      - [UUIDとULID](#uuidとulid)
      - [ULIDFieldモデルフィールドの定義](#ulidfieldモデルフィールドの定義)
    - [書籍モデルの定義](#書籍モデルの定義)
  - [書籍、書籍分類及び書籍分類詳細フィクスチャーの作成](#書籍書籍分類及び書籍分類詳細フィクスチャーの作成)
  - [書籍アプリモデルを管理サイトに追加](#書籍アプリモデルを管理サイトに追加)
  - [まとめ](#まとめ)

## 書籍（`books`）アプリの追加

書籍（`books`）アプリを次の通り作成します。

```bash
python manage.py startapp books
```

次に、プロジェクト設定ファイルの`INSTALLED_APP`に次を追加して、書籍アプリをプロジェクトに追加します。

```python
# ./book_management/settings.py
  INSTALLED_APPS = [
      "django.contrib.admin",
      "django.contrib.auth",
      "django.contrib.contenttypes",
      "django.contrib.sessions",
      "django.contrib.messages",
      "django.contrib.staticfiles",
      "debug_toolbar",
      "divisions.apps.DivisionsConfig",
+     "books.apps.BooksConfig",
  ]
```

次に、管理サイトに表示する書籍アプリの名前を設定します。

```python
# ./books/apps.py
  class BooksConfig(AppConfig):
      default_auto_field = "django.db.models.BigAutoField"
      name = "books"
+     verbose_name = "書籍アプリ"
```

最後に、`python manage.py shell`でインタラクティブコンソールが正常に起動することを確認した後、次の通り変更をリポジトリにコミットします。

```bash
git add --all
git commit -m '書籍アプリを追加'
```

> 9c8370d (tag: 028-add-books-app)

## 書籍アプリモデルの実装

### 作成日時及び更新日時モデルフィールドの共有

作成日時及び更新日時モデルフィールドは、書籍アプリのすべてのモデルに持たせます。
書籍アプリのモデルを実装する前に、作成日時及び更新日時フィールドを抽象基本モデルに抽出して、それぞれのモデルで共有できるようにします。

`./core`ディレクトリを作成して、`.core/__init__.py`と`./core/models.py`を作成します。

```bash
mkdir core
touch core/__init__.py
code core/models.py
```

`touch`コマンドは、指定したファイルが存在しない場合はそのファイルを作成して、存在する場合はそのファイルのタイムスタンプを更新します。

`./core/models.py`に次を追加します。

```python
# ./core/models.py
from django.db import models


class TimestampModel(models.Model):
    """作成日時と更新日時をモデルフィールドに持つ抽象基本モデル"""

    # 作成日時
    created_at = models.DateTimeField("作成日時", auto_now_add=True)
    # 更新日時
    updated_at = models.DateTimeField("更新日時", auto_now=True)

    class Meta:
        abstract = True
```

`abstract = True`は、`TimestampModel`が[抽象基本モデル](https://docs.djangoproject.com/en/4.2/topics/db/models/#abstract-base-classes)であることをDjangoに伝えます。
これにより、Djangoは、`TimestampModel`をマイグレーションの対象にせず、`TimestampModel`に対応するテーブルをデータベースに作成しません。

次に、部署モデルを次の通り変更します。

```python
# ./divisions/models.py
  from django.db import models

+ from core.models import TimestampModel


- class Division(models.Model):
+ class Division(TimestampModel):
      """部署モデル

      django.db.models.Modelを継承したDivisionモデルを定義します。
      """

      # 部署コード
      code = models.CharField("部署コード", primary_key=True, max_length=2)
      # 部署名
      name = models.CharField("部署名", max_length=80)
-     # 作成日時
-     created_at = models.DateTimeField("作成日時", auto_now_add=True)
-     # 更新日時
-     updated_at = models.DateTimeField("更新日時", auto_now=True)

      class Meta:
```

開発サーバーを起動して、部署一覧、詳細、登録、更新及び削除ページで、以前と同様に部署モデルインスタンスを操作できるか確認してください。
部署モデルインスタンスを操作できた場合は、次の通り変更をリポジトリにコミットします。

```bash
git add ./divisions/
git add ./core/
git commit -m '作成日時及び更新日時モデルフィールドを共有'
```

> 07aa769 (tag: 029-timestamp-model)

### 書籍分類モデルの実装

書籍分類モデル（`Classification`）を次の通り実装します。
書籍分類モデルの実装は、部署モデルとほとんど同じです。

```python
# ./books/models.py
from django.db import models

from core.models import TimestampModel


class Classification(TimestampModel):
    """書籍分類モデル"""

    # 書籍分類コード
    code = models.CharField("書籍分類コード", primary_key=True, max_length=3)
    # 書籍分類名
    name = models.CharField("書籍分類名", max_length=80)

    class Meta:
        db_table = "classifications"
        verbose_name = verbose_name_plural = "書籍分類"
        ordering = ("code",)

    def __str__(self) -> str:
        """書籍分類モデルインスタンス名を返却する。

        Returns:
            書籍分類モデルインスタンス名。
        """
        return self.name

    def full_name(self) -> str:
        """書籍分類モデルインスタンスのフルネームを返却する。

        Returns:
            書籍分類モデルのフルネーム。
        """
        return f"{self.code}: {self.name}"
```

書籍分類モデルを実装したらマイグレーションを実行します。

```bash
python manage.py makemigrations
# マイグレーションの内容を確認したい場合は次を実行
# python manage.py sqlmigrate books 0001
python manage.py migrate
```

書籍分類モデルをマイグレーションしたら、次の通り変更をリポジトリにコミットします。

```bash
git add ./books/
git commit -m '書籍分類モデルを実装'
```

> 7ba6926 (tag: 030-implement-classification-model)

### 書籍分類詳細モデルの実装

書籍分類詳細モデル（`ClassificationDetail`）を次の通り実装します。
書籍分類詳細モデルは、書籍分類モデルを参照（書籍分類詳細モデルは書籍分類モデルを外部参照）するため、Pythonの仕様上、書作分類モデルより下の行に実装する必要があります。

```python
# ./books/models.py
class ClassificationDetail(TimestampModel):
    """書籍分類詳細モデル"""

    # 書籍分類詳細コード
    code = models.CharField("書籍分類詳細コード", primary_key=True, max_length=3)
    # 書籍分類
    classification = models.ForeignKey(
        Classification,
        db_column="classification_code",
        on_delete=models.PROTECT,
        related_name="classification_details",
        verbose_name="書籍分類詳細",
    )
    # 書籍分類詳細名
    name = models.CharField("書籍分類詳細名", max_length=80)

    class Meta:
        db_table = "classification_details"
        verbose_name = verbose_name_plural = "書籍分類詳細"
        ordering = ("code",)

    def __str__(self) -> str:
        """書籍分類詳細モデルインスタンス名を返却する。

        Returns:
            書籍分類詳細モデルインスタンス名。
        """
        return self.name

    def full_name(self) -> str:
        """書籍分類詳細モデルインスタンスのフルネームを返却する。

        Returns:
            書籍分類詳細モデルのフルネーム。
        """
        return f"{self.code}: {self.name}"
```

`ForeignKey`モデルフィールドは、1対多の関連を表現するために他または自身のモデルを参照する外部キーをモデルに追加します。
書籍分類詳細モデルインスタンスから、次の通り書籍分類モデルインスタンスにアクセスできます。

> **外部参照、外部キーとは**
>
> 外部キーとは、データベースにおいて、別のテーブルの主キーを参照するために使用される制約です。
> つまり、外部キーはあるテーブルのカラムにある値が、別のテーブルの主キーとして存在することを保証します。
> なお、別テーブルの主キーを参照することを外部参照と呼びます。
> 外部キー制約は、データベースの整合性を保つために非常に重要です。
> 例えば、注文と商品という2つのテーブルがある場合、注文のテーブルには商品のIDを記録する必要があります。
> この場合、注文テーブルには、商品テーブルの主キーを参照する外部キー制約を追加する必要があります。
> 外部キー制約を設定することにより、データベースに不整合なデータが入力されることを防ぐことができます。
> 例えば、注文のテーブルに、存在しない商品のIDを記録できなくなります。
> 外部キー制約は、データベースのパフォーマンスにも影響を与えます。
> 外部キー制約を追加すると、テーブル間の関係性が明確になり、データベースの最適化が容易になります。

```python
# 書籍分類詳細モデルインスタンスを構築
classification_detail = ClassificationDetail(...)
# 書籍分類詳細モデルインスタンスの書籍分類モデルインスタンスを取得
classification = classification_detail.classification
# 上記書籍分類モデルインスタンスのcodeとname属性を出力
print(f"{classification.code}: {classification.name}")
```

書籍分類モデルフィールドに対応するテーブルのカラムには、書籍分類コードが記録されます。
このため、テーブルに作成されるカラムの名前を`db_column`で`classification_code`を指定しています。
なお、`db_column`でカラム名を指定しない場合、カラム名は`classification`になります。

また、`on_delete = models.PROTECT`を指定することで、書籍分類詳細が参照する書籍分類が削除されないように保護します。
`on_delete`に設定する主な値を次に示します。
なお、`on_delete`はデータベースに由来するため、次の説明でレコードはモデルインスタンス、カラムはモデルフィールドに対応します。

| `on_delete`に設定できる値 | 説明 |
| --------------------- | --- |
| `CASCADE`             | 参照先のレコードが削除されたとき、そのレコードを参照するすべてのレコードが削除されます。 |
| `PROTECT`             | 参照先のレコードが削除されるとき、そのレコードを参照するレコードが存在する場合、[ProtectedError](https://docs.djangoproject.com/en/4.2/ref/exceptions/#django.db.models.ProtectedError)を発生させて、参照先のレコードを削除しません。 |
| `RESTRICT`            | `PROTECT`とほとんど同じですが、[RestrictedError](https://docs.djangoproject.com/en/4.2/ref/exceptions/#django.db.models.RestrictedError)が発生します。しかし、`PROTECTED`と異なり、同じトランザクション内で削除される参照先のレコードを参照するすべてのレコードを削除したり、参照先を変更したりする場合、`RestrictedError`は発生せず、参照先のレコードが削除されます。 |
| `SET_NULL`            | 参照先のレコードが削除されたとき、それを参照しているすべてのレコードのカラムを`NULL`に設定します。`SET_NULL`を指定する場合、その`ForeignKey`は`null = True`を設定する必要があります。 |
| `SET_DEFAULT`         | `SET_NULL`と異なりモデルフィールドに設定したデフォルト値を設定します。 |

`on_delete`に設定できる値の詳細は、[ここ](https://docs.djangoproject.com/en/4.2/ref/models/fields/#django.db.models.ForeignKey.on_delete)を参照してください。

> Djangoバージョン4.2時点では、データベースの外部キーに`ON_DELETE`を設定せず、Djangoはデータベースが実施する`ON_DELETE`の処理を模倣します。

`related_name="classification_details"`は、参照先から参照されるモデルインスタンスを逆参照するときの名前を指定しています。
名前を指定しなくても、Djangoは`<model_name>_set`で逆参照できますが、ここでは明示的に名前を付けています。
ただし、モデルで2つ以上の`ForeignKey`モデルフィールドが同じモデルを参照するとき、`<model_name>_set`が重複するため、明示的に`related_name`で異なる名前を付ける必要があります。
次の通り、`related_name`により、書籍分類モデルインスタンスから、書籍分類詳細モデルインスタンスを得られます。

```python
classification = Classification.objects.get(code="000")
for classification_detail in classification.classification_details.all():
    print(classification_detail)
```

> 上記コードは、同じ書籍分類を持つ書籍分類詳細を、`classification_details`によって書籍分類から逆参照しています。

書籍分類詳細モデルを実装後、マイグレーションを実行します。

```bash
python manage.py makemigrations
# マイグレーションの内容を確認したい場合は次を実行
# python manage.py sqlmigrate books 0002
python manage.py migrate
```

書籍分類詳細モデルをマイグレーションしたら、次の通り変更をリポジトリにコミットします。

```bash
git add ./books/
git commit -m '書籍分類詳細モデルを実装'
```

> 7c124ad (tag: 031-implement-classification-detail-model)

### 書籍モデルの実装

#### プライマリーキーが指定されていないモデル

Djangoは、プライマリーキーが指定されていないモデル（`primary_key = True`を指定したモデルフィールドがないモデル）に対して、プライマリーキーとなる[AutoField](https://docs.djangoproject.com/en/4.2/ref/models/fields/#autofield)または[BigAutoField](https://docs.djangoproject.com/en/4.2/ref/models/fields/#autofield)型の`id`モデルフィールドを自動的に追加します。
`AutoField`及び`BigAutoField`型のモデルフィールドの値は、名前の通りモデルインスタンスが登録されるときに自動的に整数で採番されます。
DBMSに`PostgreSQL`を使用している場合、採番には`シーケンス`が利用されます。

モデルのプライマリーキーの型が整数型の場合、部署詳細ページのURLのように決め打ちでページにアクセスできるようになるなど、セキュリティー上の問題が発生する可能性があります。
整数型のプライマリーキーの採用は、アプリケーションまたはモデルに必要とされる要件から判断してください。

> **Python3の整数型**
>
> C、C++、Rustなど、普通のプログラミング言語では、整数型が扱える範囲に制限があります。
> しかし、Pythonにおいてメモリに空きがある限り（おそらく連続したメモリ領域を確保できる限り）上限または下限なく整数を扱えます。

#### UUIDとULID

ランダムな識別子として[UUID](https://ja.wikipedia.org/wiki/UUID)と[ULID](https://github.com/ulid/spec)があります。
これらの特徴を次に示します。

- 分散システムで生成したときに、重複や偶然の一致が発生しないように設計されています。
- 128ビットの数値で表現されます。
- UUIDは16進数法で36文字の文字列で、ULIDは26文字の文字列で表現されます。
- UUIDには時間的順序がありませんが、ULIDには時間的順序があります。

<!-- cspell: disable -->
> UUIDを16進数法で表現した文字列は、`eaae952d-ab1b-4671-8076-e79980633f87`のようになります。
> ULIDを16進数法で表現した文字列は、`01GYV46C5KXWDRKMB1WR3TW6RK`のようになります。
> また、UUIDの重複に関しては、[UUIDを重複させるにはどれだけ時間がかかるのか試してみた](https://zenn.dev/kiyocy24/articles/uuid-duplicate-time)を参照してください。
<!-- cspell: enable -->

本Webアプリケーションでは、書籍を登録した順番で管理したいため、書籍モデルのプライマリーキーの型に`ULID`を採用します。
なお、書籍モデルのプライマリーキーの値は、[python-ulid](https://python-ulid.readthedocs.io/en/latest/)パッケージを利用して採番します。

#### ULIDFieldモデルフィールドの定義

Djangoは、`UUID`を記録する[UUIDField](https://docs.djangoproject.com/en/4.2/ref/models/fields/#uuidfield)を提供しますが、`ULID`を記録するモデルフィールドを提供していません。

そこで、`python-ulid`パッケージをインストールして、`ULID`を記録する`ULIDField`を`./core/models.py`に次の通り実装します。

Djangoで独自のモデルフィールドを実装する場合は、[独自のモデルフィールドを作成する方法](https://docs.djangoproject.com/en/4.2/ref/models/fields/#charfield)を参照してください。

```bash
pip install python-ulid
```

```python
# ./core/models.py
+ class ULIDField(models.CharField):
+     """ULIDモデルフィールド"""
+
+     def __init__(self, *args, **kwargs) -> None:
+         kwargs["max_length"] = 26
+         super().__init__(*args, **kwargs)
+
+     def db_type(self, connection) -> str:
+         return "char(26)"
```

`ULIDField`は、[CharField](https://docs.djangoproject.com/en/4.2/ref/models/fields/#charfield)を継承しており、文字列の最大長さを`ULID`の16進数法表現の文字数である26文字に制限しています。

`ULIDField`の実装を次の通りリポジトリにコミットします。

```bash
pip freeze > requirements.txt
git add ./core/models.py
git add requirements.txt
git commit -m 'ULIDFieldを実装'
```

> de3162a (tag: 032-implement-ulid-model-field)

### 書籍モデルの定義

書籍モデル（`Book`）を次の通り実装します。
書籍モデルは、書籍分類詳細モデルを外部参照するため、書籍分類詳細モデルと同様に、書作分類詳細モデルより下の行に実装する必要があります。

```python
# ./books/models.py
+ import ulid
+
  from django.db import models

- from core.models import TimestampModel
+ from core.models import TimestampModel, ULIDField

  (...省略...)

+ class Book(TimestampModel):
+     """書籍モデル"""
+
+     # 書籍ID
+     id = ULIDField("書籍ID", primary_key=True, editable=False, default=ulid.ULID)
+     # タイトル
+     title = models.CharField("タイトル", max_length=120)
+     # 書籍分類詳細
+     classification_detail = models.ForeignKey(
+         ClassificationDetail,
+         db_column="classification_detail_code",
+         on_delete=models.PROTECT,
+         related_name="classification_detail_books",
+         verbose_name="書籍分類詳細",
+     )
+     # 著者または訳者
+     authors = models.TextField("著者または訳者", null=True, blank=True)
+     # ISBN
+     isbn = models.CharField("ISBN", max_length=40, null=True, blank=True)
+     # 出版社
+     publisher = models.CharField("出版社", max_length=80, null=True, blank=True)
+     # 発行日
+     published_at = models.DateField("発行日", null=True, blank=True)
+     # 管理部署
+     division = models.ForeignKey(
+         "divisions.Division",
+         db_column="division_code",
+         on_delete=models.PROTECT,
+         related_name="books",
+         verbose_name="管理部署",
+     )
+     # 廃棄済み
+     disposed = models.BooleanField("廃棄済み", default=False)
+     # 廃棄日
+     disposed_at = models.DateField("廃棄日", null=True, blank=True)
+
+     class Meta:
+         db_table = "books"
+         verbose_name = verbose_name_plural = "書籍"
+         ordering = ("id",)
+
+     def __str__(self) -> str:
+         """書籍のタイトルを返却する。
+
+         Returns:
+             書籍のタイトル。
+         """
+         return self.title
```

書籍モデルをマイグレーションして、次の通り変更をリポジトリにコミットします。

```bash
python manage.py makemigrations
# python manage.py sqlmigrate books 0003
python manage.py migrate
git add ./books/.
git commit -m '書籍モデルを実装'
```

> 5f1d241 (tag: 033-implement-book-model)

## 書籍、書籍分類及び書籍分類詳細フィクスチャーの作成

書籍、書籍分類及び書籍分類詳細フィクスチャーを`./books/fixtures/`ディレクトリに作成します。

```bash
mkdir ./books/fixtures
```

なお、書籍分類及び書籍分類詳細は、[日本十進分類法新訂10版](https://ja.wikipedia.org/wiki/%E6%97%A5%E6%9C%AC%E5%8D%81%E9%80%B2%E5%88%86%E9%A1%9E%E6%B3%95)の`類目表（第1次区分表）`及び`綱目表（第2次区分表）`に対応します。

書籍、書籍分類及び書籍詳細フィクスチャーは、[ここ](https://github.com/xjr1300/book_management/tree/main/books/fixtures)にあるので、ダウンロードして`./books/fixtures`ディレクトリに保存してください。

書籍、書籍分類及び書籍詳細フィクスチャーの例を次に示します。

<!-- cspell: disable -->
```yaml
# ./books/fixtures/books.yaml
# 書籍フィクスチャー
- model: books.book
  pk: 01GYV46C5KXWDRKMB1WR3TW6RK
  fields:
    created_at: 2023-04-25 00:00:00+00:00
    updated_at: 2023-04-25 00:00:00+00:00
    title: Fluent Python
    classification_detail: "000"
    authors: "（著）Luciano Ramalho\r\n（監修）豊沢　聡、桑井　博之\r\n（訳）梶原　玲子"
    isbn: ISBN978-4-87311-817-8
    publisher: オライリー・ジャパン
    published_at: 2017-10-11
    division: "58"
    disposed: false
    disposed_at: null
- model: books.book
  pk: 01GYV585X4JNDCSVA3BY93KN54
  fields:
    created_at: 2023-04-25 00:00:00+00:00
    updated_at: 2023-04-25 00:00:00+00:00
    title: プログラミングRust
    classification_detail: "000"
    authors: "（著）Jim Blandy、Jason Orendorff\r\n（訳）中田　秀基"
    isbn: ISBN978-4-87311-855-0
    publisher: オライリー・ジャパン
    published_at: 2018-08-08
    division: "58"
    disposed: false
    disposed_at: null
- model: books.book
  pk: 01GYZYA8NS02G8NWCFWBZ6KS9T
  fields:
    title: プロを目指す人のためのTypeScript入門
    classification_detail: "000"
    authors: 鈴木  僚太
    isbn: ISBN978-4-297-12747-3
    publisher: 技術評論社
    published_at: 2022-05-05
    division: "58"
    disposed: false
    disposed_at: null
    created_at: 2023-04-25 00:00:00+00:00
    updated_at: 2023-04-25 00:00:00+00:00
```
<!-- cspell: enable -->

```yaml
# ./books/fixtures/classifications.yaml
# 書籍分類フィクスチャー
- model: books.classification
  pk: "000"
  fields:
    name: 総記
    created_at: 2023-04-24 00:00:00.000000+00:00
    updated_at: 2023-04-24 00:00:00.000000+00:00
- model: books.classification
  pk: "100"
  fields:
    name: 哲学
    created_at: 2023-04-24 00:00:00.000000+00:00
    updated_at: 2023-04-24 00:00:00.000000+00:00
```

```yaml
# ./books/fixtures/classification_details.yaml
# 書籍分類詳細フィクスチャー
- model: books.classificationdetail
  pk: "000"
  fields:
    classification: "000"
    name: 総記
    created_at: 2023-04-24 00:00:00.000000+00:00
    updated_at: 2023-04-24 00:00:00.000000+00:00
- model: books.classificationdetail
  pk: "010"
  fields:
    classification: "000"
    name: 図書館、図書館情報学
    created_at: 2023-04-24 00:00:00.000000+00:00
    updated_at: 2023-04-24 00:00:00.000000+00:00
```

書籍分類、書籍分類詳細、書籍の順番でフィクスチャーをデータベースに登録します。

> 上記と異なる順番でフィクスチャーをデータベースに登録することを試みると、外部キーエラーが発生します。

フィクスチャーをデータベースに登録後、次の通り変更をリポジトリにコミットします。

```bash
python manage.py loaddata --format=yaml ./books/fixtures/classifications.yaml
python manage.py loaddata --format=yaml ./books/fixtures/classification_details.yaml
python manage.py loaddata --format=yaml ./books/fixtures/books.yaml
git add ./books/fixtures
git commit -m '書籍分類及び書籍分類詳細フィクスチャーを作成'
```

> 4aba444 (tag: 034-add-books-fixtures)

## 書籍アプリモデルを管理サイトに追加

書籍アプリモデルを管理サイトに登録するために、`./books/admin.py`を次で置き換えます。

```python
# ./books/admin.py
from django.contrib import admin

from .models import Book, Classification, ClassificationDetail


class ClassificationAdmin(admin.ModelAdmin):
    """書籍分類モデルアドミン"""

    list_display = (
        "code",
        "name",
        "created_at",
        "updated_at",
    )


class ClassificationDetailAdmin(admin.ModelAdmin):
    """書籍分類詳細モデルアドミン"""

    list_display = (
        "code",
        "classification",
        "name",
        "created_at",
        "updated_at",
    )
    list_filter = ("classification",)


class BookAdmin(admin.ModelAdmin):
    """書籍モデルアドミン"""

    list_display = (
        "title",
        "classification_detail",
        "authors",
        "published_at",
        "division",
    )
    list_filter = (
        "classification_detail__classification",
        "division",
    )
    search_fields = (
        "title",
        "classification_detail__classification__name",
        "classification_detail__name",
        "authors",
        "division__name",
    )


admin.site.register(Classification, ClassificationAdmin)
admin.site.register(ClassificationDetail, ClassificationDetailAdmin)
admin.site.register(Book, BookAdmin)
```

書籍分類モデルアドミン（`ClassificationAdmin`）は、部署モデルアドミン（`DivisionAdmin`）と同様です。

書籍分類詳細モデルアドミン（`ClassificationDetailAdmin`）は、`list_filter = ("classification",)`を設定しています。
この設定により、管理サイトの書籍分類詳細一覧ページ（`http://localhost:8000/admin/books/classificationdetail/`）は、書籍分類でフィルタして書籍分類詳細を表示できます。

書籍モデルアドミン（`BookAdmin`）も書籍分類詳細モデルアドミンと同様な設定をしており、管理サイトの書籍一覧ページ（`http://localhost:8000/admin/books/book/`）で書籍分類または管理部署でフィルタして書籍を表示できます。

また、管理サイトの書籍一覧ページは、`検索ボックス`に入力した文字列を、`search_fields`に指定したモデルフィールドの値に含む書籍モデルインスタンスを検索できます。

次に、`search_field`に指定した項目と、検索対象を示します。

- title: 書籍のタイトル
- classification_detail__classification__name: 書籍分類詳細の書籍分類の名前
- classification_detail__name: 書籍分類詳細の名前
- authors: 著者または訳者
- division__name: 管理部署の名前

> `list_filter`や`search_field`などでは、属性などを`.（ドット）`ではなく`__（アンダーバー2つ）`で指定します。
> これは、クエリAPIでも同賞です。
>
> ```python
> classification_details = ClassificationDetail.objects.filter(classification__code="000")
> classification_details
> # <QuerySet [<ClassificationDetail: 総記>, <ClassificationDetail: 図書館、図書館情報学>, <ClassificationDetail: 図書、書誌学>, <ClassificationDetail: 百科事典、用語索引>, <ClassificationDetail: 一般論文集、一般講演集、雑著>, <ClassificationDetail: 逐次刊行物、一般年鑑>, <ClassificationDetail: 団体、博物館>, <ClassificationDetail: ジャーナリズム、新聞>, <ClassificationDetail: 書、全集、選集>, <ClassificationDetail: 貴重書、郷土資料、その他の特別コレクション>]>
> ```

開発サーバーを起動して、書籍アプリのモデルが管理サイトに表示されているか確認してください。
書籍アプリのモデルが管理サイトに表示されていることを確認できたら、次の通り変更をリポジトリにコミットします。

```bash
git add ./books/admin.py
git commit -m '書籍アプリのモデルを管理サイトに登録'
```

> f69a5dd (tag: 035-register-books-app-models-to-admin-site)

## まとめ

本章では、書籍アプリのモデルを定義して、それらを管理サイトに追加しました。

次の章では、書籍アプリページを実装します。
