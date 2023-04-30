# Bootstrapで書籍ページの装飾とレイアウト

- [Bootstrapで書籍ページの装飾とレイアウト](#bootstrapで書籍ページの装飾とレイアウト)
  - [django-bootstrap5のインストール](#django-bootstrap5のインストール)
  - [Bootstrap用のベーステンプレートの作成](#bootstrap用のベーステンプレートの作成)
  - [書籍ベースページテンプレートの実装](#書籍ベースページテンプレートの実装)
  - [書籍分類リンクテンプレートにBootstrapを適用](#書籍分類リンクテンプレートにbootstrapを適用)
  - [書籍一覧テンプレートにBootstrapを適用](#書籍一覧テンプレートにbootstrapを適用)
  - [書籍詳細表示テンプレートにBootstrapを的章](#書籍詳細表示テンプレートにbootstrapを的章)
  - [書籍詳細テンプレートにBootstrapを適用](#書籍詳細テンプレートにbootstrapを適用)
  - [書籍フォームテンプレートにBootstrapを適用](#書籍フォームテンプレートにbootstrapを適用)
  - [書籍削除テンプレートにBootstrapを適用](#書籍削除テンプレートにbootstrapを適用)
  - [まとめ](#まとめ)

[Bootstrap](https://getbootstrap.jp/docs/5.0/getting-started/introduction/)は、Twitter社（現X社）が開発した、ウェブサイトやWebアプリケーションを作成するフロントエンドWebアプリケーションフレームワークです。
Bootstrapは、タイポグラフィ、フォーム、ボタン、ナビゲーションなどの部品を、HTML、CSS及びJavaScriptで提供しています。

また、主にBootstrapのフォーム部品をDjangoで利用することを支援する[django-bootstrap5](https://django-bootstrap5.readthedocs.io/en/latest/index.html)パッケージがあります。

本章では、Bootstrap5とdjango-bootstrap5を使用して、書籍ページを装飾してレイアウトします。

Bootstrapについての説明は[ここ](https://getbootstrap.jp/docs/5.0/getting-started/introduction/)を参照してください。
また、django-bootstrap5についての説明は[ここ](https://django-bootstrap5.readthedocs.io/en/latest/index.html)を参照してください。

> Bootstrapが提供するデフォルトの色やレイアウトなどを変更したい場合、CSSでBootstrapのCSSを上書きするなどして、独自の装飾やレイアウトなどを設定できます。

## django-bootstrap5のインストール

次の通りdjango-bootstrap5をインストールして、`requirements.txt`を更新します。

```bash
pip install django-bootstrap5
pip freeze > requirements.txt
```

プロジェクト設定ファイルの`INSTALL_APPS`に`django_bootstrap5`を追加します。

```python
# ./book_management/settings.py
  INSTALLED_APPS = [
      "django.contrib.admin",
      "django.contrib.auth",
      "django.contrib.contenttypes",
      "django.contrib.sessions",
      "django.contrib.messages",
      "django.contrib.staticfiles",
+     "django_bootstrap5",
      "debug_toolbar",
      "divisions.apps.DivisionsConfig",
      "books.apps.BooksConfig",
  ]
```

次の通り変更をリポジトリにコミットします。

```bash
git add ./book_management/settings.py
git add requirements.txt
git commit -m 'django-bootstrap5をインストール'
```

> a758707 (tag: 046-install-django-bootstrap5)

## Bootstrap用のベーステンプレートの作成

Bootstrap用のベーステンプレートを次の通り作成します。
なお、Bootstrap用のベーステンプレートは、django-bootstrap5が提供する`django_bootstrap5/bootstrap5.html`の`head`要素の`title`要素のコンテンツを空にしています。

```html
<!-- templates/bootstrap_base.html -->
{% extends 'django_bootstrap5/bootstrap5.html' %}

{% block bootstrap5_title %}
  {% block title %}{{ title }}{% endblock %}
{% endblock %}
```

Bootstrap用のベーステンプレートを実装したら、次の通り変更をリポジトリにコミットします。

```bash
git add ./templates/bootstrap_base.html
git commit -m 'Bootstrap用のベーステンプレートを実装'
```

> a47fa54 (tag: 047-implement-bootstrap-base-template)

なお、django-bootstrap5が提供する`django_bootstrap5/bootstrap5.html`の内容は次の通りです。

```html
<!-- django_bootstrap5/bootstrap5.html （一部整形） -->
<!DOCTYPE html>
{% load django_bootstrap5 %}
{% load i18n %}
{% get_current_language as LANGUAGE_CODE %}
<html lang="{{ LANGUAGE_CODE|default:'en_us' }}">
<head>
  <!-- Required meta tags -->
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
  <!-- Page title -->
  <title>{% block bootstrap5_title %}django-bootstrap5 template title{% endblock %}</title>
  <!-- Bootstrap CSS -->
  {% bootstrap_css %}
  <!-- Bootstrap JavaScript if it is in head -->
  {% if 'javascript_in_head'|bootstrap_setting %}
    {% bootstrap_javascript %}
  {% endif %}
  {% block bootstrap5_extra_head %}{% endblock %}
</head>
<body>
  <!-- 継承したレンプレートで、ページのヘッダを実装するブロック-->
  {% block bootstrap6_before_content %}{% endblock %}
  <!-- 継承したレンプレートで、ページのメインコンテンツを実装するブロック-->
  {% block bootstrap5_content %} CONTENT {% endblock %}
  <!-- 継承したレンプレートで、ページのフッターを実装するブロック-->
  {% block bootstrap5_after_content %}{% endblock %}
  <!-- Bootstrap JavaScript if it is in body -->
  {% if not 'javascript_in_head'|bootstrap_setting %}
    {% bootstrap_javascript %}
  {% endif %}
  {% block bootstrap5_extra_script %}{% endblock %}
</body>
</html>
```

`django_bootstrap5/bootstrap5.html`は、`bootstrap5_content`ブロックにページのコンテンツを、`bootstrap5_before_content`ブロックにページのヘッダー（HTMLの`head`要素ではありません）、`bootstrap5_after_content`にページのフッターを追加します。

また、Bootstrap5のCSSやJavaScriptは、このテンプレートが導入してくれます。

## 書籍ベースページテンプレートの実装

それぞれの書籍ページが継承する書籍ベースページテンプレートを次の通り実装します。

```html
<!-- ./books/templates/books/book_base_page.html -->
{% extends 'bootstrap_base.html' %}

{% block bootstrap5_before_content %}
  <div class="container-fluid">
    <h2>{{ title }}</h2>
  </div>
{% endblock bootstrap5_before_content %}
```

書籍ページベーステンプレートを実装したら、次の通り変更をリポジトリにコミットします。

```bash
git add ./books/templates/books/book_base_page.html
git commit -m '書籍ベースページテンプレートを実装'
```

> d5fe309 (tag: 048-implement-book-base-page-template)

## 書籍分類リンクテンプレートにBootstrapを適用

書籍分類リンクテンプレートを次で入れ替えます。

```html
<!-- ./books/templates/books/_classification_links.html -->
<div class="nav">
  {% for classification in classification_list %}
    <li class="nav-item">
      {% if classification.code != current_classification.code %}
        <a class="nav-link" href="{% url 'books:book-list' %}?classification_code={{ classification.code }}">
          {{ classification.name }}
        </a>
      {% else %}
        <a class="nav-link active" aria-current="page" href="#">{{ classification.name }}</a>
      {% endif %}
    </li>
  {% endfor %}
  <li class="nav-item">
    {% if current_classification %}
      <a class="nav-link" href="{% url 'books:book-list' %}">すべて</a>
    {% else %}
      <a class="nav-link active" aria-current="page" href="#">
        すべて
      </a>
    {% endif %}
  </li>
</div>
```

書籍分類リンクテンプレートを実装したら、次の通り変更をリポジトリにコミットします。

```bash
git add ./books/templates/books/_classification_links.html
git commit -m '書籍分類リンクテンプレートにBootstrapを適用'
```

> 1e0bcb7 (tag: 049-implement-classification-links-template)

## 書籍一覧テンプレートにBootstrapを適用

書籍一覧テンプレートを次で入れ替えます。

<!-- cspell: disable -->
```html
<!-- ./books/templates/books/book_list.html -->
{% extends 'books/book_base_page.html' %}

{% block bootstrap5_content %}
  <div class="container-fluid">
    {% include 'books/_classification_links.html' %}
    {% if book_list %}
      <table class="table table-striped table-hover table-sm ps-3">
        <thead class="table-dark">
        <tr>
          <th>タイトル</th>
          <th>分類</th>
          <th>分類詳細</th>
          <th>著者または訳者など</th>
          <th>出版社</th>
          <th>管理部署</th>
          <th>廃棄</th>
          <th/>
        </tr>
        </thead>
        <tbody>
        {% for book in book_list %}
          <tr>
            <td>{{ book.title }}</td>
            <td>{{ book.classification_detail.classification }}</td>
            <td>{{ book.classification_detail }}</td>
            <td>{% if book.authors %}{{ book.authors | truncatechars:40 }}{% else %}ー{% endif %}</td>
            <td>{{ book.publisher | default_if_none:"ー" }}</td>
            <td>{{ book.division }}</td>
            <td>{{ book.disposed | yesno:"済,ー" }}</td>
            <td>
              <a href="{% url 'books:book-detail' pk=book.id %}">詳細</a>
              <a href="{% url 'books:book-update' pk=book.id %}">更新</a>
              <a href="{% url 'books:book-delete' pk=book.id %}">削除</a>
            </td>
          </tr>
        {% endfor %}
        </tbody>
      </table>
    {% else %}
      <div class="ms-3">書籍が存在しません。</div>
    {% endif %}
    <div>
      <a class="ms-3" href="{% url 'books:book-create' %}">書籍登録</a>
    </div>
  </div>
{% endblock bootstrap5_content %}
```
<!-- cspell: enable -->

ブラウザで書籍一覧ページにアクセスして、Bootstrapで装飾及びレイアウトした結果を確認した後、次の通り変更をリポジトリにコミットします。

```bash
git add ./books/templates/books/book_list.html
git commit -m '書籍一覧テンプレートにBootstrapを適用'
```

> f1df953 (tag: 050-apply-bootstrap-to-book-list-template)

## 書籍詳細表示テンプレートにBootstrapを的章

書籍書斎表示テンプレートにBootstrapを次の通り適用します。

```html
<!-- ./books/templates/books/_book_detail.html -->
<table class="table table-striped table-hover table-sm ps-3">
  <tr>
    <th class="table-dark text-end pe-3 pe-3">タイトル</th>
    <td class="ps-3">{{ book.title }}</td>
  </tr>
  <tr>
    <th class="table-dark text-end pe-3">分類</th>
    <td class="ps-3">{{ book.classification_detail.classification }}</td>
  </tr>
  <tr>
    <th class="table-dark text-end pe-3">分類詳細</th>
    <td class="ps-3">{{ book.classification_detail }}</td>
  </tr>
  <tr>
    <th class="table-dark text-end pe-3">著者または訳者</th>
    <td class="ps-3">{% if book.authors %}{{ book.authors }}{% else %}ー{% endif %}
  </tr>
  <tr>
    <th class="table-dark text-end pe-3">ISBN</th>
    <td class="ps-3">{{ book.isbn | default_if_none:"ー" }}</td>
  </tr>
  <tr>
    <th class="table-dark text-end pe-3 text-end pe-3">出版社</th>
    <td class="ps-3">{{ book.publisher | default_if_none:"ー" }}</td>
  </tr>
  <tr>
    <th class="table-dark text-end pe-3">発行日</th>
    <td class="ps-3">{{ book.published_at | default_if_none:"ー" }}</td>
  </tr>
  <tr>
    <th class="table-dark text-end pe-3">管理部署</th>
    <td class="ps-3">{{ book.division }}</td>
  </tr>
  <tr>
    <th class="table-dark text-end pe-3">廃棄</th>
    <td class="ps-3">{{ book.disposed | yesno:"済,ー" }}</td>
  </tr>
  {% if book.disposed %}
    <tr>
      <th class="table-dark text-end pe-3">廃棄日</th>
      <td class="ps-3">{{ book.disposed_at }}</td>
    </tr>
  {% endif %}
  <tr>
    <th class="table-dark text-end pe-3">登録日時</th>
    <td class="ps-3">{{ book.created_at }}</td>
  </tr>
  <tr>
    <th class="table-dark text-end pe-3">更新日時</th>
    <td class="ps-3">{{ book.updated_at }}</td>
  </tr>
</table>
```

書籍詳細表示テンプレートにBootstrapを適用したら、次の通り変更をリポジトリにコミットします。

```bash
git add ./books/templates/books/_book_detail.html
git commit -m 'books/_book_detail.htmlにBootstrapを適用'
```

> afea70c (tag: 051-apply-bootstrap-to-_book_detail.html)

## 書籍詳細テンプレートにBootstrapを適用

書籍詳細テンプレートにBootstrapを次の通り適用します。

```html
<!-- ./books/templates/books/book_detail.html -->
{% extends 'books/book_base_page.html' %}

{% block bootstrap5_content %}
  <div class="container-fluid">
    {% include "books/_book_detail.html" %}
    <div>
      <a href="{% url 'books:book-list' %}">書籍一覧</a>
      <a href="{% url 'books:book-create' %}">書籍登録</a>
      <a href="{% url 'books:book-update' pk=book.id %}">書籍更新</a>
      <a href="{% url 'books:book-delete' pk=book.id %}">書籍削除</a>
    </div>
  </div>
{% endblock bootstrap5_content %}
```

ブラウザで書籍詳細ページにアクセスして、Bootstrapで装飾及びレイアウトした結果を確認した後、次の通り変更をリポジトリにコミットします。

```bash
git add ./books/templates/books/book_detail.html
git commit -m '書籍詳細テンプレートにBootstrapを適用'
```

> 0e39c1e (tag: 052-apply-bootstrap-to-book-detail-template)

## 書籍フォームテンプレートにBootstrapを適用

書籍フォームテンプレートに適用するスタイルを指定したファイルを次の通り作成します。

> このファイルはCSSファイルとして妥当ですが、テンプレート内にレンダリングされるため、一般的なCSSファイルの使用方法と異なります。

```css
/* ./books/templates/books/_button_width.css */
.button-width {
  width: 10rem;
}
```

書籍フォームテンプレートにBootstrapを次の通り適用します。

```html
<!-- ./books/templates/books/book_form.html -->
{% extends 'books/book_base_page.html' %}
{% load django_bootstrap5 %}

{% block bootstrap5_extra_head %}
  <style>
    {% include 'books/_button_width.css' %}
  </style>
{% endblock %}

{% block bootstrap5_content %}
  <div class="container-fluid pb-3">
    <form method="post">
      {% csrf_token %}
      <div class="row">
        <div class="col">
          {% bootstrap_field form.title size='sm' %}
        </div>
      </div>
      <div class="row">
        <div class="col">
          {% bootstrap_field form.classification size='sm' %}
        </div>
        <div class="col">
          {% bootstrap_field form.classification_detail size='sm' %}
        </div>
      </div>
      <div class="col">
        {% bootstrap_field form.authors size='sm' %}
      </div>
      <div class="row">
        <div class="col">
          {% bootstrap_field form.publisher size='sm' %}
        </div>
        <div class="col">
          {% bootstrap_field form.published_at size='sm' %}
        </div>
        <div class="col">
          {% bootstrap_field form.isbn size='sm' %}
        </div>
      </div>
      <div class="row">
        <div class="col-6">
          {% bootstrap_field form.division size='sm' %}
        </div>
        <div class="col-2" style="padding-top:2.2rem">
          {% bootstrap_field form.disposed size='sm' %}
        </div>
        <div class="col-4">
          {% bootstrap_field form.disposed_at size='sm' %}
        </div>
      </div>
      <div class="row">
        <div class="col">
          <span>
            <a href="{% url 'books:book-list' %}">書籍一覧</a>
            {% if action == "更新" %}
              <a href="{% url 'books:book-detail' form.instance.id %}">書籍詳細</a>
              <a href="{% url 'books:book-delete' form.instance.id %}">書籍削除</a>
            {% endif %}
          </span>
          <span class="float-end">
            {% bootstrap_button action button_type='submit' button_class='btn-primary' extra_classes='button-width' size='sm' %}
          </span>
        </div>
      </div>
    </form>
  </div>
  <script>
    //
    // 書籍分類詳細の選択肢を選択された書籍分類でフィルタする処理をJavaScriptで実装
    //
    // すべての書籍分類詳細のJSON表現
    // safeテンプレートフィルタで、DjangoがJSON表現をレンダリングするときにXSS攻撃などを防ぐための無害化処理（サニタイズ）を無効化
    const classificationDetails = {{ classification_details | safe }};

    // 書籍分類詳細の選択肢を設定する関数
    const setClassificationDetailOptions = () => {
      // 書籍分類フォームフィールド（select要素）で選択された書籍分類の書籍分類コードを取得
      // getElementById関数は、引数で指定されたid属性を持つHTML要素を検索
      // 次のコードは、取得した書籍分類フォームフィールドで選択された書籍分類の書籍分類コードを取得
      // <field>.auto_idは、Djangoがフォームフィールドに設定するid属性の値
      const code = document.getElementById("{{ form.classification.auto_id }}").value;
      // すべての書籍分類詳細から上記で取得した書籍分類コードを持つ書籍分類詳細を抽出
      const filteredDetails = classificationDetails.filter(cd => cd.classification === code);
      // 書籍分類詳細フォームフィールドを取得
      const select = document.getElementById("{{ form.classification_detail.auto_id }}");
      // 書籍分類詳細フォームフィールドの子要素をすべて削除
      while (select.firstChild) {
        select.removeChild(select.lastChild);
      }
      // 書籍分類詳細フォームフィールド（select要素）に選択肢をoption要素として追加
      filteredDetails.forEach(detail => {
        let option = document.createElement("option");
        option.value = detail.code;
        option.textContent = detail.name;
        select.appendChild(option);
      });
    }

    // ページがロードされた後に実行される関数を登録
    window.onload = () => {
      setClassificationDetailOptions();
      // 書籍分類詳細フォームフィールドを取得
      const select = document.getElementById("{{ form.classification_detail.auto_id }}");
      // 書籍更新ページの場合は、setClassificationDetailOption関数によって、書籍の書籍分類詳細がクリアされるため
      // 書籍分類詳細を再設定
      if ("{{ action }}" === "更新") {
        select.value = "{{ book.classification_detail.code }}";
      }
    }

    // 書籍分類フォームフィールドの選択が変更された後に実行する関数を登録
    document.getElementById("{{ form.classification.auto_id }}")
        .addEventListener("change", (ev) => {
          setClassificationDetailOptions();
        });
  </script>
{% endblock bootstrap5_content %}
```

ブラウザで書籍登録及び更新ページにアクセスして、Bootstrapで装飾及びレイアウトした結果を確認した後、次の通り変更をリポジトリにコミットします。

```bash
git add ./books/
git commit -m '書籍フォームテンプレートにBootstrapを適用'
```

> bd21c4f (tag: 053-apply-bootstrap-to-book-form-template)

## 書籍削除テンプレートにBootstrapを適用

書籍削除テンプレートにBootstrapを次の通り適用します。

```html
<!-- ./books/templates/books/book_confirm_delete.html -->
{% extends 'books/book_base_page.html' %}
{% load django_bootstrap5 %}

{% block bootstrap5_extra_head %}
  <style>
    {% include 'books/_button_width.css' %}
  </style>
{% endblock %}

{% block bootstrap5_content %}
  <div class="container-fluid">
    <div>この書籍を削除しますか?</div>
    <form method="post">
      {% csrf_token %}
      {% include "books/_book_detail.html" %}
      <div class="col">
        <span>
          <a href="{% url 'books:book-list' %}">書籍一覧</a>
          <a href="{% url 'books:book-detail' pk=book.id %}">書籍詳細</a>
          <a href="{% url 'books:book-update' pk=book.id %}">書籍更新</a>
        </span>
        <span class="float-end">
          {% bootstrap_button '削除' button_type='submit' button_class='btn-danger' extra_classes='button-width' size='sm' %}
        </span>
      </div>
    </form>
  </div>
{% endblock bootstrap5_content %}
```

ブラウザで書籍削除ページにアクセスして、Bootstrapで装飾及びレイアウトした結果を確認した後、次の通り変更をリポジトリにコミットします。

```bash
git add ./books/templates/books/book_confirm_delete.html
git commit -m '書籍削除テンプレートにBootstrapを適用'
```

> ab55f06 (tag: 054-apply-bootstrap-to-book-delete-template)

## まとめ

本層では、書籍ページを`Bootstrap`を使用して装飾及びレイアウトしました。

次の章では、ユーザー認証機能を実装して、書籍登録、更新及び削除ページには、認証済みユーザーしかアクセスできないように制限します。
