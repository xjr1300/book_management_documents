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
  - [Django REST Frameworkによる書籍アプリモデルを操作するAPIの実装](#django-rest-frameworkによる書籍アプリモデルを操作するapiの実装)
    - [シリアライザー (Serializer)](#シリアライザー-serializer)
    - [書籍分類APIの実装](#書籍分類apiの実装)
      - [書籍分類シリアライザーの実装](#書籍分類シリアライザーの実装)

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

```bash
pip install djangorestframework
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

> fa70ef6 (tag: 064-install-django-rest-framework)

## 書籍APIアプリの追加

書籍アプリのモデルを操作するアプリを次の通り追加します。

```bash
python manage.py startapp api1
```

アプリ名の`api1`の末尾の`1`は、APIのバージョンを示しています。
本チュートリアルでAPIをバージョン管理しませんが、APIをバージョニングするためにバージョン番号をつける事例があります。

次の通り変更をリポジトリにコミットします。

```bash
git add ./api1
git commit -m 'api1アプリを追加'
```

> ab08a80 (tag: 065-add-api1-app)

