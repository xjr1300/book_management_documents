# REST API

- [REST API](#rest-api)
  - [REST APIとは](#rest-apiとは)
    - [リソース指向の設計](#リソース指向の設計)
    - [HTTPメソッドの使用](#httpメソッドの使用)
    - [ステートレスな設計（状態を持たない設計）](#ステートレスな設計状態を持たない設計)
    - [書式化されたデータの使用](#書式化されたデータの使用)
  - [Django REST Framework](#django-rest-framework)
  - [Django REST Frameworkのインストール](#django-rest-frameworkのインストール)
  - [書籍APIアプリの追加](#書籍apiアプリの追加)
  - [シリアライザー (Serializer) とは](#シリアライザー-serializer-とは)
  - [書籍分類APIの実装](#書籍分類apiの実装)
    - [書籍分類シリアライザーの実装](#書籍分類シリアライザーの実装)
    - [書籍分類APIビューの実装](#書籍分類apiビューの実装)
    - [書籍分類APIビューのディスパッチ](#書籍分類apiビューのディスパッチ)
    - [書籍分類APIの呼び出し](#書籍分類apiの呼び出し)
      - [書籍分類一覧APIの呼び出し](#書籍分類一覧apiの呼び出し)
      - [書籍分類（`Classification`）詳細APIの呼び出し](#書籍分類classification詳細apiの呼び出し)
      - [書籍分類登録APIの呼び出し](#書籍分類登録apiの呼び出し)
      - [書籍分類更新APIの呼び出し](#書籍分類更新apiの呼び出し)
      - [書籍分類削除APIの呼び出し](#書籍分類削除apiの呼び出し)
  - [書籍分類詳細APIの実装](#書籍分類詳細apiの実装)
    - [書籍分類詳細更新用シリアライザーの実装](#書籍分類詳細更新用シリアライザーの実装)
    - [書籍分類詳細シリアライザーの実装](#書籍分類詳細シリアライザーの実装)
    - [書籍分類詳細APIビューの実装](#書籍分類詳細apiビューの実装)
    - [書籍分類詳細APIビューのディスパッチ](#書籍分類詳細apiビューのディスパッチ)
    - [書籍分類詳細（`ClassificationDetail`）APIの呼び出し](#書籍分類詳細classificationdetailapiの呼び出し)
      - [書籍分類詳細一覧APIの呼び出し](#書籍分類詳細一覧apiの呼び出し)
      - [書籍分類詳細詳細APIの呼び出し](#書籍分類詳細詳細apiの呼び出し)
      - [書籍分類詳細登録APIの呼び出し](#書籍分類詳細登録apiの呼び出し)
      - [書籍分類詳細更新APIの呼び出し](#書籍分類詳細更新apiの呼び出し)
    - [書籍分類詳細削除APIの呼び出し](#書籍分類詳細削除apiの呼び出し)
  - [書籍APIの実装](#書籍apiの実装)
    - [書籍シリアライザーの実装](#書籍シリアライザーの実装)
      - [書籍読み込み専用シリアライザーの実装](#書籍読み込み専用シリアライザーの実装)
      - [書籍書き込み専用シリアライザーの実装](#書籍書き込み専用シリアライザーの実装)
    - [書籍ビューセットの実装](#書籍ビューセットの実装)
    - [書籍ビューセットのディスパッチ](#書籍ビューセットのディスパッチ)
  - [書籍APIの保護](#書籍apiの保護)
    - [JWT (JSON Web Token)とは](#jwt-json-web-tokenとは)
    - [認証API](#認証api)
      - [Simple JWTのインストール](#simple-jwtのインストール)
      - [Simple JWTの設定](#simple-jwtの設定)
      - [認証APIの実装](#認証apiの実装)
    - [書籍登録、更新及び削除APIの保護](#書籍登録更新及び削除apiの保護)
    - [書籍API保護の確認](#書籍api保護の確認)
      - [認証APIの呼び出し](#認証apiの呼び出し)
      - [書籍一覧及び詳細APIの呼び出し](#書籍一覧及び詳細apiの呼び出し)
      - [アクセストークンを付けて書籍登録APIを呼び出し](#アクセストークンを付けて書籍登録apiを呼び出し)
      - [アクセストークンなしで書籍登録APIを呼び出し](#アクセストークンなしで書籍登録apiを呼び出し)
      - [アクセストークンなしで書籍更新APIを呼び出し](#アクセストークンなしで書籍更新apiを呼び出し)
      - [アクセストークンなしで書籍削除APIを呼び出し](#アクセストークンなしで書籍削除apiを呼び出し)
      - [アクセストークンを付けて書籍更新APIを呼び出し](#アクセストークンを付けて書籍更新apiを呼び出し)
      - [アクセストークンを付けて書籍削除APIを呼び出し](#アクセストークンを付けて書籍削除apiを呼び出し)

本章では、書籍分類、書籍分類詳細及び書籍を取得、登録、更新及び削除するWeb APIを`REST`形式で作成します。

## REST APIとは

`REST API`は、Webアプリケーションの機能を提供するためのAPIの設計方法の1つです。
`REST API`は、`Representational State Transfer`の略で、HTTPプロトコルを使用してクライアントとサーバー間でデータを転送する目的で使用されます。

`REST API`は、Webアプリケーションの機能を外部から利用するために広く使用されており、Web APIを外部に提供するWebサイトの多くは`REST API`を採用しています。

ただし、`REST API`に明確な仕様は無いため、何が`RESTful（レストフル、レストを満たす）`なのかは曖昧です。

> `REST API`には、[N+1問題](https://restfulapi.net/rest-api-n-1-problem/)と呼ばれる課題があり、これを解決する[GraphQL](https://graphql.org/)がありますが、現時点（2023-04-30）で内部提供向けに実装されていることが多く、外部提供は進んでいません。
> `GraphQL`は、`REST API`と異なり明確な仕様が定義されています。

`REST API`の設計原則を次に示します。

- リソース指向の設計
- HTTPメソッドの使用
- ステートレスな設計（状態を持たない設計）
- 書式化されたデータの使用

### リソース指向の設計

リソース指向の設計では、リソースに対してユニークな識別子（`URI`）を割り当てることが重要です。
書籍の場合であれば、`http://localhost/api/books/`のようなURIを割り当てます。

### HTTPメソッドの使用

HTTPメソッドは、次の通り区別して使用します。

- GETメソッド: リソースの取得
- POSTメソッド: リソースの作成
- PUTメソッド: リソースの更新
- PATCHメソッド: リソースの一部更新
- DELETEメソッド: リソースの削除

### ステートレスな設計（状態を持たない設計）

Djangoは、認証情報をセッションに記録しますが、`REST API`ではセッションなどの状態を持たずに、リクエストとレスポンスのみを使用して通信を行います。
認証状態などは、後で説明する`JWT（「じょっと」と呼びます）`で判断します。

### 書式化されたデータの使用

`REST API`では、書式化されたデータを使用して、クライアントとサーバー間でデータを転送します。
`JSON`や`XML`などの形式を利用することで、異なるプログラミング言語やプラットフォーム間での互換性が向上します。
本Webアプリケーションは、`JSON`を採用します。

## Django REST Framework

[Django REST Framework（DRF)](https://www.django-rest-framework.org/)は、Djangoのモデルを基に`REST API`の実装を支援するパッケージです。

本チュートリアルでは、書籍分類、書籍詳細及び書籍を`CRUD`する`REST API`をDRFで実装します。

DRFは次の通りリクエストを処理します。

1. ビューが、リクエストを受信する。
2. `シリアライザー`が、リクエストデータを解析する（`デシリアライズ`）。
3. `シリアライザー`が、モデルインスタンスを操作する。
4. `シリアライザー`が、結果を生成する（`シリアライズ`）。
5. ビュー、が結果をレスポンスとして送信する。

`シリアライザー`は、後で説明します。

## Django REST Frameworkのインストール

次の通りDRFをインストールします。

<!-- cspell: disable -->
```bash
pip install djangorestframework djangorestframework-stubs
```
<!-- cspell: enable -->

`mypy`による静的型チェックのために、`./pyproject.toml`ファイルを次の通り変更します。

```toml
# ./pyproject.toml
  [tool.mypy]
  python_version = "3.9"
  no_strict_optional = true
  ignore_missing_imports = true
  check_untyped_defs = true
  exclude = ['^manage\.py$', '^settings\.py$', '^migrations$', 'venv']
- plugins = ["mypy_django_plugin.main"]
+ plugins = ["mypy_django_plugin.main", "mypy_drf_plugin.main"]
```

プロジェクト設定ファイルの`INSTALLED_APP`に`rest_framework`を次の通り追加します。

```python
# ./book_management/settings.py
  INSTALLED_APPS = [
      "django.contrib.admin",
      "django.contrib.auth",
      "django.contrib.contenttypes",
      "django.contrib.sessions",
      "django.contrib.messages",
      "django.contrib.staticfiles",
+     "rest_framework",
      "django_bootstrap5",
      "debug_toolbar",
      "accounts.apps.AccountsConfig",
      "divisions.apps.DivisionsConfig",
      "books.apps.BooksConfig",
  ]
```

次の通り`requirements.txt`を更新した後、変更をリポジトリにコミットします。

```bash
pip freeze > requirements.txt
git add ./requirements.txt
git commit -m 'Django REST Frameworkをインストール'
```

> 48bef78 (tag: 064-install-django-rest-framework)

## 書籍APIアプリの追加

書籍アプリのモデルを操作するアプリを次の通り追加します。

```bash
python manage.py startapp api1
```

アプリ名の`api1`の末尾の`1`は、APIのバージョンを示しています。
本チュートリアルでAPIをバージョン管理しませんが、APIをバージョニングするためにバージョン番号をつける事例があります。

プロジェクト設定ファイルの`INSTALLED_APP`に`api`アプリを追加します。

```python
# ./book_management/settings.py
  INSTALLED_APPS = [
      "django.contrib.admin",
      "django.contrib.auth",
      "django.contrib.contenttypes",
      "django.contrib.sessions",
      "django.contrib.messages",
      "django.contrib.staticfiles",
+     "rest_framework",
      "django_bootstrap5",
      "debug_toolbar",
      "accounts.apps.AccountsConfig",
      "divisions.apps.DivisionsConfig",
      "books.apps.BooksConfig",
      "api1.apps.Api1Config",
  ]
```

次の通り変更をリポジトリにコミットします。

```bash
git add ./api1
git commit -m 'api1アプリを追加'
```

> 9ae6d11 (tag: 065-add-api1-app)

## シリアライザー (Serializer) とは

[シリアライザー (Serializer)](https://www.django-rest-framework.org/api-guide/serializers/)は、JSONなどのフォーマットとPythonオブジェクトの間の変換を行うクラスです。
シリアライザーを使用することで、PythonオブジェクトをJSONやXMLなどの形式に相互変換できます。
シリアライザーは、通常、次の処理をします。

- リクエストデータのバリデーション
- PythonオブジェクトをJSONやXMLなどのフォーマットに変換する
- JSONやXMLなどのフォーマットをPythonオブジェクトに変換する
- レスポンスデータのバリデーション

シリアライザーは、Djangoのフォームと同様に定義され、また処理内容も同様です。

## 書籍分類APIの実装

書籍APIを実装する`./api1/books/`ディレクトリと`./api1/books/__init__.py`ファイルを次の通り作成して、

```bash
mkdir ./api1/books
touch ./api1/books/__init__.py
```

### 書籍分類シリアライザーの実装

書式分類シリアライザー（`ClassificationSerializer`）を次の通り実装します。

```python
# ./api1/books/serializers.py
from typing import Any

from rest_framework import serializers

from books.models import Classification


class ClassificationSerializer(serializers.Serializer):
    """書籍分類シリアライザー"""

    # 書籍分類コード
    code = serializers.CharField(max_length=3)
    # 書籍分類名
    name = serializers.CharField(max_length=80)
    # 作成日時
    created_at = serializers.DateTimeField(read_only=True)
    # 更新日時
    updated_at = serializers.DateTimeField(read_only=True)

    def create(self, validated_data: Any) -> Classification:
        """
        validated_dataから書籍分類を作成して返却する。

        Args:
            validated_data: 書籍分類シリアライザーが検証したデータ。

        Returns:
            書籍分類モデルインスタンス。
        """
        return Classification.objects.create(**validated_data)

    def update(self, instance: Classification, validated_data: Any) -> Classification:
        """
        既存の書籍分類モデルインスタンスをvalidated_dataで更新して返却する。

        Args:
            instance: 更新する書籍分類モデルインスタンス。
            validated_data: 書籍分類シリアライザーが検証したデータ。

        Returns:
            書籍分類モデルインスタンス。
        """
        instance.code = validated_data.get("code", instance.code)
        instance.name = validated_data.get("name", instance.name)
        instance.save()
        return instance
```

書籍分類シリアライザーの作成日時フィールドと更新日時は、本アプリケーションが自動的に設定するため、読み込み専用（`read_only=True`）にしています。
書籍分類シリアライザーを実装したら、次の通り変更をリポジトリにコミットします。

```bash
git add ./api1/books/
git commit -m '書籍分類シリアライザーを実装'
```

> c4edf10 (tag: 066-implement-classification-serializer)

### 書籍分類APIビューの実装

書籍分類APIビューを次の通り実装します。

```python
# ./api1/books/views.py
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.request import Request
from rest_framework.response import Response

from books.models import Classification

from .serializers import ClassificationSerializer


@api_view(["GET", "POST"])
def classification_list(request: Request) -> Response:
    """すべての書籍分類を返却するか、書籍分類を登録する。

    GETメソッドの場合は、すべての書籍分類モデルインスタンスを返却する。
    POSTメソッドの場合は、書籍分類モデルインスタンスを登録する。

    Args:
        request: リクエストインスタンス。
    Returns:
        レスポンス。
    """
    if request.method == "GET":
        # GETメソッドの場合は、すべての書籍分類モデルインスタンスを返却
        classifications = Classification.objects.all()
        serializer = ClassificationSerializer(classifications, many=True)
        return Response(serializer.data)

    elif request.method == "POST":
        # POSTメソッドの場合は、書籍分類モデルインスタンスを登録
        serializer = ClassificationSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)


@api_view(["GET", "PUT", "DELETE"])
def classification_detail(request: Request, code: str) -> Response:
    """
    書籍分類を取得、更新または削除する。

    Args:
        request: リクエストインスタンス。
        code: 書籍分類コード。
    Returns:
        レスポンス。
    """
    # 書籍分類コードから書籍分類モデルインスタンスを取得
    try:
        instance = Classification.objects.get(code=code)
    except Classification.DoesNotExist:
        # 書籍分類モデルインスタンスを取得できない場合は、`404 Not Found`を返却
        return Response(status=status.HTTP_404_NOT_FOUND)

    if request.method == "GET":
        # GETメソッドの場合は、書籍分類モデルインスタンスを返却
        serializer = ClassificationSerializer(instance)
        return Response(serializer.data)

    elif request.method == "PUT":
        # PUTメソッドの場合は、書籍分類モデルインスタンスを更新して返却
        # 書籍分類コードの更新は不可
        request.data["code"] = code
        serializer = ClassificationSerializer(instance, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    elif request.method == "DELETE":
        # DELETEメソッドの場合は、書籍分類モデルインスタンスを削除
        instance.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

`@api_view`デコレーターの主な機能は、そのビューがレスポンスするHTTPメソッドのリストを受けとります。
デフォルトは、`GET`メソッドのみです。
例えば、`classification_list`関数ビューは、`GET`メソッドと`POST`メソッドにレスポンスします。
`@api_view`デコレーターに指定したメソッド以外でそのビューが呼び出された場合、`405 Method Not Allowed`をレスポンスします。

`rest_framework.request.Request`は、`django.http.HttpRequest`をDRFが拡張したクラスです。

`ClassificationSerializer`は、書籍分類モデルインスタンスをJSON形式に変換（`シリアライズ`）して、また受け取った`POST`データを書籍分類モデルインスタンスに変換（`デシリアライズ`）します。

書籍分類APIビューを実装したら、次の通り変更をリポジトリにコミットします。

```bash
git add ./api1/books/views.py
git commit -m '書籍分類APIビューを実装`
```

> c66d7e4 (tag: 067-implement-classification-views)

### 書籍分類APIビューのディスパッチ

書籍分類APIビューを次の通りディスパッチします。

```python
# ./api1/urls.py
from django.urls import path

from .books import views

urlpatterns = [
    path('books/classifications/', views.classification_list),
    path('books/classifications/<str:code>/', views.classification_detail),
]
```

また、ルートURLconfを次の通り設定します。

```python
# ./book_management/urls.py
  urlpatterns = [
      path("accounts/", include("accounts.urls")),
      path("divisions/", include("divisions.urls")),
      path("books/", include("books.urls")),
+     path("api1/", include("api1.urls")),
      path("admin/", admin.site.urls),
  ]
```

書籍分類ビューをディスパッチしたら、次の通り変更をリポジトリにコミットします。

```bash
git add ./api1/urls.py
git add ./book_management/urls.py
git commit -m '書籍分類APIを実装'
```

> e1523a5 (tag: 068-implement-classification-api)

### 書籍分類APIの呼び出し

書籍分類APIをターミナルから`curl`コマンドで呼び出します。
次のコマンドをターミナルで実行して、`curl`コマンドがインストールされているか確認します。

```bash
curl --version
```

`curl`コマンドのバージョンが表示されたら`curl`コマンドがインストールされています。
`curl`コマンドがインストールされていない場合は、次の通り`curl`コマンドをインストールします。

```bash
sudo apt -y install curl
```

また、レスポンスとして得られるJSONを整形するために`jq`コマンドを使用します。
次のコマンドをターミナルで実行して、`jq`コマンドがインストールされているか確認します。

```bash
jq --version
```

`jq`コマンドのバージョンが表示されたら`jq`コマンドがインストールされています。
`jq`コマンドがインストールされていない場合は、次の通り`jq`コマンドをインストールします。

```bash
sudo apt -y install jq
```

#### 書籍分類一覧APIの呼び出し

書籍分類一覧APIを次の通り呼び出します。
なお、`#`は、書籍分類一覧APIからのレスポンスを`jq`コマンドで整形した結果です。
実際の書籍分類一覧APIからのレスポンスは、レスポンスデータが大きくならないように、スペースや改行が削除されているはずです。

<!-- cspell: disable -->
```bash
curl -s http://localhost:8000/api1/books/classifications/ | jq .
[
  {
    "code": "000",
    "name": "総記",
    "created_at": "2023-04-24T09:00:00+09:00",
    "updated_at": "2023-04-24T09:00:00+09:00"
  },
  {
    "code": "100",
    "name": "哲学",
    "created_at": "2023-04-24T09:00:00+09:00",
    "updated_at": "2023-04-24T09:00:00+09:00"
  },
  {
    "code": "200",
    "name": "歴史",
    "created_at": "2023-04-24T09:00:00+09:00",
    "updated_at": "2023-04-24T09:00:00+09:00"
  },
  {
    "code": "300",
    "name": "社会科学",
    "created_at": "2023-04-24T09:00:00+09:00",
    "updated_at": "2023-04-24T09:00:00+09:00"
  },
  {
    "code": "400",
    "name": "自然科学",
    "created_at": "2023-04-24T09:00:00+09:00",
    "updated_at": "2023-04-24T09:00:00+09:00"
  },
  {
    "code": "500",
    "name": "技術",
    "created_at": "2023-04-24T09:00:00+09:00",
    "updated_at": "2023-04-24T09:00:00+09:00"
  },
  {
    "code": "600",
    "name": "産業",
    "created_at": "2023-04-24T09:00:00+09:00",
    "updated_at": "2023-04-24T09:00:00+09:00"
  },
  {
    "code": "700",
    "name": "芸術",
    "created_at": "2023-04-24T09:00:00+09:00",
    "updated_at": "2023-04-24T09:00:00+09:00"
  },
  {
    "code": "800",
    "name": "言語",
    "created_at": "2023-04-24T09:00:00+09:00",
    "updated_at": "2023-04-24T09:00:00+09:00"
  },
  {
    "code": "900",
    "name": "文学",
    "created_at": "2023-04-24T09:00:00+09:00",
    "updated_at": "2023-04-24T09:00:00+09:00"
  }
]
```
<!-- cspell: enable -->

なお、`curl`コマンドで指定した`-s`は、サイレントモードを示しており、リクエストを発行してレスポンスを受け取るまでの経過時間などを表示しないオプションです。

ちなみに、書籍一覧APIを許可されていない｀PUT`メソッドで呼び出した場合、`405 Method Not Allowed`が返却されます。

<!-- cspell: disable -->
```bash
curl -X PUT -i -s http://localhost:8000/api1/books/classifications/
# HTTP/1.1 405 Method Not Allowed
# Date: Tue, 02 May 2023 02:09:16 GMT
# Server: WSGIServer/0.2 CPython/3.11.2
# Content-Type: application/json
# Vary: Accept, Cookie
# Allow: GET, POST, OPTIONS
# djdt-store-id: f5aa6483c73b465d8be98dccd32fbdc3
# Server-Timing: TimerPanel_utime;dur=3.642999999996732;desc="User CPU time", TimerPanel_stime;dur=2.131000000019867;desc="System CPU time", TimerPanel_total;dur=5.774000000016599;desc="Total CPU time", TimerPanel_total_time;dur=7.985115051269531;desc="Elapsed time", SQLPanel_sql_time;dur=0;desc="SQL 0 queries", CachePanel_total_time;dur=0;desc="Cache 0 Calls"
# X-Frame-Options: DENY
# Content-Length: 64
# X-Content-Type-Options: nosniff
# Referrer-Policy: same-origin
# Cross-Origin-Opener-Policy: same-origin
#
# {"detail":"メソッド \"PUT\" は許されていません。"}
```
<!-- cspell: enable -->

なお、`curl`コマンドで指定した`-X`は、HTTPメソッドを指定するオプションで、ここでは`PUT`メソッドを指定しています。
また、`-i`は、レスポンスに加えて、レスポンスヘッダを出力するオプションです。

#### 書籍分類（`Classification`）詳細APIの呼び出し

書籍分類詳細APIを次の通り呼び出します。

<!-- cspell: disable -->
```bash
curl -s http://localhost:8000/api1/books/classifications/100/ | jq .
# {
#   "code": "100",
#   "name": "哲学",
#   "created_at": "2023-04-24T09:00:00+09:00",
#   "updated_at": "2023-04-24T09:00:00+09:00"
# }
```
<!-- cspell: enable -->

#### 書籍分類登録APIの呼び出し

書籍分類登録APIを次の通り呼び出します。
次のコマンドでは、コード`999`及び名前`ダミー書籍分類`の書式分類を登録します。
なお、`curl`コマンドの`-H`は、リクエストヘッダに独自のヘッダを追加するオプションで、ここでは`POST`するデータが`JSON`であることを、`Content-Type: application/json`で指定しています。
また、`-d`は、`POST`データを指定するオプションで、登録する書籍分類をJSONで表現しています。
<!-- cspell: disable -->
```bash
curl -X POST -H "Content-Type: application/json" -d '{"code": "999", "name": "ダミー書籍分類"}' -s http://localhost:8000/api1/books/classifications/ | jq .
# {
#   "code": "999",
#   "name": "ダミー書籍分類",
#   "created_at": "2023-05-02T14:08:30.530657+09:00",
#   "updated_at": "2023-05-02T14:08:30.530686+09:00"
# }
```
<!-- cspell: enable -->

登録した書籍分類が、実際に登録されているか確認します。

<!-- cspell: disable -->
```bash
curl -s http://localhost:8000/api1/books/classifications/999/
# {"code":"999","name":"ダミー書籍分類","created_at":"2023-05-02T14:08:30.530657+09:00","updated_at":"2023-05-02T14:08:30.530686+09:00"}
```
<!-- cspell: enable -->

#### 書籍分類更新APIの呼び出し

書籍分類更新APIを次の通り呼び出します。
次のコマンドでは、書籍分類コード`999`の書籍分類名を`ダミー書籍分類（更新後）`に変更します。

```bash
curl -X PUT -H "Content-Type: application/json" -s -d '{"code": "999", "name": "ダミー書籍分類（更新後）"}' -s http://localhost:8000/api1/books/classifications/999/ | jq .
# {
#   "code": "999",
#   "name": "ダミー書籍分類（更新後）",
#   "created_at": "2023-05-02T14:08:30.530657+09:00",
#   "updated_at": "2023-05-02T14:11:27.625771+09:00"
# }
```

更新した書籍分類が、実際に更新されているか確認します。

<!-- cspell: disable -->
```bash
curl -s http://localhost:8000/api1/books/classifications/999/ | jq .
# {
#   "code": "999",
#   "name": "ダミー書籍分類（更新後）",
#   "created_at": "2023-05-02T14:08:30.530657+09:00",
#   "updated_at": "2023-05-02T14:11:27.625771+09:00"
# }
```
<!-- cspell: enable -->

#### 書籍分類削除APIの呼び出し

書籍分類削除APIを次の通り呼び出します。
次のコマンドは、前に登録した書籍分類コード`999`の書籍分類を削除します。

<!-- cspell: disable -->
```bash
curl -X DELETE -i -s http://localhost:8000/api1/books/classifications/999/
# HTTP/1.1 204 No Content
# Date: Tue, 02 May 2023 05:13:39 GMT
# Server: WSGIServer/0.2 CPython/3.11.2
# Vary: Accept, Cookie
# Allow: DELETE, OPTIONS, GET, PUT
# djdt-store-id: 72b8ad62f8434b328300ad57c71c3287
# Server-Timing: TimerPanel_utime;dur=8.587999999999596;desc="User CPU time", TimerPanel_stime;dur=2.523000000000053;desc="System CPU time", TimerPanel_total;dur=11.110999999999649;desc="Total CPU time", TimerPanel_total_time;dur=12.850046157836914;desc="Elapsed time", SQLPanel_sql_time;dur=0.6101131439208984;desc="SQL 4 queries", CachePanel_total_time;dur=0;desc="Cache 0 Calls"
# X-Frame-Options: DENY
# Content-Length: 0
# X-Content-Type-Options: nosniff
# Referrer-Policy: same-origin
# Cross-Origin-Opener-Policy: same-origin
```
<!-- cspell: enable -->
HTTPレスポンスステータスコードが`204 No Content`であるため、書籍分類コード`999`の書籍分類が削除されたはずです。

削除した書籍分類が、実際に削除されているか確認します。

<!-- cspell: disable -->
```bash
curl -i -s http://localhost:8000/api1/books/classifications/999/ | grep HTTP
# HTTP/1.1 404 Not Found
```
<!-- cspell: enable -->

レスポンスステータスコードが`404 Not Found`であるため、書籍分類コード`999`の書籍分類が削除されていることを確認できました。

## 書籍分類詳細APIの実装

書籍分類シリアライザーは、`rest_framework.serializers.Serializer`を継承して実装しました。
また、書籍分類ビューは関数ビューとして実装しました。

書籍分類詳細APIの実装は、モデルからシリアライザーを実装する[rest_framework.serializers.ModelSerializer](https://www.django-rest-framework.org/api-guide/serializers/#modelserializer)と、[ジェネリックなクラスビュー](https://www.django-rest-framework.org/api-guide/generic-views/)で実装します。
なお、書籍分類詳細コードを変更できないようにするため、更新用のシリアライザーとそれ以外のシリアライザーに分けて書籍分類詳細シリアライザーを実装します。

### 書籍分類詳細更新用シリアライザーの実装

書籍分類詳細更新用シリアライザーを次の通り実装します。

```python
# ./api1/books/serializers.py
- from rest_framework import serializers
+ from rest_framework import exceptions, serializers

- from books.models import Classification
+ from books.models import Classification, ClassificationDetail

  (...省略...)

+ class ClassificationDetailUpdateSerializer(serializers.ModelSerializer):
+     """書籍分類詳細更新用シリアライザー"""
+
+     # 書籍分類コード
+     classification_code = serializers.CharField(
+         max_length=3, source="classification", label="書籍分類コード"
+     )
+
+     class Meta:
+         model = ClassificationDetail
+         fields = [
+             "classification_code",
+             "name",
+         ]
+
+     def _get_classification(self, classification_code: str) -> Classification:
+         """書籍分類コードから書籍分類モデルインスタンスを取得する。
+
+         Args:
+             classification_code: 書籍分類コード。
+         Returns:
+             書籍分類モデルインスタンス。
+         Exceptions:
+             rest_framework.exceptions.NotFound: 書籍分類が見つからない場合。
+         """
+         try:
+             return Classification.objects.get(code=classification_code)
+         except Classification.DoesNotExist:
+             raise exceptions.NotFound(detail="Classification doesn't exist")
+
+     def update(
+         self, instance: ClassificationDetail, validated_data: Any
+     ) -> ClassificationDetail:
+         """書籍分類詳細を更新する。
+
+         Args:
+             validated_data: 書籍分類詳細シリアライザーが検証したデータ。
+         Returns:
+             更新した書籍分類詳細モデルインスタンス。
+         Exceptions:
+             rest_framework.exceptions.NotFound: 書籍分類が見つからない場合。
+         """
+         validated_data["classification"] = self._get_classification(
+             validated_data["classification"]
+         )
+         return super().update(instance, validated_data)
```

書籍分類詳細更新用シリアライザーは、`ModelSerializer`を継承しています。

`ModelSerializer`は、`Meta`の`model`にモデルを指定する必要があります。
また、`fields`にシリアライザーに持たせるフィールドを指定する必要があります。
モデルのすべてのフィールドを指定したい場合は、`fields = "__all__"`を指定します。

書籍分類詳細は書籍分類名モデルフィールド（`name`）を持つため、書籍分類詳細更新用シリアライザーに`name`フィールドを追加する必要はありません。
代わりに、単に`fields`に`name`を追加します。

書籍分類詳細は書籍分類モデルフィールド（`classification`）を持つため、`fields`にそのフィールドを指定できます。
しかし、フィールド名を`classification`ではなく`classification_code`にしたいため、`fields`に指定していません。
代わりに、書籍分類詳細更新用シリアライザーに、`classification_code`フィールドを定義して、`fields`に追加しています。
なお、`ModelSerializer`で定義したフィールドは、必ず`fields`に追加する必要があります。

`ModelSerializer`は、ジェネリックにモデルを更新する`update`メソッドを実装していますが、書籍分類モデルフィールドにない`classification_code`を`fields`に追加したため、オリジナルな実装をオーバーライドして、`PUT`された`classification_code`から書籍分類モデルインスタンスを取得して、シリアライザーが検証したデータに追加しています。

書籍分類詳細更新用シリアライザーを実装したら、次の通り変更をリポジトリにコミットします。

```bash
git add ./api1/books/serializers.py
git commit -m `書籍分類詳細更新用シリアライザーを実装'
```

> 3bbd5f3 (tag: 069-implement-classification-detail-serializer-for-update)

### 書籍分類詳細シリアライザーの実装

書籍分類詳細一覧、詳細、登録及び削除用のシリアライザーを次の通り実装します。

```python
# ./api1/books/serializers.py
class ClassificationDetailSerializer(ClassificationDetailUpdateSerializer):
    """書籍分類詳細シリアライザー"""

    # 書籍分類名
    classification_name = serializers.SerializerMethodField(
        "_get_classification_name", label="書籍分類名"
    )
    # 作成日時
    created_at = serializers.DateTimeField(read_only=True)
    # 更新日時
    updated_at = serializers.DateTimeField(read_only=True)

    class Meta:
        model = ClassificationDetail
        fields = [
            "code",
            "classification_code",
            "classification_name",
            "name",
            "created_at",
            "updated_at",
        ]

    def _get_classification_name(self, obj: ClassificationDetail) -> str:
        """書籍分類名を返却する。

        Args:
            obj: 書籍分類詳細モデルインスタンス。
        Returns:
            書籍分類名。
        """
        return obj.classification.name

    def create(self, validated_data: Any) -> ClassificationDetail:
        """書籍分類詳細を登録する。

        Args:
            validated_data: 書籍分類詳細シリアライザーが検証したデータ。
        Returns:
            作成した書籍分類詳細モデルインスタンス。
        Exceptions:
            rest_framework.exceptions.NotFound: 書籍分類が見つからない場合。
        """
        validated_data["classification"] = self._get_classification(
            validated_data["classification"]
        )
        return super().create(validated_data)
```

通常の書籍分類詳細シリアライザーは、書籍分類名（`classification_name`）、作成日時（`created_at`）及び更新日時（`updated_at`）フィールドを追加しています。

書籍分類詳細名フィールドは、書籍分類詳細コードが示す書籍分類詳細名を示すために追加しています。
書籍分類名フィールドは、書籍分類詳細モデルに存在しないため、`SerializerMethodField`で定義して、書籍分類名を返却するメソッド（`_get_classification_name`）を指定しています。

また、作成日時（`created_at`）と更新日時（`updated_at`）フィールドは読み込み専用であるため、`DateTimeField`を定義しています。

通常の書籍分類詳細シリアライザーを実装したら、次の通り変更をリポジトリにコミットします。

```bash
git add ./api1/books/serializers.py
git commit -m '書籍分類詳細シリアライザーを実装'
```

> 6d720df (tag: 070-implement-classification-detail-serializer)

### 書籍分類詳細APIビューの実装

書籍分類ビューは関数ビューとして実装しました。
DRFは、[ジェネリックなクラスビュー](https://www.django-rest-framework.org/api-guide/generic-views/)としていくつか提供しています。

書籍分類詳細ビューは、クラスビューの[ListCreateAPIView](https://www.django-rest-framework.org/api-guide/generic-views/#listcreateapiview)を継承した書籍分類詳細一覧登録ビューと、[RetrieveUpdateDestroyAPIView](https://www.django-rest-framework.org/api-guide/generic-views/#retrieveupdatedestroyapiview)を継承した書籍分類詳細更新削除ビューで実装します。

```python
# ./api1/books/views.py
- from rest_framework import status
+ from rest_framework import generics, serializers, status
  from rest_framework.decorators import api_view
  from rest_framework.request import Request
  from rest_framework.response import Response

- from books.models import Classification
+ from books.models import Classification, ClassificationDetail

- from .serializers import ClassificationSerializer
+ from .serializers import (
+     ClassificationDetailSerializer,
+     ClassificationDetailUpdateSerializer,
+     ClassificationSerializer,
+ )

  (...省略...)

class ClassificationDetailListCreateView(generics.ListCreateAPIView):
    """書籍分類詳細一覧登録ビュー"""

    queryset = ClassificationDetail.objects.all()
    serializer_class = ClassificationDetailSerializer


class ClassificationRetrieveUpdateDestroyView(generics.RetrieveUpdateDestroyAPIView):
    """書籍分類詳細更新削除ビュー"""

    queryset = ClassificationDetail.objects.all()
    serializer_class = ClassificationDetailSerializer
    lookup_field = "code"

    def get_serializer_class(self) -> serializers.Serializer:
        if self.request.method.lower() in ("put", "patch"):
            return ClassificationDetailUpdateSerializer
        return super().get_serializer_class()
```

DRFのクラスビューでは、クエリセットとシリアライザーを定義する必要があります。
なお、クエリセットは、通常、そのビューで表示するモデルインスタンスを含むクエリセットを設定します。

書籍分類詳細一覧登録ビューは、書籍分類詳細シリアライザーをシリアライザーのクラスとして設定しています。
書籍分類詳細更新削除ビューも同様に、書籍分類詳細シリアライザーを設定していますが、`get_serializer_class`メソッドをオーバーライドすることで、リクエストが`POST`または`PATCH`メソッドの場合は、書籍分類詳細更新用シリアライザーを使用するようにしています。

また、書籍分類詳細更新削除ビューは、後でこのビューをディスパッチするときにパスコンバーターで指定した`code`で書籍分類詳細モデルインスタンスを取得するように設定（`lookup_field = "code"`）しています。
なお、`lookup_field`を指定しない時のデフォルトは`"pk"`です。

書籍分類詳細APIビューを実装したら、次の通り変更をリポジトリにコミットします。

```bash
git add ./api1/books/views.py
git commit -m '書籍分類詳細ビューを実装'
```

> 739190e (tag: 071-implement-classification-detail-views)

### 書籍分類詳細APIビューのディスパッチ

書籍分類詳細APIビューを次の通りディスパッチします。

```python
  urlpatterns = [
      path("books/classifications/", views.classification_list),
      path("books/classifications/<str:code>/", views.classification_detail),
+     path(
+         "books/classification-details/",
+         views.ClassificationDetailListCreateView.as_view(),
+     ),
+     path(
+         "books/classification-details/<str:code>/",
+         views.ClassificationRetrieveUpdateDestroyView.as_view(),
+     ),
  ]
```

書籍分類詳細APIビューをディスパッチしたら、次の通り変更をリポジトリにコミットします。

```bash
git add api1/urls.py
git commit -m '書籍分類詳細APIを実装'
```

> edc65fc (tag: 072-implement-classification-detail-api)

### 書籍分類詳細（`ClassificationDetail`）APIの呼び出し

#### 書籍分類詳細一覧APIの呼び出し

書籍分類詳細一覧APIを次の通り呼び出します。

```bash
curl -s http://localhost:8000/api1/books/classification-details/ | jq .
# [
#   {
#     "code": "000",
#     "classification_code": "000",
#     "classification_name": "総記",
#     "name": "総記",
#     "created_at": "2023-04-24T09:00:00+09:00",
#     "updated_at": "2023-04-24T09:00:00+09:00"
#   },
#   {
#     "code": "010",
#     "classification_code": "000",
#     "classification_name": "総記",
#     "name": "図書館、図書館情報学",
#     "created_at": "2023-04-24T09:00:00+09:00",
#     "updated_at": "2023-04-24T09:00:00+09:00"
#   },
#   {
#     "code": "020",
#     "classification_code": "000",
#     "classification_name": "総記",
#     "name": "図書、書誌学",
#     "created_at": "2023-04-24T09:00:00+09:00",
#     "updated_at": "2023-04-24T09:00:00+09:00"
#   },
#   {
#     "code": "030",
#     "classification_code": "000",
#     "classification_name": "総記",
#     "name": "百科事典、用語索引",
#     "created_at": "2023-04-24T09:00:00+09:00",
#     "updated_at": "2023-04-24T09:00:00+09:00"
#   },
# (...省略...)
#   {
#     "code": "990",
#     "classification_code": "900",
#     "classification_name": "文学",
#     "name": "その他の諸言語文学",
#     "created_at": "2023-04-24T09:00:00+09:00",
#     "updated_at": "2023-04-24T09:00:00+09:00"
#   }
# ]
```

#### 書籍分類詳細詳細APIの呼び出し

書籍分類詳細詳細APIを次の通り呼び出します。

```bash
curl -s http://localhost:8000/api1/books/classification-details/110/ | jq .
# {
#   "code": "110",
#   "classification_code": "100",
#   "classification_name": "哲学",
#   "name": "哲学各論",
#   "created_at": "2023-04-24T09:00:00+09:00",
#   "updated_at": "2023-04-24T09:00:00+09:00"
# }
```

#### 書籍分類詳細登録APIの呼び出し

書籍分類詳細登録APIを次の通り呼び出します。

```bash
curl -X POST -H "Content-Type: application/json" -s -d '{"code": "999", "classification_code": "100", "name": "ダミー書籍分類詳細"}' http://localhost:8000/api1/books/classification-details/ | jq .
# {
#   "code": "999",
#   "classification_code": "100",
#   "classification_name": "哲学",
#   "name": "ダミー書籍分類詳細",
#   "created_at": "2023-05-02T19:52:40.354312+09:00",
#   "updated_at": "2023-05-02T19:52:40.354326+09:00"
# }
```

#### 書籍分類詳細更新APIの呼び出し

書籍分類詳細更新APIを次の通り呼び出します。

```bash
curl -X PATCH -H "Content-Type: application/json" -s -d '{"classification_code": "200", "name": "更新後のダミー書籍分類詳細"}' http://localhost:8000/api1/books/classification-details/999/ | jq .
# {
#   "classification_code": "200",
#   "name": "更新後のダミー書籍分類詳細"
# }
```

### 書籍分類詳細削除APIの呼び出し

書籍分類詳細削除APIを次の通り呼び出します。

```bash
curl -X DELETE -i -s http://localhost:8000/api1/books/classification-details/999/ | grep HTTP
# HTTP/1.1 204 No Content
```

## 書籍APIの実装

書籍分類詳細APIは、DRFが適用する`ジェネリックなクラスビュー`で実装しました。
書籍APIは、書籍分類詳細APIの実装を一通り持っている[rest_framework.viewsets.https://www.django-rest-framework.org/api-guide/viewsets/#modelviewset]で実装します。

### 書籍シリアライザーの実装

書籍APIでは、書籍一覧、詳細、削除APIで使用する書籍読み込み専用シリアライザーと、書籍登録、更新APIで使用する書籍書き込み専用シリアライザーの2つのシリアライザーを実装します。

#### 書籍読み込み専用シリアライザーの実装

書籍読み込み専用シリアライザーを次の通り実装します。

```python
# ./api1/books/serializers.py
  from rest_framework import exceptions, serializers

- from books.models import Classification, ClassificationDetail
+ from books.models import Book, Classification, ClassificationDetail
+ from divisions.models import Division

  (...省略...)

+ class ClassificationReadOnlySerializer(serializers.ModelSerializer):
+     """書籍分類モデル読み込み専用シリアライザー"""
+
+     class Meta:
+         model = Classification
+         fields = (
+             "code",
+             "name",
+         )
+
+
+ class ClassificationDetailReadOnlySerializer(serializers.ModelSerializer):
+     """書籍分類詳細モデル読み込み専用シリアライザー"""
+
+     classification = ClassificationReadOnlySerializer(label="書籍分類")
+
+     class Meta:
+         model = Classification
+         fields = (
+             "code",
+             "classification",
+             "name",
+         )
+
+
+ class DivisionReadOnlySerializer(serializers.ModelSerializer):
+     """部署モデルシリアライザー"""
+
+     class Meta:
+         model = Division
+         fields = (
+             "code",
+             "name",
+         )
+
+
+ class BookReadOnlySerializer(serializers.ModelSerializer):
+     """書籍シリアライザー"""
+
+     # 書籍ID
+     id = serializers.CharField(label="書籍ID")
+     # 書籍分類詳細
+     classification_detail = ClassificationDetailReadOnlySerializer(label="書籍分類詳細")
+     # 管理部署
+     division = DivisionReadOnlySerializer(label="管理部署")
+
+     class Meta:
+         model = Book
+         fields = (
+             "id",
+             "title",
+             "classification_detail",
+             "authors",
+             "isbn",
+             "publisher",
+             "published_at",
+             "division",
+             "disposed",
+             "disposed_at",
+             "created_at",
+             "updated_at",
+         )
```

書籍読み込み専用シリアライザーを実装したら、次の通り変更をリポジトリにコミットします。

```bash
git add ./api1/books/serializers.py
git commit -m '書籍読み込み専用シリアライザーを実装'
```

> b8044a6 (tag: 073-implement-book-read-only-serializer)

#### 書籍書き込み専用シリアライザーの実装

書籍書き込み専用シリアライザーを次の通り実装します。

```python
# ./api1/books/serializers.py
class BookWriteOnlySerializer(serializers.ModelSerializer):
    """書籍書き込み専用シリアライザー"""

    # 書籍ID
    id = serializers.CharField(max_length=26, read_only=True)
    # 書籍分類詳細コード
    classification_detail_code = serializers.CharField(
        max_length=3, source="classification_detail", label="書籍分類詳細コード"
    )
    # 管理部署コード
    division_code = serializers.CharField(
        max_length=2, source="division", label="管理部署コード"
    )

    class Meta:
        model = Book
        fields = (
            "id",
            "title",
            "classification_detail_code",
            "authors",
            "isbn",
            "publisher",
            "published_at",
            "division_code",
            "disposed",
            "disposed_at",
        )

    def _get_classification_detail(self, code: str) -> ClassificationDetail:
        """書籍分類詳細コードから書籍分類詳細モデルインスタンスを取得して返却する。

        Args:
            code: 書籍分類詳細コード。
        Returns:
            書籍分類詳細モデルインスタンス。
        Exceptions:
            rest_framework.exceptions.NotFound: 書籍分類詳細が見つからない場合。
        """
        try:
            return ClassificationDetail.objects.get(code=code)
        except ClassificationDetail.DoesNotExist:
            raise exceptions.NotFound(detail="Classification detail doesn't exist")

    def _get_division(self, code: str) -> Division:
        """部署コードから部署モデルインスタンスを取得して返却する。

        Args:
            code: 部署コード。
        Returns:
            部署モデルインスタンス。
        Exceptions:
            rest_framework.exceptions.NotFound: 部署が見つからない場合。
        """
        try:
            return Division.objects.get(code=code)
        except Division.DoesNotExist:
            raise exceptions.NotFound(detail="Division doesn't exist")

    def _organize_validated_data(self, validated_data: Any) -> Any:
        """書籍書き込みシリアライザーが検証したデータを整理する。

        書籍書き込みシリアライザーが検証したデータに、書籍分類詳細及び部署モデルインスタンスを設定する。

        Args:
            validated_data: 書籍書き込みシリアライザーが検証したデータ。
        Returns:
            書籍書き込みシリアライザーが検証したデータを整理した結果。
        Exceptions:
            rest_framework.exceptions.NotFound: 書籍分類詳細または部署が見つからない場合。
        """
        validated_data["classification_detail"] = self._get_classification_detail(
            validated_data["classification_detail"]
        )
        validated_data["division"] = self._get_division(validated_data["division"])
        return validated_data

    def create(self, validated_data: Any) -> Book:
        """書籍を登録する。

        Args:
            validated_data: 書籍書き込み専用シリアライザーが検証したデータ。
        Returns:
            作成した書籍モデルインスタンス。
        Exceptions:
            rest_framework.exceptions.NotFound: 書籍分類詳細または部署が見つからない場合。
        """
        return super().create(self._organize_validated_data(validated_data))

    def update(self, instance: Any, validated_data: Any) -> Book:
        """書籍を更新する。

        Args:
            validated_data: 書籍書き込み専用シリアライザーが検証したデータ。
        Returns:
            更新した書籍モデルインスタンス。
        Exceptions:
            rest_framework.exceptions.NotFound: 書籍分類詳細または部署が見つからない場合。
        """
        return super().update(instance, self._organize_validated_data(validated_data))
```

書籍書き込み専用シリアライザーを実装したら、次の通り変更をリポジトリにコミットします。

```bash
git add ./api1/books/serializers.py
git commit -m '書籍書き込み専用シリアライザーを実装'
```

> 5229a06 (tag: 074-implement-book-write-only-serializer)

### 書籍ビューセットの実装

次の通り書籍ビューセット（`BookViewSet`）を実装します。

```python
# ./api1/books/views.py
- from rest_framework import generics, serializers, status
+ from rest_framework import generics, serializers, status, viewsets
  from rest_framework.decorators import api_view
  from rest_framework.request import Request
  from rest_framework.response import Response

- from books.models import Classification, ClassificationDetail
+ from books.models import Book, Classification, ClassificationDetail

- from .serializers import (ClassificationDetailSerializer,
-                           ClassificationDetailUpdateSerializer,
-                           ClassificationSerializer)
+ from .serializers import (BookReadOnlySerializer, BookWriteOnlySerializer,
+                           ClassificationDetailSerializer,
+                           ClassificationDetailUpdateSerializer,
+                           ClassificationSerializer)

  (...省略...)

+ class BookViewSet(viewsets.ModelViewSet):
+     """書籍ビューセット"""
+
+     queryset = Book.objects.all()
+     serializer_class = BookReadOnlySerializer
+
+     def get_serializer_class(self) -> serializers.Serializer:
+         """書籍シリアライザークラスを返却する。
+
+         Returns:
+             書籍シリアライザー。
+         """
+         if self.request.method.lower() in ("post", "put", "patch", "delete"):
+             return BookWriteOnlySerializer
+         return BookReadOnlySerializer
```

`get_serializer_class`メソッドをオーバーライドして、HTTPリクエストメソッドが`POST（登録）`、`PUT（更新）`、`PATCH（一部更新）`及び`DELETE（削除）`の場合は、書籍書き込み専用シリアライザーを使用して、`GET（一覧、詳細）`の場合は書籍読み込み専用シリアライザーを使用するようにしています。

書籍ビューセットを実装したら、次の通り変更をリポジトリにコミットします。

```bash
git add api1/books/views.py
git commit -m '書籍ビューセットを実装'
```

> bd2ef49 (tag: 075-implement-book-viewset)

### 書籍ビューセットのディスパッチ

次の通り書籍ビューセットをディスパッチします。

```python
  from django.urls import path
+ from rest_framework.routers import DefaultRouter

  from .books import views

  urlpatterns = [
      (...省略...)
  ]
+
+
+ # 書籍ビューセットをディスパッチ
+ router = DefaultRouter()
+ router.register("books", viewset=views.BookViewSet)
+ urlpatterns += router.urls
```

[rest_framework.routers.DefaultRouter](https://www.django-rest-framework.org/api-guide/routers/#defaultrouter)は、書籍分類または書籍分類詳細APIのようなURLをビューセットのために生成します。

`urlpatterns += router.urls`で、`DefaultRouter`が生成するURLを`urlpatterns`に追加することで、書籍ビューセットをディスパッチしています。

書籍ビューセットをディスパッチしたら、次の通り変更をリポジトリにコミットします。

```bash
git add ./api1/urls.py
git commit -m '書籍ビューセットをディスパッチ'
```

> aef3213 (tag: 076-dispatch-book-viewset)

## 書籍APIの保護

書籍登録、更新及び削除APIの呼び出しを、認証済みユーザーのみ許可します。
認証済みユーザーであることを判別するために[JWT (JSON Web Token)](https://jwt.io/)を利用します。

### JWT (JSON Web Token)とは

JWT（JSON Web Token）は、WebアプリケーションやAPIで認証や認可を行うための安全な方法の1つです。
JWTは、JSON形式で表現されるトークンであり、ユーザーの認証情報やアプリケーションへのアクセス許可などの情報を含んでいます。

### 認証API

#### Simple JWTのインストール

DRFでJWTを生成するために[Simple JWT](https://django-rest-framework-simplejwt.readthedocs.io/en/latest/index.html)パッケージを次の通りインストールします。

```bash
pip install djangorestframework-simplejwt
pip freeze > requirements.txt
git add requirements.txt
git commit -m 'Simple JWTパッケージをインストール'
```

> 0c5d58d (tag: 077-install-simple-jwt)

#### Simple JWTの設定

プロジェクト設定ファイルを次の通り変更します。

```python
# ./book_management/settings.py
  import os
+ from datetime import timedelta
  from pathlib import Path
  from typing import List

  (...省略...)

  INSTALLED_APPS = [
      "django.contrib.admin",
      "django.contrib.auth",
      "django.contrib.contenttypes",
      "django.contrib.sessions",
      "django.contrib.messages",
      "django.contrib.staticfiles",
      "rest_framework",
+     "rest_framework_simplejwt",
      "django_bootstrap5",
      "debug_toolbar",
      "accounts.apps.AccountsConfig",
      "divisions.apps.DivisionsConfig",
      "books.apps.BooksConfig",
      "api1.apps.Api1Config",
  ]

  (...省略..)
+ # Django REST Framework
+ REST_FRAMEWORK = {
+     "DEFAULT_AUTHENTICATION_CLASSES": (
+         "rest_framework_simplejwt.authentication.JWTAuthentication",
+     )
+ }
+
+ # Simple JWT
+ SIMPLE_JWT = {
+     "REFRESH_TOKEN_LIFETIME": timedelta(hours=1),
+     "USER_ID_FIELD": "email",
+ }
```

認証APIは、アクセストークンとリフレッシュトークンを返却します。
アクセストークンは、認証済みユーザーのみがアクセス可能なページをリクエストするときに、リクエストヘッダに追加します。
また、リフレッシュトークンは、アクセストークンの有効期間が過ぎたときに、有効なアクセストークンを取得するときに使用します。
リフレッシュトークンの有効期間が過ぎた場合は、再度アクセストークンとリフレッシュトークンを取得する必要があります。
このため、リフレッシュートークンの有効期間は、アクセストークンより長くなっています。
`Simple JWT`パッケージのデフォルトの有効期間は次の通りです。

- アクセストークン: 5分
- リフレッシュトークン: 1日

本チュートリアルでは、アクセストークンの有効期間を1時間に変更しています。

また、本Webアプリケーションのユーザーは`id`モデルフィールドがないため、`Simple JWT`が`email`モデルフィールドをユーザーIDモデルフィールドとして認識するように、`"USER_ID_FIELD": "email"`を設定しています。

`Simple JWT`を設定したら、次の通り変更をリポジトリにコミットします。

```bash
git add ./book_management/settings.py
git commit -m 'Simple JWTを設定'
```

> c378e2a (tag: 078-configure-simple-jwt)

#### 認証APIの実装

次の通り認証APIを実装します。

```python
# ./api1/urls.py
  from django.urls import path
  from rest_framework.routers import DefaultRouter
+ from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView

  from .books import views

  urlpatterns = [

      (...省略...)

+     path("auth/token/", TokenObtainPairView.as_view()),
+     path("auth/token/refresh/", TokenRefreshView.as_view()),
  ]
```

```bash
git add ./api1/urls.py
git commit -m '認証APIを実装'
```

> d987dc1 (tag: 079-implement-authentication-api)

### 書籍登録、更新及び削除APIの保護

認証済みユーザーのみ書籍登録、更新及び削除APIを呼び出せるように、次の通り書籍ビューセットに[パーミッション（権限）](https://www.django-rest-framework.org/api-guide/permissions/)を設定します。
なお、パーミッションは、DRFが提供する以外にも、独自のパーミッションを定義してビューになどに設定できます。

```bash
# ./api1/books/views.py
- from rest_framework import generics, serializers, status, viewsets
+ from rest_framework import generics, permissions, serializers, status, viewsets
  from rest_framework.decorators import api_view
  from rest_framework.request import Request
  from rest_framework.response import Response

  (...省略...)

  class BookViewSet(viewsets.ModelViewSet):

      queryset = Book.objects.all()
      serializer_class = BookReadOnlySerializer
+     permission_classes = [
+         permissions.IsAuthenticatedOrReadOnly,
+     ]

      def get_serializer_class(self) -> serializers.Serializer:
```

DRFのクラスビューやビューセットのクラス変数`permission_classes`の設定や、`get_permission_classes`メソッドをオーバーライドすることで、APIのビューを保護できます。

ここでは、`permissions.IsAuthenticatedOrReadOnly`パーミッションを設定することで、認証済みユーザーから`POST`、`PUT`、`PATCH`及び`DELETE`メソッド（`安全でないメソッド`）で、認証されていないユーザーから`GET`、`HEAD`及び`OPTIONS`メソッド（`安全なメソッド`）で送信されたリクエストを受け付けます。
また、認証されていないユーザーから`安全でないメソッド`で送信されたリクエストを拒否して、`401 Unauthorized`を返却します。

書籍ビューセットにパーミッションを設定したら、次の通り変更をコミットします。

```bash
git add ./api1/books/views.py
git commit -m '書籍ビューセットにパーミッションを設定'
```

> cd3bcea (tag: 080-set-permission-to-book-viewset)

### 書籍API保護の確認

#### 認証APIの呼び出し

次の通り認証APIを呼び出します。

<!-- cspell: disable -->
```bash
curl -X POST -H "Content-Type: application/json" -s -d '{"email": "spam@example.com", "password": "django"}
' http://localhost:8000/api1/auth/token/ | jq .
# {
#   "refresh": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0b2tlbl90eXBlIjoicmVmcmVzaCIsImV4cCI6MTY4MzA5Mzg0MSwiaWF0IjoxNjgzMDkwMjQxLCJqdGkiOiJmNTYzOGE2MjhkY2M0MDVjYmQxZDg0OWM2NGJkMTFjNSIsInVzZXJfaWQiOiJzcGFtQGV4YW1wbGUuY29tIn0.rWjDZWPMskMsOkGWVwU_P90c3Cu-RWIJ9XKq62TllKs",
#   "access": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIiwiZXhwIjoxNjgzMDkwNTQxLCJpYXQiOjE2ODMwOTAyNDEsImp0aSI6IjQ4MDFmM2I4MDk4NDQwZmRiYzE4NjYwZmRjMzY2MDRiIiwidXNlcl9pZCI6InNwYW1AZXhhbXBsZS5jb20ifQ.YjsgfL4mzeG0b8Yb4MfOMOxYxGC6iAFGdymHqiY4Fhk"
# }
```
<!-- cspell: enable -->

#### 書籍一覧及び詳細APIの呼び出し

アクセストークンなしで、書籍一覧及び詳細APIを呼び出せることを確認します。

<!-- cspell: disable -->
```bash
curl -s http://localhost:8000/api1/books/
# [{"id":"01GYV46C5KXWDRKMB1WR3TW6RK","title":"Fluent Python","classification_detail":{"code":"000","classification":{"code":"000","name":"総記"},"name":"総記"},"authors":"（著）Luciano Ramalho\r\n（監修）豊沢　聡、桑井　博之\r\n（訳）梶原　玲子","isbn":"ISBN978-4-87311-817-8","publisher":"オライリー・ジャパン","published_at":"2017-10-11","division":{"code":"58","name":"ICT開発室"},"disposed":false,"disposed_at":null,"created_at":"2023-04-25T09:00:00+09:00","updated_at":"2023-04-25T09:00:00+09:00"},{"id":"01GYV585X4JNDCSVA3BY93KN54","title":"プログラミングRust","classification_detail":{"code":"000","classification":{"code":"000","name":"総記"},"name":"総記"},"authors":"（著）Jim Blandy、Jason Orendorff\n（訳）中田　秀基","isbn":"ISBN978-4-87311-855-0","publisher":"オライリー・ジャパン","published_at":"2018-08-08","division":{"code":"58","name":"ICT開発室"},"disposed":false,"disposed_at":null,"created_at":"2023-04-25T09:00:00+09:00","updated_at":"2023-04-25T09:00:00+09:00"},{"id":"01GYZYA8NS02G8NWCFWBZ6KS9T","title":"プロを目指す人のためのTypeScript入門","classification_detail":{"code":"000","classification":{"code":"000","name":"総記"},"name":"総記"},"authors":"鈴木  僚太","isbn":"ISBN978-4-297-12747-3","publisher":"技術評論社","published_at":"2022-05-05","division":{"code":"58","name":"ICT開発室"},"disposed":false,"disposed_at":null,"created_at":"2023-04-25T09:00:00+09:00","updated_at":"2023-04-25T09:00:00+09:00"}]

curl -s http://localhost:8000/api1/books/01GYZYA8NS02G8NWCFWBZ6KS9T/
# {"id":"01GYZYA8NS02G8NWCFWBZ6KS9T","title":"プロを目指す人のためのTypeScript入門","classification_detail":{"code":"000","classification":{"code":"000","name":"総記"},"name":"総記"},"authors":"鈴木  僚太","isbn":"ISBN978-4-297-12747-3","publisher":"技術評論社","published_at":"2022-05-05","division":{"code":"58","name":"ICT開発室"},"disposed":false,"disposed_at":null,"created_at":"2023-04-25T09:00:00+09:00","updated_at":"2023-04-25T09:00:00+09:00"}
```

#### アクセストークンを付けて書籍登録APIを呼び出し

アクセストークンを付けて、次の通り書籍登録APIを呼び出せることを確認します。

```bash
curl -X POST -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIiwiZXhwIjoxNjgzMDkxNjQ3LCJpYXQiOjE2ODMwOTEzNDcsImp0aSI6IjE2MGUzMDM2NTk4MjQ4MWE4ZDAxMmZiOWQ0YTA4NDlmIiwidXNlcl9pZCI6InNwYW1AZXhhbXBsZS5jb20ifQ.NcO0N6G8Z-lzTRWY5WCfv-XdxGV8F1lGQ3gNBw1ei0U" -i -s -d '{"title": "ハンズオンWebAssembly", "classification_detail_code": "000", "authors": "（著）Gerard Gallant\r\n（訳）北原　憲、洲崎　俊、西谷　完太、磯野　亘平、米内　貴志", "isbn": "ISBN978-4-8144-0010-2", "publisher": "オライリー・ジャパン", "published_at": "2022-10-07", "division_code": "58"}' http://localhost:8000/api1/books/
# {"id":"01GZG10ZM1DJKAAVA8J404EB48","title":"ハンズオンWebAssembly","classification_detail_code":"総記","authors":"（著）Gerard Gallant\r\n（訳）北原　憲、洲崎　俊、西谷　完太、磯野　亘平、米内　貴志","isbn":"ISBN978-4-8144-0010-2","publisher":"オライリー・ジャパン","published_at":"2022-10-07","division_code":"ICT開発室","disposed":false,"disposed_at":null}%
```

#### アクセストークンなしで書籍登録APIを呼び出し

アクセストークンなしで、書籍登録APIを呼び出せないことを確認します。

```bash
curl -X POST -H "Content-Type: application/json" -i -s -d '{"title": "ハンズオンWebAssembly", "classification_detail_code": "000", "authors": "（著）Gerard Gallant\r\n（訳）北原　憲、洲崎　俊、西谷　完太、磯野　亘平、米内　貴志", "isbn": "ISBN978-4-8144-0010-2", "publisher": "オライリー・ジャパン", "published_at": "2022-10-07", "division_code": "58"}' http://localhost:8000/api1/books/ | grep HTTP
# HTTP/1.1 401 Unauthorized
```

#### アクセストークンなしで書籍更新APIを呼び出し

アクセストークンなしで、書籍更新APIを呼び出せないことを確認します。

<!-- cspell: enable -->
```bash
curl -X PUT -H "Content-Type: application/json" -i -s -d '{"title": "ハンズオンWebAssembly", "classification_detail_code": "000", "authors": "（著）Gerard Gallant\r\n（訳）北原　憲、洲崎　俊、西谷　完太、磯野　亘平、米内　貴志", "isbn": "ISBN978-4-8144-0010-2", "publisher": "オライリー・ジャパン", "published_at": "2022-10-07", "division_code": "58"}' http://localhost:8000/api1/books/01GZG10ZM1DJKAAVA8J404EB48/ | grep HTTP
# HTTP/1.1 401 Unauthorized
```
<!-- cspell: enable -->

#### アクセストークンなしで書籍削除APIを呼び出し

アクセストークンなしで、書籍削除APIを呼び出せないことを確認します。

```bash
curl -X DELETE -H "Content-Type: application/json" -i -s http://localhost:8000/api1/books/01GZG10ZM1DJKAAVA8J404EB48/ | grep HTTP
# HTTP/1.1 401 Unauthorized
```

#### アクセストークンを付けて書籍更新APIを呼び出し

アクセストークンを付けて、書籍更新APIを呼び出せることを確認します。

<!-- cspell: disable -->
```bash
curl -X PUT -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIiwiZXhwIjoxNjgzMDkzMzU4LCJpYXQiOjE2ODMwOTMwNTgsImp0aSI6IjAzODE4MjIwNmYwZTRiNmI5ZTNjMTQyMThjZDA3Y2QzIiwidXNlcl9pZCI6InNwYW1AZXhhbXBsZS5jb20ifQ.G4tT8xX05ee1DnITw0kZuMuSjFNUHK5TE6EOntp-0PI" -s -d '{"title": "ハンズオンWebAssembly", "classification_detail_code": "000", "authors": "（著）Gerard Gallant\r\n（訳）北原　憲、洲崎　俊、西谷　完太、磯野　亘平、米内　貴志", "isbn": "ISBN978-4-8144-0010-2", "publisher": "オライリー・ジャパン", "published_at": "2022-10-07", "division_code": "58"}' http://localhost:8000/api1/books/01GZG10ZM1DJKAAVA8J404EB48/
# {"id":"01GZG10ZM1DJKAAVA8J404EB48","title":"ハンズオンWebAssembly","classification_detail_code":"総記","authors":"（著）Gerard Gallant\r\n（訳）北原　憲、洲崎　俊、西谷　完太、磯野　亘平、米内　貴志","isbn":"ISBN978-4-8144-0010-2","publisher":"オライリー・ジャパン","published_at":"2022-10-07","division_code":"ICT開発室","disposed":false,"disposed_at":null}
```
<!-- cspell: enable -->

#### アクセストークンを付けて書籍削除APIを呼び出し

アクセストークンを付けて、書籍削除APIを呼び出せることを確認します。

<!-- cspell: disable -->
```bash
curl -X DELETE -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIiwiZXhwIjoxNjgzMDkzNzQ4LCJpYXQiOjE2ODMwOTM0NDgsImp0aSI6IjgxNWRmMDA3ZTA0NjRkNzRhN2FjMzFlZjcyYzRkMDM4IiwidXNlcl9pZCI6InNwYW1AZXhhbXBsZS5jb20ifQ.KY_6wUz-qlB7sg6GsTlk1P5_8Q0pGRm9p41BH9C5dNo" -s -i http://localhost:8000/api1/books/01GZG10ZM1DJKAAVA8J404EB48/ | grep HTTP
# HTTP/1.1 204 No Content
```
