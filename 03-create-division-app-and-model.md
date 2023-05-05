# 部署アプリの追加と部署モデルの実装及び操作

- [部署アプリの追加と部署モデルの実装及び操作](#部署アプリの追加と部署モデルの実装及び操作)
  - [部署アプリの追加](#部署アプリの追加)
  - [試行: 最初のビュー](#試行-最初のビュー)
    - [最初のビューの実装](#最初のビューの実装)
    - [アプリ用のURLconfの定義](#アプリ用のurlconfの定義)
    - [ルートURLconfの設定](#ルートurlconfの設定)
    - [リクエストの処理確認](#リクエストの処理確認)
  - [部署モデルの実装](#部署モデルの実装)
  - [部署アプリの有効化](#部署アプリの有効化)
  - [DjangoクエリセットAPIの操作](#djangoクエリセットapiの操作)
    - [クエリセットAPIを使用した部署モデルの操作](#クエリセットapiを使用した部署モデルの操作)
  - [モデルインスタンスの出力の変更とメソッドの追加](#モデルインスタンスの出力の変更とメソッドの追加)
  - [管理サイト](#管理サイト)
    - [スーパーユーザーの作成](#スーパーユーザーの作成)
  - [管理サイトの有効化](#管理サイトの有効化)
  - [管理サイトのカスタマイズ](#管理サイトのカスタマイズ)
  - [フィクスチャー（Fixture）](#フィクスチャーfixture)
    - [部署モデルインスタンスをフィクスチャーに出力](#部署モデルインスタンスをフィクスチャーに出力)
    - [部署モデルインスタンスをフィクスチャーに追加](#部署モデルインスタンスをフィクスチャーに追加)
    - [部署フィクスチャーで部署モデルインスタンスを登録](#部署フィクスチャーで部署モデルインスタンスを登録)
  - [まとめ](#まとめ)

## 部署アプリの追加

プロジェクトに部署アプリ（`divisions`）を追加するために、vscodeのターミナルに次を入力します。

```bash
python manage.py startapp divisions
```

部署アプリを追加すると、プロジェクトディレクトリに部署アプリ用の`divisions`ディレクトリが作成されます。
`divisions`ディレクトリのコンテンツを次に示します。

```text
book_management/divisions/
 ├── __init__.py
 ├── admin.py
 ├── apps.py
 ├── migrations/
 ├── models.py
 ├── tests.py
 └── views.py
 ```

アプリディレクトリ内のファイルやディレクトリを次に示します。

| 名前           | 説明                                                             |
| ------------- | --------------------------------------------------------------- |
| `__init__.py` | 省略                                                             |
| `admin.py`    | アプリで定義したモデルを管理サイト登録する方法などを定義するファイル             |
| `apps.py`     | アプリケーションの設定を定義するファイル                                  |
| `migrations/` | アプリのモデルをデータベースに反映するマイグレーションファイルを格納するディレクトリ |
| `models.py`   | アプリのモデルを定義するファイル                                        |
| `tests.py`    | アプリの単体テストを実装するファイル                                     |
| `views.py`    | アプリのビューを定義するファイル                                        |

## 試行: 最初のビュー

### 最初のビューの実装

`./divisions/views.py`ファイルを開いて、次のコードでファイルの内容を置き換えます。
この`index`関数は、Djangoにおける`関数ベースドビュー (function-based view、関数ビュー)`と呼ばれ、関数でビューを実装したものです。

> 本チュートリアルでは、`クラスベースドビュー (class-based view、クラスビュー)`でアプリのビューを実装または再実装します。
> クラスビューは、理解することが多少難しいですが、Webアプリケーションに必要な基本的な機能を多く提供するため、関数ビューと比較して、実装するコードを大幅に減らします。

```python
# ./divisions/views.py
from django.http import HttpResponse, HttpRequest


def index(request: HttpRequest) -> HttpResponse:
    return HttpResponse("Hello, world. This is divisions app.")
```

### アプリ用のURLconfの定義

次の通り`./divisions/urls.py`ファイルを作成します。

```bash
code divisions/urls.py
```

その後、`./divisions/urls.py`ファイルに、次のコードを入力します。

```python
# ./divisions/urls.py
from django.urls import path

from . import views

app_name = "divisions"

urlpatterns = [
    path("", views.index, name="index"),
]
```

`./divisions/urls.py`は、`divisions`アプリのURLconfです。
アプリのURLconfは、`Djangoのインストールとプロジェクト構築`で説明したルートURLconfがインクルードするファイルで、リクエストURLを`ディスパッチ`するビューを指定するファイルです。

`./divisions/views.py`である`views`モジュールを`from . import views`でインポートしています。
これは同じディレクトリ(`.`)にある`views.py`ファイルを`views`モジュールとしてインポートすることを指示しています。

また、次で説明する[ルートURLconfの設定](#ルートurlconfの設定)によって、`path`関数が、`http://<site>/divisions/`へのリクエストを`./divisions/views.py`ファイルに定義した`index`ビューが処理するように設定します。

### ルートURLconfの設定

> 本アプリケーションが、`/divisions/`で始まるリクエストを受け取ったときは、すべて部署アプリで処理します。
> よって、ここで実施する内容は、試行ではなく永続です。

ルートURLconfを開いて、次の通り編集します。

```python
# ./book_management/urls.py
  from django.conf import settings
  from django.contrib import admin
  from django.urls import include, path

  urlpatterns = [
+     path("divisions/", include("divisions.urls")),
      path("admin/", admin.site.urls),
  ]
```

この設定により、Webアプリケーションが、`/divisions/`で始まるリクエストURLを受け取ったとき、部署アプリの`./divisions/urls.py`で指定したビューがそのリクエストを処理するようになります。

### リクエストの処理確認

次の通り開発用サーバーを起動して、ブラウザで`http://127.0.0.1:8000/divisions/`にアクセスします。

> 上記で起動した開発サーバーを停止していない場合は、再起動する必要はありません。
> 開発サーバーは、プロジェクトで変更された内容を自動的に反映します。

```bash
python manage.py runserver
```

ブラウザが、`Hello, world. This is divisions app.`を表示した場合、正確にビューにリクエストがディスパッチされています。

試行的に実装したビューが動作したら、次の通り変更をリポジトリにコミットします。

```bash
git add --all
git commit -m '部署アプリを追加'
```

> 9081ca4 (tag: 005-add-divisions-app)

## 部署モデルの実装

`./divisions/models.py`の内容を次ですべて置き換えて、部署モデルを実装します。

```python
# ./divisions/models.py
# django.db.modelsモジュールをインポート
from django.db import models


class Division(models.Model):
    """部署モデル

    django.db.models.Modelを継承したDivisionモデルを定義します。
    """

    # 部署コード
    code = models.CharField(primary_key=True, max_length=2)
    # 部署名
    name = models.CharField(max_length=80)
    # 作成日時
    created_at = models.DateTimeField(auto_now_add=True)
    # 更新日時
    updated_at = models.DateTimeField(auto_now=True)
```

- Djangoのモデルは、直接的または間接的に`django.db.models.Model`を[継承](https://docs.python.org/ja/3.10/tutorial/classes.html#inheritance)する必要があります。
- 部署モデルは、部署コード（`code`）、部署名（`name`）、作成日時（`created_at`）そして更新日時（`updated_at`）`モデルフィールド`を持ちます。
- モデルフィールドはデフォルトでNULLを許容しないため、モデルインスタンスを登録または更新するときに必ず値を設定する必要があります。
- 部署コードモデルフィールドに`primary_key=True`を指定することで、このモデルフィールドがモデルインスタンスを一意に特定する`プライマリーキー`であることをDjangoに伝えます。
- 部署コードモデルフィールドは2文字まで、部署名モデルフィールドは80文字までの文字列を設定できます。
- `django.db.models.DateTimeField`の`auto_now_add`引数は、モデルインスタンスを登録した日時を自動的に記録します。
- `django.db.models.DateTimeField`の`auto_now`引数は、モデルインスタンスを更新した日時を自動的に記録します。

> プライマリーキー（主キー）
>
> モデルインスタンスを一意に特定するためには、モデルインスタンスを一意に特定するための識別子となるプライマリーキーが必要になります。
> 部署モデルでは部署コードがプライマリーキーに指定されており、`58`という部署コードは`ICT開発室`であることを一意に特定します。
> プライマリーキーはデータベースから由来するもので、データベースではテーブルのレコードを一意に特定するために指定または使用されます。
>
> プライマリーキーは、Djangoにおけるモデルインスタンスや、データベースにおけるレコードを特定するために必要であり、プライマリーキーのないモデルやテーブルはほとんど存在しません。
> 存在しても、おそらくモデルインスタンスやレコードを取り扱うことが煩雑になるでしょう。
>
> データベースでは複数のカラムでプライマリーキーを構成して`複合キー`にできますが、Djangoでは複数のモデルフィールドでプライマリーキーを構成できず、1つのモデルフィールドのみプライマリーキーに指定できます。

モデルについては、次を参照してください。

- [モデル](https://docs.djangoproject.com/en/4.2/topics/db/models/)

モデルフィールドについては、次を参照してください。

- [モデルフィールドリファレンス](https://docs.djangoproject.com/en/4.2/ref/models/fields/)

## 部署アプリの有効化

プロジェクトで部署アプリを有効にするために、プリジェクト設定ファイルの`INSTALL_APPS`を次の通り変更します。

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
+     "divisions.apps.DivisionsConfig",
  ]
```

上記により、プロジェクトに部署アプリが存在することをDjangoに伝えます。

部署モデルをデータベースに反映するマイグレーションファイルを作成するために、次のコマンドを実行します。
コマンドの出力結果は先頭に`#`をつけて示します。

```bash
python manage.py makemigrations divisions
# Migrations for 'divisions':
#   divisions/migrations/0001_initial.py
#     - Create model Division
```

出力結果から、Djangoが部署（`Division`）モデルを作成することを伝えています。

また、作成されたマイグレーションファイル`./divisions/migrations/0001_initial.py`のファイルパスから、そのマイグレーションファイルが、`divisions`アプリの`0001`番のマイグレーションであることを識別できます。

作成したマイグレーションが、データベースにどのようなSQL文を発行するか、次のコマンドで確認できます。
コマンドの出力結果は先頭に`#`をつけて示します。

```bash
python manage.py sqlmigrate divisions 0001
# BEGIN;
# --
# -- Create model Division
# --
# CREATE TABLE "divisions_division" ("code" varchar(2) NOT NULL PRIMARY KEY, "name" varchar(80) NOT NULL, "created_at" datetime NOT NULL, "updated_at" datetime NOT NULL);
# COMMIT;
```

`CREATE TABLE`、`varchar`、`NOT NULL`、`PRIMARY KEY`、`datetime`などのキーワードに着目してください。

出力結果から、次の3つのSQL文が発行されていることがわかります。

1. トランザクション開始（`BEGIN`）
2. `divisions_division`テーブル作成（`CREATE TABLE "divisions_division" (...)`）
3. トランザクションコミット（`COMMIT`）

> **モデルが自動的に与えるテーブル名について**
>
> Djangoは、`<アプリ名>_<モデル名>`の書式で表現される名前でテーブルを作成します。
> なお、部署モデルに対応するテーブルのテーブル名は、後で変更します。
>
> **トランザクション**
>
> トランザクションは、トランザクションの開始から終了までの処理が、すべて成功して完了したか、まったく何も処理されなかった（変更されなかった）かを保証します。
> 銀行の口座振替では、自分の口座からお金が減り、相手の口座にお金が増えるように処理されます。
> この処理がすべて完全に成功するか、まったく何もされないかを保証する必要があります。
> この処理がすべて完全に成功した場合は、変更を確定（コミット）する必要があります。
> 逆に、自分の口座からお金が減る処理が成功して、相手の口座にお金が増える処理が失敗した場合、すべての変更を破棄（ロールバック）しないと、自分のお金が虚空に消滅します。

最後に、次のコマンドで部署モデルをデータベースに反映します（部署テーブルをデータベースに作成します）。
2つのコマンドを提示していますが、実行結果は同じです。
最初のコマンドは、適用されていないマイグレーションをすべてデータベースに反映します。
2番目のコマンドは、`divisions`アプリの`0001`番目のマイグレーションをデータベースに反映します。

```bash
python manage.py migrate
#または
python manage.py migrate divisions 0001
# Operations to perform:
#   Target specific migration: 0001_initial, from divisions
# Running migrations:
#   Applying divisions.0001_initial... OK
```

データベースのマイグレーションは、基本的に次の2つのコマンドを実行して、必要に応じてデータベースに発行されるSQL文を確認します。

```bash
# マイグレーション作成
python manage.py makemigrations
# SQL文確認（必要に応じて）
# python manage.py sqlmigrate <app> <number>
# マイグレーション反映
python manage.py migrate
```

## DjangoクエリセットAPIの操作

### クエリセットAPIを使用した部署モデルの操作

Djangoが提供する[クエリセットAPI](https://docs.djangoproject.com/en/4.2/topics/db/queries/)で部署モデルを操作します。

Pythonのインタラクティブコンソールを実行するために、次のコマンドを実行します。

```bash
python manage.py shell
```

起動したインタラクティブコンソールで、次を実行します。
なお、インタラクティブコンソールに表示された`>>>`は、入力プロンプトです。
インタラクティブコンソールには、先頭に`#`のない行を入力してください。
また、先頭が`#`から始まる行は説明のため、インタラクティブコンソールに入力する必要はありません。
出力結果は、入力したコマンドの下の行の`#`です。

```python
# シェルに部署モデルをインポート
from divisions.models import Division

# すべての部署モデルインスタンスを取得（部署が登録されていないため結果は空）
Division.objects.all()
# <QuerySet []>

# 部署モデルインスタンスを作成
>>> ict = Division()

# 部署モデルインスタンスに、ICT開発室の部署コードと部署名を設定
>>> ict.code = "58"
>>> ict.name = "ICT開発室"

# 部署モデルインスタンスをデータベースに保存
>>> ict.save()

# 登録した部署を確認
# 作成日時と更新日時がプロジェクト設定ファイルで設定したタイムゾーンではなく、世界標準時であることに注意
ict.code
# '58'
ict.name
# 'ICT開発室'
ict.created_at
# datetime.datetime(2023, 4, 19, 2, 17, 41, 815016, tzinfo=datetime.timezone.utc)
ict.updated_at
# datetime.datetime(2023, 4, 19, 2, 17, 41, 815042, tzinfo=datetime.timezone.utc)

# 道路第1Ｇを登録
road1 = Division(code="51", name="道路第1Ｇ")
road1.save()

# 登録した道路第1Ｇを確認
road1
# <Division: Division object (51)>

# すべての部署モデルインスタンスを取得
Division.objects.all()
# <QuerySet [<Division: Division object (58)>, <Division: Division object (51)>]>

# 登録されている部署を出力
# printはTabキーを1回押下した後に入力
# printの行の末尾でEnterキーを2回押下
for d in Division.objects.all():
    print(f"{d.code}: {d.name}, {d.created_at}, {d.updated_at}")

# 58: ICT開発室, 2023-04-19 02:17:41.815016+00:00, 2023-04-19 02:17:41.815042+00:00
# 51: 道路第1Ｇ, 2023-04-19 02:18:57.961611+00:00, 2023-04-19 02:18:57.961641+00:00

# road1変数を削除
del road1
road1
# Traceback (most recent call last):
#   File "<console>", line 1, in <module>
# NameError: name 'road1' is not defined

# 登録した道路第1Ｇを取得
road1 = Division.objects.get(pk="51")
road1.name
# '道路第1Ｇ'

# 部署名を`道路第1Ｇ`から`道路第2Ｇ`に変更
#road1.name = "道路第2Ｇ"
road1.save()

# 登録されている部署を出力
# 道路第2Ｇの作成日時と更新日時に注意
for d in Division.objects.all():
    print(f"{d.code}: {d.name}, {d.created_at}, {d.updated_at}")

# 58: ICT開発室, 2023-04-19 02:17:41.815016+00:00, 2023-04-19 02:17:41.815042+00:00
# 51: 道路第2Ｇ, 2023-04-19 02:18:57.961611+00:00, 2023-04-19 02:21:59.815208+00:00

# django.utils.timezoneモジュールをインポートして、現在日時を取得
from django.utils import timezone
dt = timezone.now()
dt
# datetime.datetime(2023, 5, 5, 8, 15, 9, 920556, tzinfo=datetime.timezone.utc)

# 部署コード51で道路第3Ｇを登録
# プライマリーキーに同じ部署コードをもつ部署モデルインスタンスが存在するため、
# 実際には、プライマリーキーが一致する部署モデルインスタンスが更新されます。
road3 = Division(code="51", name="道路第3Ｇ", created_at=dt, updated_at=dt)
road3.save()

# 登録されている部署を出力
# 道路第3Ｇの作成日時と更新日時に注意（Divisionコンストラクタで日時を無視）
for d in Division.objects.all():
    print(f"{d.code}: {d.name}, {d.created_at}, {d.updated_at}")

# 58: ICT開発室, 2023-04-19 02:17:41.815016+00:00, 2023-04-19 02:17:41.815042+00:00
# 51: 道路第3Ｇ, 2023-05-05 08:15:30.668455+00:00, 2023-05-05 08:15:30.668478+00:00

# 道路第3Ｇを削除
Division.objects.filter(pk="51").delete()
# (1, {'divisions.Division': 1})

# 登録されている部署を出力
for d in Division.objects.all():
    print(f"{d.code}: {d.name}, {d.created_at}, {d.updated_at}")

# 58: ICT開発室, 2023-04-19 02:17:41.815016+00:00, 2023-04-19 02:17:41.815042+00:00

# インタラクティブコンソールを終了
exit()
```

> Djangoの[クエリセットAPI](https://docs.djangoproject.com/en/4.2/ref/models/querysets/)は[遅延評価（`lazy`）](https://docs.djangoproject.com/en/4.2/topics/db/queries/#querysets-are-lazy)で、例えば`<model>.objects.all()`した結果が**本当に必要になったとき**に、データベースにアクセスします。
> ここで**本当に必要になったとき**とは、`<model>.objects.all()`で取得したモデルインスタンスの数を`len`関数で取得する場合などです。
>
> **クエリセットは遅延評価されます - Djangoドキュメント**
>
> クエリセットは遅延評価されます - クエリセットを作成する動作は任意のデータベース動作をもたらしません。
> クエリセットに対してフィルタ（`filter`）を何回も重ねることができ、実際にDjangoはクエリセットが評価されるまでクエリを実行しません。
> 次の例を確認してください。
>
> ```python
> >> q = Entry.objects.filter(headline__startswith="What")
> >> q = q.filter(pub_date__lte=datetime.date.today())
> >> q = q.exclude(body_text__icontains="food")
> >> print(q)
> ```
>
> これは3回のデータベースアクセスをしているように見えますが、実際には1回のみデータベースにアクセスしており、それは最後の行の`print(q)`です。
> 一般的に、クエリセットの結果は、それらを「尋ねる」までデータベースから取得されません。
> クエリセットに「尋ねた」とき、クエリセットはデータベースにアクセスすることにより評価されます。
> 評価される正確なタイミングの詳細は、[いつクエリセットが評価されるのか](https://docs.djangoproject.com/en/4.2/ref/models/querysets/#when-querysets-are-evaluated)を参照してください。

## モデルインスタンスの出力の変更とメソッドの追加

すべての部署モデルインスタンスを取得したとき、次の通り出力されました。

```python
Division.objects.all()
# <QuerySet [<Division: Division object (58)>, <Division: Division object (51)>]>
```

部署モデルインスタンスが`print`されたとき、より意味のある文字列を出力するように、`Division`モデルに`__str__`メソッドを追加します。
`__str__`メソッドは、Djangoがモデルインスタンスを出力する際に呼び出され、出力はこのメソッドが返却した文字列になります。

また、部署コードと部署名を連結した文字を返却する`full_name`メソッドと、まったく意味がありませんが、部署名の文字数を返却する`name_len`メソッドを追加します。

`./divisions/models.py`を次の通り変更します。

```python
# ./divisions/models.py
  from django.db import models


  class Division(models.Model):
      """部署モデル"""

      # 部署コード
      code = models.CharField(primary_key=True, max_length=2)
      # 部署名
      name = models.CharField(max_length=80)
      # 作成日時
      created_at = models.DateTimeField(auto_now_add=True)
      # 更新日時
      updated_at = models.DateTimeField(auto_now=True)

+     def __str__(self) -> str:
+         """部署モデルインスタンス名を返却する。
+
+         Returns:
+             部署モデルインスタンス名。
+         """
+         return self.name
+
+     def full_name(self) -> str:
+         """部署モデルインスタンスのフルネームを返却する。
+
+         Returns:
+             部署モデルインスタンスのフルネーム。
+         """
+         return f"{self.code}: {self.name}"
+
+     def name_len(self) -> int:
+         """部署名の文字数を返却する。
+
+         Returns:
+             部署名の文字数。
+         """
+         return len(self.name)
```

次に、Pythonインタラクティブコンソールを再起動して、部署モデルに実装したメソッドが機能するか確認します。

> 既存のPythonインタラクティブコンソールでは、部署モデルの変更が反映されていないため、正常に機能しません。
> 必ず、Pythonインタラクティブコンソールを再起動してください。

```bash
python manage.py shell
```

```python
from divisions.models import Division
Division.objects.all()
# <QuerySet [<Division: ICT開発室>]>
ict = Division.objects.get(pk="58")
ict
# <Division: ICT開発室>
ict.full_name()
# '58: ICT開発室'
ict.name_len()
# 6
```

Djangoが提供するクエリセットAPI（≒`ORM`）によって、モデルインスタンスの`save`メソッドを呼び出すことで、モデルに対応するデータベースのテーブルにレコードが登録または更新されます。

> **ORM (Object Relational Mapping)**
>
> ORMは、データベースとオブジェクト指向プログラミング言語の間で、データをやり取りする機能を提供します。
> データベースとDjangoにおいては、テーブルがモデル、カラム（列）がモデルのフィールド、モデルインスタンスがレコードに対応します。

DjangoのORMの詳細は、次を参照してください。

- [Making queries](https://docs.djangoproject.com/en/4.2/topics/db/queries/)
- [Django ORM Cookbook (Django 2.0)](https://books.agiliq.com/projects/django-orm-cookbook/en/latest/)

次の通り部署モデルの実装をリポジトリにコミットします。

```bash
git add --all
git commit -m '部署モデルを実装'
```

> 2278943 (tag: 006-implement-division-models)

## 管理サイト

Djangoは、定義したモデルを操作するWebサイトを`管理サイト (Admin Site)`として動的に作成して提供します。

管理サイトは、自由にデータを参照または更新できるため、本アプリケーションの管理者のみ使用して、一般ユーザーには公開しないでください。

> Djangoにおいて、アプリケーションの管理者はスーパーユーザーまたはスタッフです。
> スーパーユーザーは、後で説明するユーザーモデルインスタンスの`is_superuser`属性が`True`に設定されているユーザーです。
> また、同様にスタッフは`is_staff`属性が`True`に設定されているユーザーです。
> スーパーユーザー及びスタッフは、管理サイトにアクセスできます。

### スーパーユーザーの作成

管理サイトには、スーパーユーザーまたはスタッフしかアクセスできません。

> 管理サイトでユーザーをスーパーユーザーまたはスタッフに設定または解除できます。

次のコマンドを実行して、スーパーユーザーを作成します。
コマンドを実行すると、Djangoからの問い合わせに対する回答を入力します。
なお、パスワードの問い合わせに対して、短いまたは類推しやすいパスワードを入力すると警告が出力されます。
上記警告が表示された場合は、Djangoによるパスワード検証を回避するために、`Bypass password validation and create user anyway?`で`y`を入力します。

> チュートリアルのため、Djangoのパスワード検証機能を回避していますが、推奨されません。

```bash
python manage.py createsuperuser
ユーザー名 (leave blank to use 'xjr1300'): django
メールアドレス: xjr1300.04@gmail.com
Password:
Password (again):
このパスワードは ユーザー名 と似すぎています。
このパスワードは短すぎます。最低 8 文字以上必要です。
このパスワードは一般的すぎます。
Bypass password validation and create user anyway? [y/N]: y
Superuser created successfully.
```

Djangoのパスワード検証機能は、プロジェクト設定ファイルの`AUTH_PASSWORD_VALIDATORS`で設定されています。
デフォルトで設定されている検証方法以外に、Djangoまたはサードパーティが提供する検証を利用できます。
なお、デフォルトでパスワードは次が検証されます。

- 短いパスワードを拒否（`MinimumLengthValidator`）
- 一般的なパスワードを拒否（`CommonPasswordValidator`）
- ユーザーの属性と似たパスワードを拒否（`UserAttributeSimilarityValidator`）
- 数字だけのパスワードを距離（`NumericPasswordValidator`）

## 管理サイトの有効化

`./divisions/admin.py`を開いて、次のとおり変更します。

```python
# ./divisions/admin.py
  from django.contrib import admin

+ from .models import Division

- # Register your models here.
+ admin.site.register(Division)
```

開発サーバーが起動していない場合は、開発サーバーを起動してください。
その後、ブラウザで`http://localhost:8000/admin/`にアクセスしてください。

管理サイトのログインページが表示されるため、`createsuperuser`コマンドで入力したユーザー名とパスワードを入力して`ログイン`ボタンをクリックしてください。

本チュートリアルでは説明しませんが、管理サイトが部署モデルインスタンスを操作する機能を提供していることを確認できます。

## 管理サイトのカスタマイズ

管理サイトにおける部署モデル及びそのインスタンスの表示をカスタマイズして、管理サイトで部署モデルインスタンスを操作しやすくします。
`./divisions/admin.py`、`./divisions/models.py`及び`./divisions/apps.py`を次の通り変更します。

```python
# ./divisions/admin.py
  from django.contrib import admin

  from .models import Division


+ class DivisionAdmin(admin.ModelAdmin):
+     list_display = (
+         "code",
+         "name",
+         "created_at",
+         "updated_at",
+     )
+

- admin.site.register(Division)
+ admin.site.register(Division, DivisionAdmin)
```

```python
# ./divisions/models.py

      # 部署コード
-     code = models.CharField(primary_key=True, max_length=2)
+     code = models.CharField("部署コード", primary_key=True, max_length=2)
      # 部署名
-     name = models.CharField(max_length=80)
+     name = models.CharField("部署名", max_length=80)
      # 作成日時
-     created_at = models.DateTimeField(auto_now_add=True)
+     created_at = models.DateTimeField("作成日時", auto_now_add=True)
      # 更新日時
-     updated_at = models.DateTimeField(auto_now=True)
+     updated_at = models.DateTimeField("更新日時", auto_now=True)
+
+     class Meta:
+         db_table = "divisions"
+         verbose_name = verbose_name_plural = "部署"
+         ordering = ("code",)
+
      def __str__(self) -> str:
          """部署モデルインスタン名を返却する。"""
```

```python
# ./divisions/apps.py
  from django.apps import AppConfig


  class DivisionsConfig(AppConfig):
      default_auto_field = "django.db.models.BigAutoField"
      name = "divisions"
+     verbose_name = "部署アプリ"
```

その後、部署モデルを次の通りマイグレーションします。

```bash
python manage.py makemigrations
# 出力
# Migrations for 'divisions':
#   divisions/migrations/0002_alter_division_options_alter_division_code_and_more.py
#     - Change Meta options on division
#     - Alter field code on division
#     - Alter field created_at on division
#     - Alter field name on division
#     - Alter field updated_at on division
#     - Rename table for division to divisions
python manage.py migrate
```

ブラウザで管理サイトを表示すると、`DIVISIONS`と表示されていた部署アプリが、`DivisionConfig.verbose_name`の設定により、`部署アプリ`と表示されていることを確認できます。

`Division`または`Division`と表示されていた部署モデルの名前が、`Divisions.Meta.verbose_name`及び`verbose_name_plural`の設定により、`部署`と表示されていることを確認できます。
なお、`verbose_name_plural`は、モデルの複数形の名前を指定します。

部署を表示するリストのヘッダーが、モデルフィールドに設定した文字列に変更されていることも確認できます。

`ordering`は、取得した部署モデルインスタンスを並び替える順番を設定しています。

なお、`Division.Meta.db_table`により、部署モデルインスタンスを記録するテーブルの名前が`divisions_division`から`divisions`に変更されました。

部署モデルを管理サイトに登録したら、次の通り変更をリポジトリにコミットします。

```bash
git add --all
git commit -m '部署モデルを管理サイトに登録'
```

> 08242bb (tag: 007-register-division-models-to-admin-site)

管理サイトをカスタマイズする詳細な方法は、次を参照してください。

- [The Django admin site](https://docs.djangoproject.com/en/4.2/ref/contrib/admin/)
- [Django Admin Cookbook (Django 2.0)](https://books.agiliq.com/projects/django-admin-cookbook/)を参照してください。

## フィクスチャー（Fixture）

`フィクスチャー`とは、データベースに記録されているモデルインスタンスをファイルに記録したものです。
フィクスチャーを利用することで、データベースにモデルインスタンスを登録し、逆にデータベースに記録されているモデルインスタンスをファイルに出力できます。

本チュートリアルでは、フィクスチャーを[YAML](https://yaml.org/)形式のテキストファイルで扱います。
Djangoはより一般的な[JSON](https://developer.mozilla.org/ja/docs/Learn/JavaScript/Objects/JSON)でフィクスチャーを管理できますが、JSONはコメントを記録できないためYAMLを使用しています。。

> なお、YAML形式のファイルは、一般的に拡張子`.yml`ですが、Djangoは拡張子`.yaml`をYAMLファイルとして認識するようです。

次の通り、DjangoがYAML形式ファイルを取り扱えるように`PyYAML`パッケージをインストールして、そのパッケージとバージョンを`requirements.txt`に出力します。
その後、変更をリポジトリにコミットします。

```bash
pip install PyYAML
pip freeze > requirements.txt
git add --all
git commit -m 'PyYAMLパッケージをインストール'
```

> bbcf045 (tag: 008-install-pyyaml)

### 部署モデルインスタンスをフィクスチャーに出力

データベースに記録されている部署モデルインスタンスの内容をフィクスチャーに出力するために、次のコマンドを入力します。

```bash
mkdir divisions/fixtures
python manage.py dumpdata divisions.division --format=yaml > divisions/fixtures/divisions.yaml
```

出力された部署フィクスチャーを次に示します。

```yaml
- model: divisions.division
  pk: '58'
  fields:
    name: ICT開発室
    created_at: 2023-04-19 02:17:41.815016+00:00
    updated_at: 2023-04-19 02:17:41.815042+00:00
```

### 部署モデルインスタンスをフィクスチャーに追加

`./divisions/fixtures/divisions.yaml`ファイルを開いて、道路第3Ｇと東京本社営業部を部署フィクスチャーに追加します。

```yaml
+ - model: divisions.division
+   pk: "61"
+   fields:
+     name: 道路第3Ｇ
+     created_at: 2023-04-19 00:00:00.000000+00:00
+     updated_at: 2023-04-19 00:00:00.000000+00:00
+ - model: divisions.division
+   pk: "80"
+   fields:
+     name: 東京本社営業部
+     created_at: 2023-04-19 00:00:00.000000+00:00
+     updated_at: 2023-04-19 00:00:00.000000+00:00
```

### 部署フィクスチャーで部署モデルインスタンスを登録

次の通り、部署フィクスチャーを使用して部署モデルインスタンスをデータベースに登録します。

```bash
python manage.py loaddata ./divisions/fixtures/divisions.yaml
# Installed 3 object(s) from 1 fixture(s)
```

DjangoのクエリセットAPIを使用して、部署モデルインスタンスが正しく登録されていること確認してください。

```bash
python manage.py shell
```

```python
from divisions.models import Division
Division.objects.all()
# <QuerySet [<Division: ICT開発室>, <Division: 道路第3Ｇ>, <Division: 東京本社営業部>]>
exit()
```

次の通り、部署フィクスチャーをリポジトリにコミットします。

```bash
git add --all
git commit -m '部署フィクスチャーを追加'
```

> d85fbae (tag: 009-add-division-fixture)

## まとめ

本章では、プロジェクトに部署アプリを追加して、部署モデルを実装しました。
また、管理サイトで部署モデルインスタンスを参照したり、クエルセットAPIを利用してモデルインスタンスを`CRUD`しました。
さらに、部署モデルインスタンスをフィクスチャーに保存した後、2つの部署を部署フィクスチャーに追加して、部署モデルインスタンスをデータベースに記録しました。

次の章では、部署モデルインスタンスを操作する関数ビューを実装します。
