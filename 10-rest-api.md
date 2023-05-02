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
      - [書籍分類詳細APIの呼び出し](#書籍分類詳細apiの呼び出し)
      - [書籍分類登録APIの呼び出し](#書籍分類登録apiの呼び出し)
      - [書籍分類更新APIの呼び出し](#書籍分類更新apiの呼び出し)
      - [書籍分類削除APIの呼び出し](#書籍分類削除apiの呼び出し)
  - [書籍分類詳細APIの実装](#書籍分類詳細apiの実装)
    - [書籍分類詳細更新用シリアライザーの実装](#書籍分類詳細更新用シリアライザーの実装)
    - [書籍分類詳細シリアライザーの実装](#書籍分類詳細シリアライザーの実装)
    - [書籍分類詳細APIビューの実装](#書籍分類詳細apiビューの実装)
    - [書籍分類詳細APIビューのディスパッチ](#書籍分類詳細apiビューのディスパッチ)
    - [書籍分類詳細APIの呼び出し](#書籍分類詳細apiの呼び出し-1)
      - [書籍分類詳細一覧APIの呼び出し](#書籍分類詳細一覧apiの呼び出し)
      - [書籍分類詳細詳細APIの呼び出し](#書籍分類詳細詳細apiの呼び出し)
      - [書籍分類詳細登録APIの呼び出し](#書籍分類詳細登録apiの呼び出し)
      - [書籍分類詳細更新APIの呼び出し](#書籍分類詳細更新apiの呼び出し)
    - [書籍分類詳細削除APIの呼び出し](#書籍分類詳細削除apiの呼び出し)

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
pip install djangorestframework
```
<!-- cspell: enable -->

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

> fa70ef6 (tag: 064-install-django-rest-framework)

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

> c890271 (tag: 065-add-api1-app)

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

> d6e4e58 (tag: 066-implement-classification-serializer)

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

> ac8e845 (tag: 067-implement-classification-views)

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

> dd540f7 (tag: 068-implement-classification-api)

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

#### 書籍分類詳細APIの呼び出し

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
+         max_length=3, source="classification.code", label="書籍分類コード"
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
+             raise exceptions.NotFound(detail="classification doesn't found")
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
+         classification = self._get_classification(
+             validated_data["classification"]["code"]
+         )
+         validated_data["classification"] = classification
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

> d00ead3 (tag: 069-implement-classification-detail-serializer-for-update)

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
        classification = self._get_classification(
            validated_data["classification"]["code"]
        )
        validated_data["classification"] = classification
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

> 03c104c (tag: 070-implement-classification-detail-serializer)

### 書籍分類詳細APIビューの実装

書籍分類ビューは関数ビューとして実装しました。
DRFは、[ジェネリックなクラスビュー](https://www.django-rest-framework.org/api-guide/generic-views/)としていくつか提供しています。

書籍分類詳細ビューは、クラスビューの[ListCreateAPIView](https://www.django-rest-framework.org/api-guide/generic-views/#listcreateapiview)を継承した書籍分類詳細一覧登録ビューと、[RetrieveUpdateDestroyAPIView](https://www.django-rest-framework.org/api-guide/generic-views/#retrieveupdatedestroyapiview)を継承した書籍分類詳細詳細更新削除ビューで実装します。

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
書先分類詳細詳細更新削除ビューも同様に、書籍分類詳細シリアライザーを設定していますが、`get_serializer_class`メソッドをオーバーライドすることで、リクエストが`POST`または`PATCH`メソッドの場合は、書籍分類詳細更新用シリアライザーを使用するようにしています。

また、書籍分類詳細更新削除ビューは、後でこのビューをディスパッチするときにパスコンバーターで指定した`code`で書籍分類詳細モデルインスタンスを取得するように設定（`lookup_field = "code"`）しています。
なお、`lookup_field`を指定しない時のデフォルトは`"pk"`です。

書籍分類詳細APIビューを実装したら、次の通り変更をリポジトリにコミットします。

```bash
git add ./api1/books/views.py
git commit -m '書籍分類詳細ビューを実装'
```

> 445a44b (tag: 071-implement-classification-detail-views)

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

> 4c42754 (tag: 072-implement-classification-detail-api)

### 書籍分類詳細APIの呼び出し

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
