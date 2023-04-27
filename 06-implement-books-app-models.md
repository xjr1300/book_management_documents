# 書籍アプリのモデルの実装

- [書籍アプリのモデルの実装](#書籍アプリのモデルの実装)
  - [書籍（`books`）アプリの追加](#書籍booksアプリの追加)
  - [書籍アプリモデルの実装](#書籍アプリモデルの実装)
    - [作成日時及び更新日時モデルフィールドのリファクタリング](#作成日時及び更新日時モデルフィールドのリファクタリング)
    - [書籍分類モデルの実装](#書籍分類モデルの実装)
    - [書籍分類詳細モデルの実装](#書籍分類詳細モデルの実装)
    - [書籍モデルの実装](#書籍モデルの実装)
      - [プライマリーキーが指定されていないモデル](#プライマリーキーが指定されていないモデル)
      - [UUIDとULID](#uuidとulid)
      - [ULIDFieldモデルフィールドの定義](#ulidfieldモデルフィールドの定義)
    - [書籍モデルの定義](#書籍モデルの定義)
  - [書籍分類及び書籍分類詳細フィクスチャーの作成](#書籍分類及び書籍分類詳細フィクスチャーの作成)
  - [書籍アプリモデルを管理サイトに表示](#書籍アプリモデルを管理サイトに表示)
  - [まとめ](#まとめ)

## 書籍（`books`）アプリの追加

書籍（`books`）アプリを次の通り作成します。

```bash
python manage.py startapp books
```

次に、プロジェクト設定ファイルの`INSTALLED_APP`に書籍アプリを追加して、書籍アプリをプロジェクトに追加します。

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

そして、`python manage.py shell`でインタラクティブシェルが正常に起動することを確認した後、書籍アプリの追加をリポジトリにコミットします。

```bash
git add --all
git commit -m '書籍アプリを追加'
```

> commit 887f7fd566fc2eb0588acc61aeb3847407b75067

## 書籍アプリモデルの実装

### 作成日時及び更新日時モデルフィールドのリファクタリング

作成日時及び更新日時モデルフィールドは、本プロジェクトのすべてのモデルが持つフィールドにする予定です。

実際に書籍アプリモデルを実装する前に、作成日時及び更新日時フィールドをリファクタリングします。

`./core`ディレクトリを作成して、`.core/__init__.py`と`./core/models.py`を作成します。

```bash
mkdir core
touch core/__init__.py
code core/models.py
```

> `touch`コマンドは、指定したファイルが存在しない場合はそのファイルを作成して、存在する場合はそのファイルのタイムスタンプを更新します。

`./core/models.py`に次を追加します。

```python
# ./core/models.py
from django.db import models


class TimestampModel(models.Model):
    """作成日時と更新日時をモデルフィールドに持つモデル"""

    # 作成日時
    created_at = models.DateTimeField("作成日時", auto_now_add=True)
    # 更新日時
    updated_at = models.DateTimeField("更新日時", auto_now=True)

    class Meta:
        abstract = True
```

`abstract = True`は、`TimestampModel`が[抽象基本クラスのモデル](https://docs.djangoproject.com/en/4.2/topics/db/models/#abstract-base-classes)であることをDjangoに伝えて、`TimestampModel`に対応するテーブルが作成されないようにします。

次に、部署モデルを次の通り変更します。

```python
# ./divisions/models.py
  from django.db import models

  from core.models import TimestampModel


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

開発サーバーを起動して、部署一覧、詳細、登録、更新及び削除ページで、以前と同様に部署を操作できるか確認してください。
部署を操作できた場合は、変更をリポジトリにコミットします。

```bash
git add --all
git commit -m '作成日時及び更新日時モデルフィールドをリファクタリング'
```

> commit 28e2eafa6ba84097224e2c9e57af7cb9441df2c8

### 書籍分類モデルの実装

書籍分類モデル（`Classification`）の実装を`./books/models.py`に次の通り実装します。
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

書籍分類モデルのマイグレーションに成功したら、変更をリポジトリにコミットします。

```bash
git add --all
git commit -m '書籍分類モデルを実装'
```

> commit 873f025c5cc95812aa4eceb11ddb4709951be8ec

### 書籍分類詳細モデルの実装

書籍分類詳細モデル（`ClassificationDetail`）の実装を`./books/models.py`に次の通り実装します。
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
        return f"{self.code}: {self.name.name}"
```

`ForeignKey`モデルフィールドは、1対多の関連表現するために他または自身のモデルを参照する外部キーをモデルに追加します。

書籍分類モデルインスタンスから、書籍分類モデルインスタンスにアクセスできます。

```python
# 書籍分類詳細モデルインスタンスを構築
classification_detail = ClassificationDetail(...)
# 書籍分類詳細モデルインスタンスの書籍分類モデルインスタンスを取得
classification = classification_detail.classification
# 上記書籍分類モデルインスタンスのcodeとname属性を出力
print(f"{classification.code}: {classification.name}")
```

部署分類モデルフィールドに対応するテーブルのカラムには、部署分類コードが記録されます。
このため、テーブルに作成されるカラムの名前を`db_column`で`classification_code`を指定してます。
なお、`db_column`でカラム名を指定しない場合、カラム名は`classification`になります。

また、`on_delete = models.PROTECT`を指定することで、部署分類詳細が参照する部署分類が削除されないように保護します。
`on_delete`に設定する主な値を次に示します。
`on_delete`はデータベースに由来するため、`レコード`を`インスタンス`、`カラム`をモデルフィールドに対応します。

| `on_delete`に設定できる値 | 説明 |
| --- | --- |
| `CASCADE` | 参照先のレコードが削除されたとき、そのレコードを参照するすべてのレコードが削除されます。 |
| `PROTECT` | 参照先のレコードが削除されようとしたとき、そのレコードを参照するレコードが存在する場合、[ProtectedError](https://docs.djangoproject.com/en/4.2/ref/exceptions/#django.db.models.ProtectedError)を発生させて、参照先のレコードが削除されないように保護します。 |
| `RESTRICT` | `PROTECT`とほとんど同じですが、[RestrictedError](https://docs.djangoproject.com/en/4.2/ref/exceptions/#django.db.models.RestrictedError)が発生します。しかし、`PROTECTED`と異なり、同じトランザクション内で、削除される参照先のレコードを参照するすべてのレコードを削除したり、参照先を変更したりする場合、`RestrictedError`は発生せず、参照先のレコードが削除されます。 |
| `SET_NULL` | 参照先のレコードが削除されたとき、それを参照しているすべてのレコードのカラムを`NULL`に設定します。`SET_NULL`を指定する場合、その`ForeignKey`は`null = True`を設定する必要があります。 |
| `SET_DEFAULT | `SET_NULL`と異なりモデルフィールドに設定したデフォルト値を設定します。 |

`on_delete`に設定できる値は、[ここ](https://docs.djangoproject.com/en/4.2/ref/models/fields/#django.db.models.ForeignKey.on_delete)を参照してください。

> Djangoバージョン4.2時点では、データベースの外部キーに`ON_DELETE`を設定せず、データベースの`ON_DELETE`操作を模倣します。

`related_name="classification_details"`は、参照先から参照されるモデルインスタンスを逆参照するときの名前を指定しています。
名前を指定しなくても、Djangoは`<model_name>_set`で逆参照できるようにしますが、明示的に名前をつけています。
ただし、モデルが2つ以上`ForeignKey`モデルフィールドで同じモデルを参照するとき、上記理由で明示的に`related_name`を指定する必要があります。
次の通り、書籍分類モデルインスタンスから、書籍分類詳細モデルインスタンスを得られます。

```python
classification = Classification.objects.get(code="000")
for classification_details in classification.classification_details.all():
    # 書籍分類詳細を操作
```

書籍分類詳細モデルを実装したらマイグレーションを実行します。

```bash
python manage.py makemigrations
# マイグレーションの内容を確認したい場合は次を実行
# python manage.py sqlmigrate books 0002
python manage.py migrate
```

書籍分類詳細モデルのマイグレーションに成功したら、変更をリポジトリにコミットします。

```bash
git add --all
git commit -m '書籍分類詳細モデルを実装'
```

> commit c80d06ea46b20f34228fca619111dcdf4df90b75

### 書籍モデルの実装

書籍モデル（`Book`）の実装を`./books/models.py`に次の通り実装します。
書籍モデルは、書籍分類詳細モデルを外部参照するため、書籍分類詳細モデルと同様に、書作分類詳細モデルより下の行に実装する必要があります。

#### プライマリーキーが指定されていないモデル

Djangoは、プライマリーキーが指定されていないモデル（`primary_key = True`を指定したフィールドがないモデル）があった場合、自動的にプライマリーキーとなる[AutoField](https://docs.djangoproject.com/en/4.2/ref/models/fields/#autofield)または[BigAutoField](https://docs.djangoproject.com/en/4.2/ref/models/fields/#autofield)型の`id`モデルフィールドを追加します。
これらのモデルフィールド型は、名前の通りモデルインスタンスが登録されるときに、自動的に整数を採番します。
これらの値は、データベースの`シーケンス`によって採番されます。

モデルのプライマリーキーに整数を使用した場合、部署詳細ページにアクセスするURLのように、URLを決め打ちでアクセスできるようになるなど、セキュリティー上の問題が発生するか可能性があります。

> ログイン済みユーザーのみアクセス可能なページにするなどの対策をすれば別です。

#### UUIDとULID

ランダムな識別子として[UUID](https://ja.wikipedia.org/wiki/UUID)と`ULID`があります。
これら特徴を次に示します。

- 分散システムで生成したときに、重複や偶然の一致が発生しないように設計されています。
- 128ビットの数値で表現されます。
- UUIDは16進数法で36文字の文字列で表現されますが、ULIDは26文字の文字列で表現されます。
- UUIDには時間的順序がありませんが、ULIDには時間的順序があります。

> UUIDは、16進数法で表現した文字列は、`eaae952d-ab1b-4671-8076-e79980633f87`のようになります。
> ULIDは、16進数法で表現した文字列は、`01GYV46C5KXWDRKMB1WR3TW6RK`のようになります。
> また、UUIDの重複に関しては、[UUIDを重複させるにはどれだけ時間がかかるのか試してみた](https://zenn.dev/kiyocy24/articles/uuid-duplicate-time)を参照してください。

本Webアプリケーションでは、書籍を登録した順番で管理したいため、書籍モデルのプライマリーキーの型に`ULID`を採用します。

書籍モデルのプライマリーキーを[ULID](https://python-ulid.readthedocs.io/en/latest/)で採番します。

#### ULIDFieldモデルフィールドの定義

Djangoは、`UUID`を記録する[UUIDField](https://docs.djangoproject.com/en/4.2/ref/models/fields/#uuidfield)を提供しますが、`ULID`を記録するモデルフィールドを提供していません。

そこで、`python-ulid`パッケージをインストールして、`ULID`を記録する`ULIDField`を`./core/models.py`に次の通り実装します。

> [How to create custom model fields](https://docs.djangoproject.com/en/4.2/ref/models/fields/#charfield)

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

`ULIDField`は、[CharField](https://docs.djangoproject.com/en/4.2/ref/models/fields/#charfield)から派生して、文字列の最大長さに26文字（`ULID`の16進数法表現）を設定しています。

`ULIDField`の実装をリポジトリにコミットします。

```bash
pip freeze > requirements.txt
git add ./core/models.py
git add requirements.txt
git commit -m 'ULIDFieldを実装'
```

> commit 0812c762d20d9e835082973676c7339cbf021a46

### 書籍モデルの定義

書籍モデルを`./books/models.py`に次の通り定義します。

```python
# ./books/models.py
+ import ulid
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
+     # 管理部署コード
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

書籍モデルをマイグレーションして変更をリポジトリにコミットします。

```bash
python manage.py makemigrations
# python manage.py sqlmigrate books 0003
python manage.py migrate
git add books/.
git commit -m '書籍モデルを実装'
```

> commit 47ae27400ce56d18230b6b319d7e59a10a311d20

## 書籍分類及び書籍分類詳細フィクスチャーの作成

書籍分類及び書籍分類詳細フィクスチャーを`./books/fixtures/`ディレクトリに作成ます。
なお、書籍分類及び書籍分類詳細は、[日本十進分類法新訂10版](https://ja.wikipedia.org/wiki/%E6%97%A5%E6%9C%AC%E5%8D%81%E9%80%B2%E5%88%86%E9%A1%9E%E6%B3%95)の`類目表（第1次区分表）`及び`綱目表（第2次区分表）`に対応します。

書籍分類及び書籍詳細フィクスチャーは、[ここ](https://github.com/xjr1300/book_management/tree/main/books/fixtures)にあります。

書籍分類及び書籍詳細フィクスチャーの例を次に示します。

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

書籍分類及び書籍分類詳細フィクスチャーを作成したら、データベースに登録後、変更をリポジトリにコミットします。

```bash
python manage.py loaddata --format=yaml books/fixtures/classifications.yaml
python manage.py loaddata --format=yaml books/fixtures/classification_details.yaml
git add books/fixtures
git commit -m '書籍分類及び書籍分類詳細フィクスチャーを作成'
```

> commit 6f88217924a693386036878b8d5965027d36640d

## 書籍アプリモデルを管理サイトに表示

書籍アプリモデルを管理サイトに表示するために、`./books/admin.py`を次の通り変更します。

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

書籍分類モデルアドミン（`CategoryAdmin`）は、部署モデルアドミン（`DivisionAdmin`）と同様です。

書籍分類詳細モデルアドミン（`CategoryDetailAdmin`）は、`list_filter = ("classification",)`を設定しています。
この設定により、管理サイトの書籍分類詳細一覧ページ（`http://localhost:8000/admin/books/classificationdetail/`）で書籍分類でフィルタして書籍分類を一覧表示できます。

書籍モデルアドミン（`BookAdmin`）も書籍分類詳細と同様な設定をしており、管理サイトの書籍一覧ページで書籍分類及び部署でフィルタして書籍を一覧表示できます。

また、書籍モデルアドミンでは、`search_fields`に次を指定しており、それらを対象に管理サイトの`検索ボックス`に入力された文字列を含む書籍モデルインスタンスを検索できます。

- title: 書籍のタイトル
- classification_detail__classification__name: 書籍分類詳細の書籍分類の名前
- classification_detail__name: 書籍分類詳細の名前
- authors: 著者または訳者
- division__name: 管理部署の名前

> `list_filter`や`search_field`などでは、属性などは`.（ドット）`ではなく`__（アンダーバー2つ）`で指定します。

開発サーバーを起動して、書籍アプリのモデルが表示されるか確認してください。
書籍アプリのモデルが正常に表示さていることを確認できたら、変更をリポジトリにコミットします。

```bash
git add books/admin.py
git commit -m '書籍モデルを管理サイトに表示'
```

> commit a7daeac72e007cfd071cb399403ab24881cf02fa

## まとめ

本章の10章では、書籍アプリのモデルを定義して、それらを管理サイトに登録しました。

次の章では、書籍アプリページを実装します。
