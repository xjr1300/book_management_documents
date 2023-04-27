# 書籍アプリページの実装

- [書籍アプリページの実装](#書籍アプリページの実装)
  - [ミックスインの整理](#ミックスインの整理)
  - [書籍分類及び書籍分類詳細ページの実装](#書籍分類及び書籍分類詳細ページの実装)
    - [書籍分類ページの実装](#書籍分類ページの実装)
    - [書籍分類詳細ページの実装](#書籍分類詳細ページの実装)
  - [書籍分類詳細一覧ページで書籍分類詳細を書籍分類でフィルタ表示](#書籍分類詳細一覧ページで書籍分類詳細を書籍分類でフィルタ表示)
  - [書籍フィクスチャーの作成と書籍の登録](#書籍フィクスチャーの作成と書籍の登録)
  - [書籍ページの実装](#書籍ページの実装)
    - [書籍一覧ページの実装](#書籍一覧ページの実装)
    - [書籍詳細ページの実装](#書籍詳細ページの実装)
    - [書籍フォームの実装](#書籍フォームの実装)
    - [書籍登録ページの実装](#書籍登録ページの実装)
    - [書籍更新ページの実装](#書籍更新ページの実装)
    - [書籍削除ページの実装](#書籍削除ページの実装)
  - [トランザクション](#トランザクション)
  - [まとめ](#まとめ)

## ミックスインの整理

部署アプリのビューを実装するときに定義した`PageTitleMixin`及び`FormActionMixin`は、部署アプリのビューで再利用するため、`core`モジュールに移動します。

> 部署の`DivisionSingleObjectMixin`や`DivisionFormFieldMixin`も書籍分類モデルと同様な実装になりますが、部署と書籍分類が概念が異なるため、再利用することは`やり過ぎたDRY`になるアンチパターンです。
> `PageTitleMixin`はHTMLの`head`要素の`title`要素のコンテンツを設定する、`FormActionMixin`は操作名を提供するミックスインで、ビュー共通の振る舞いを提供するため再利用します。

`./core/mixins.py`ファイルを作成して、`PageTitleMixin`と`FormActionMixin`をそのファイルに移動します。

```python
# ./core/mixins.py
from typing import Any, Dict

from django.views import generic


class PageTitleMixin(generic.base.ContextMixin):
    """ページタイトルミックスイン"""

    title = None

    def get_context_data(self, **kwargs: Any) -> Dict[str, Any]:
        ctx = super().get_context_data(**kwargs)
        ctx["title"] = self.title
        return ctx


class FormActionMixin(generic.base.ContextMixin):
    """フォームアクションミックスイン"""

    action = None

    def get_context_data(self, **kwargs: Any) -> Dict[str, Any]:
        ctx = super().get_context_data(**kwargs)
        ctx["action"] = self.action
        return ctx
```

その後、`./divisions/views.py`を次の通り変更します。

```python
# ./divisions/views.py
- from typing import Any, Dict, Type
+ from typing import Type

  from django import forms
  from django.urls import reverse, reverse_lazy
  from django.views import generic

+ from core.mixins import PageTitleMixin, FormActionMixin
+
  from .models import Division

  (...略...)

- class PageTitleMixin(generic.base.ContextMixin):
-     """ページタイトルミックスイン"""
-
-     title = None
-
-     def get_context_data(self, **kwargs: Any) -> Dict[str, Any]:
-         ctx = super().get_context_data(**kwargs)
-         ctx["title"] = se
-
-
- class FormActionMixin(generic.base.ContextMixin):
-     """フォームアクションミックスイン"""
-
-     action = None
-
-     def get_context_data(self, **kwargs: Any) -> Dict[str, Any]:
-         ctx = super().get_context_data(**kwargs)
-         ctx["action"] = self.action
-         return ctx
```

部署アプリのすべてのビューが正常に動作することを確認したら、変更をリポジトリにコミットします。

```bash
git add ./divisions/
git add ./core/
git commit -m 'ミックスインを整理'
```

> commit 52e2d06319416182136fd3af04353b0cee750f55

## 書籍分類及び書籍分類詳細ページの実装

書籍分類ビューを書籍アプリのビューを部署と同様なビューとURLパターンで実装します。
本チュートリアルで説明しませんので、実装に挑戦してください。

### 書籍分類ページの実装

書籍分類ページへのリクエストURLを次の通りとします。

- 書籍分類一覧ページ: `/books/classifications/`
- 書籍分類詳細ページ: `/books/classifications/<code>`
- 書籍分類登録ページ: `/books/classifications/create/`
- 書籍分類更新ページ: `/books/classifications/update/<code>`
- 書籍分類削除ページ: `/books/classifications/delete/<code>`

なお、`./books/urls.py`に次を追加することを忘れないでください。

```python
# ./books/urls.py

app_name = "books"
```

> commit f411718d467d626b5b8c8998668c08a80777a9c8

### 書籍分類詳細ページの実装

書籍分類詳細ページへのリクエストURLを次の通りとします。

- 書籍分類詳細一覧ページ: `/books/classification-details/`
- 書籍分類詳細詳細ページ: `/books/classification-details/<code>`
- 書籍分類詳細登録ページ: `/books/classification-details/create/`
- 書籍分類詳細更新ページ: `/books/classification-details/update/<code>`
- 書籍分類詳細削除ページ: `/books/classification-details/delete/<code>`

書籍分類詳細リストビュー（`ClassificationDetailListView`）は次の通り実装してください。

```python
class ClassificationDetailListView(
    ClassificationDetailViewMixin, PageTitleMixin, generic.ListView
):
    """書籍分類詳細一覧クラスビュー"""

    title = "書籍分類詳細一覧"
    context_object_name = "classification_detail_list"
    template_name = "books/classification_detail_list.html"
```

`context_object_name`で書籍分類詳細リストが取得する書籍分類モデルインスタンスのQuerySetのコンテキスト名を指定しています。
`context_object_name`を指定しない場合、そのコンテキスト名はモデル名を単純に小文字化した`classificationdetail_list`になります。

`template_name`で書籍分類詳細リストページのテンプレートを指定しています。
`template_name`を指定しない場合、`books/classificationdetail_list.html`になります。

`ClassificationDetailSingleObjectMixin`は次の通り実装してください。

```python
class ClassificationDetailSingleObjectMixin:
    """書籍分類詳細ビューシングルオブジェクトミックスイン"""

    pk_url_kwarg = "code"
    context_object_name = "classification_detail"
```

`context_object_name`で特定の書籍分類詳細を表示するビュー（[SingleObjectMixin](https://docs.djangoproject.com/en/4.2/ref/class-based-views/mixins-single-object/#single-object-mixins)）が取得する書籍分類詳細モデルインスタンスのコンテキスト名を指定しています。
`context_object_name`を指定しない場合、そのコンテキスト名は`classificationdetail`になります。

`ClassificationDetailFormFieldMixin`を`ClassificationDetailFormMixin`に名前を変更して、次の通り実装してください。

```python
class ClassificationDetailFormMixin(generic.edit.ModelFormMixin):
    """書籍分類詳細フォームフィールドミックスイン"""

    fields = ("code", "classification", "name")
    template_name = "books/classification_detail_form.html"

    def get_form(self, form_class: Optional[forms.Form] = None) -> forms.Form:
        form = super().get_form(form_class)
        form.fields["classification"].empty_label = None
        return form
```

クラス名の変更は、フィールドのリストだけでなくフォームを設定するミックスインに変更したことが理由です。

`template_name`の指定は書籍分類一覧ページと同様です。

上記の通り`get_form`をオーバーライドしない場合、書籍分類ドロップダウンに書籍分類を選択していないことを示す`---------`を選択できます。
書籍分類詳細モデルは、書籍分類を必ず入力する必要があるため、`---------`を選択候補から除くために`get_form`をオーバーライドしています。

`ClassificationDetailDeleteView`は次の通り実装してください。

```python
class ClassificationDetailDeleteView(
    ClassificationDetailViewMixin,
    ClassificationDetailSingleObjectMixin,
    PageTitleMixin,
    generic.DeleteView,
):
    """書籍分類詳細削除クラスビュー"""

    title = "書籍分類詳細削除"
    template_name = "books/classification_detail_confirm_delete.html"
    success_url = reverse_lazy("classification-detail-list")
```

> commit 6a8f65f9e84be41ce7645d4f6bd4421042fc2263

書籍分類詳細ページを実装したら変更をリポジトリにコミットします。

```bash
git add --all
git commit -m '書籍分類詳細ページを実装
```

> commit 6a8f65f9e84be41ce7645d4f6bd4421042fc2263

## 書籍分類詳細一覧ページで書籍分類詳細を書籍分類でフィルタ表示

書籍分類詳細一覧ページで、管理サイトと同様に、書籍分類詳細の一覧を書籍分類でフィルタ表示できるようにします。

次のようなリクエストURLを受信したら、書籍分類コードGETパラメーター（`category_code`がパラメーター名で、`code`が書籍分類コード）が一致する書籍分類詳細のみを一覧で表示します。

```url
http://localhost:8000/books/classification_details/?category_code=<code>
```

書籍分類コードGETパラメーターが指定されていないとき、または書籍分類コードがどの書籍分類にも一致しない場合は、すべての書籍分類を表示します。

`./books/views.py`のインポート文を次の通り変更します。

```python
# ./books/views.py
- from typing import Type, Optional
+ from typing import Any, Dict, Optional, Type

  from django import forms
+ from django.db.models import QuerySet
  from django.urls import reverse, reverse_lazy
  from django.views import generic
+ from django.http import HttpRequest

  from core.mixins import FormActionMixin, PageTitleMixin
```

その後、次の通り`get_classification_from_param`関数を追加して、書籍分類詳細一覧リストビューの実装を置き換えます。

```python
# ./books/views.py
def get_classification_from_param(request: HttpRequest) -> Optional[Classification]:
    """書籍分類コードGETパラメーターで指定された書籍分類コードから書籍分類モデルインスタンスを取得して返却する。

    Args:
        request: HTTPリクエスト

    Returns
        書籍分類モデルインスタンス。書籍分類コードGETパラメーターが指定されていない場合、またそのパラメータで指定された
        書籍分類コードと一致する書籍分類が存在しない場合はNone。
    """
    # 書籍分類コードGETパラメーターの値を取得
    code = request.GET.get("classification_code", None)
    # 書籍分類コードGETパラメータが指定されていない場合はNoneを返却
    if not code:
        return None

    try:
        # 書籍分類コードから書籍分類モデルインスタンスを取得
        return Classification.objects.get(code=code)
    except Classification.DoesNotExist:
        # 書籍分類コードから書籍分類モデルインスタンスを取得できない場合はNoneを返却
        return None


class ClassificationDetailListView(
    ClassificationDetailViewMixin, PageTitleMixin, generic.ListView
):
    """書籍分類詳細一覧クラスビュー"""

    title = "書籍分類詳細一覧"
    context_object_name = "classification_detail_list"
    template_name = "books/classification_detail_list.html"
    classification: Optional[Classification] = None

    def get_queryset(self) -> QuerySet[ClassificationDetail]:
        """書籍分類詳細一覧ページで表示する書籍分類詳細QuerySetを返却する。"""
        self.classification = get_classification_from_param(self.request)
        if not self.classification:
            return ClassificationDetail.objects.all()
        else:
            return ClassificationDetail.objects.filter(
                classification=self.classification
            )

    def get_context_data(self, **kwargs: Any) -> Dict[str, Any]:
        """コンテキストを取得して、そのコンテキストにすべての書籍分類モデルインスタンスと現在表示している書籍分類を登録する。"""
        ctx = super().get_context_data(**kwargs)
        ctx["classification_list"] = Classification.objects.all()
        ctx["current_classification"] = self.classification
        return ctx
````

書籍分類詳細一覧ビューを実装したら、次のURLなどで書籍分類詳細が正しくフィルタされるか確認してください。

- 総記: <http://localhost:8000/books/classification-details/?category_code=000>
- 哲学: <http://localhost:8000/books/classification-details/?category_code=100>

書籍分類詳細一覧ページで書籍分類で書籍分類詳細をフィルタできるように`./books/templates/books/classification_detail_list.html`テンプレートを次で置き換えます。

```html
{% extends 'base.html' %}

{% block inner_body %}
  <h1>書籍分類詳細一覧</h1>
  <div>
    {% for classification in classification_list %}
      {% if classification.code != current_classification.code %}
        <a href="{% url 'books:classification-detail-list' %}?classification_code={{ classification.code }}">
          {{ classification.name }}
        </a>&thinsp;
      {% else %}
        {{ classification.name }}&thinsp;
      {% endif %}
    {% endfor %}
    {% if current_classification %}
      <a href="{% url 'books:classification-detail-list' %}">すべて</a>
    {% else %}
      すべて
    {% endif %}
  </div>
  {% if classification_detail_list %}
    <ul>
      {% for classification_detail in classification_detail_list %}
        <li>
          {{ classification_detail.code }}: {{ classification_detail.classification }} > {{ classification_detail.name }}
          <a href="{% url 'books:classification-detail-detail' classification_detail.code %}">詳細</a>
          <a href="{% url 'books:classification-detail-update' classification_detail.code %}">更新</a>
          <a href="{% url 'books:classification-detail-delete' classification_detail.code %}">削除</a>
        </li>
      {% endfor %}
    </ul>
  {% else %}
    <p>書籍分類詳細が存在しません。</p>
  {% endif %}
  <a href="{% url 'books:classification-detail-create' %}">書籍分類詳細登録</a>
{% endblock inner_body %}
```

書籍分類詳細一覧ページに書籍分類一覧を書籍分類でフィルタするリンクが表示されます。
書籍分類フィルタリンクをクリックして、書籍分類詳細がその書籍分類でフィルタされることを確認できたら、次の通り変更をコミットします。

```bash
git add ./books/
git commit -m '書籍分類詳細一覧ページで書籍分類詳細を書籍分類でフィルタできるように実装'
```

> commit 57587bb53b37a6adb74465feacc1444ec40d4fc9

## 書籍フィクスチャーの作成と書籍の登録

書籍ページの実装で使用するために、`./books/fixtures/books.yaml`に書籍フィクスチャーを記録します。

<!-- cspell: disable -->
```yaml
# ./books/fixtures/books.yaml
- model: books.book
  pk: 01GYV46C5KXWDRKMB1WR3TW6RK
  fields:
    title: Fluent Python
    classification_detail: "000"
    authors: |-
      （著）Luciano Ramalho
      （監修）豊沢　聡、桑井　博之
      （訳）梶原　玲子
    isbn: ISBN978-4-87311-817-8
    publisher: オライリー・ジャパン
    published_at: 2017-10-11
    division: "58"
    disposed: false
    disposed_at: null
    created_at: 2023-04-25 00:00:00.000000+00:00
    updated_at: 2023-04-25 00:00:00.000000+00:00
- model: books.book
  pk: 01GYV585X4JNDCSVA3BY93KN54
  fields:
    title: プログラミングRust
    classification_detail: "000"
    authors: |-
      （著）Jim Blandy、Jason Orendorff
      （訳）中田　秀基
    isbn: ISBN978-4-87311-855-0
    publisher: オライリー・ジャパン
    published_at: 2018-08-08
    division: "58"
    disposed: false
    disposed_at: null
    created_at: 2023-04-25 00:00:00.000000+00:00
    updated_at: 2023-04-25 00:00:00.000000+00:00
```
<!-- cspell: enable -->

書籍フィクスチャーを次の通りデータベースに書籍を登録します。

```bash
python manage.py loaddata --format=yaml books/fixtures/books.yaml
```

書籍フィクスチャーで書籍をデータベースに登録できたら、変更をリポジトリにコミットします。

```bash
git add ./books/fixtures/books.yaml
git commit -m '書籍フィクスチャーを作成
```

> commit 5f63ac11d59d8db377bf78c2d03ddeae98cb9fab

## 書籍ページの実装

### 書籍一覧ページの実装

`./books/views.py`に次の通り書籍一覧リストビューを実装します。

```python
class BookViewMixin:
    """書籍ビューミックスイン"""

    model = Book


class BookListView(BookViewMixin, PageTitleMixin, generic.ListView):
    """書籍一覧クラスビュー"""

    title = "書籍一覧"
    classification: Optional[Classification] = None

    def get_queryset(self) -> QuerySet[Book]:
        """書籍一覧ページで表示する書籍QuerySetを返却する。"""
        self.classification = get_classification_from_param(self.request)
        if not self.classification:
            return Book.objects.all()
        else:
            return Book.objects.filter(
                classification_detail__classification=self.classification
            )

    def get_context_data(self, **kwargs: Any) -> Dict[str, Any]:
        """コンテキストを取得して、そのコンテキストにすべての書籍分類モデルインスタンスを登録する。"""
        ctx = super().get_context_data(**kwargs)
        ctx["classification_list"] = Classification.objects.all()
        ctx["current_classification"] = self.classification
        return ctx
```

書籍一覧テンプレートを`./books/templates/books/book_list.html`に次の通り実装します。
なお、書籍の一覧はHTMLの`table`要素で出力します。

```html
<!-- ./books/templates/books/book_list.html -->
{% extends 'base.html' %}

{% block inner_body %}
  <h1>書籍一覧</h1>
  <div>
    {% for classification in classification_list %}
      {% if classification.code != current_classification.code %}
        <a href="{% url 'books:book-list' %}?classification_code={{ classification.code }}">
          {{ classification.name }}
        </a>&thinsp;
      {% else %}
        {{ classification.name }}&thinsp;
      {% endif %}
    {% endfor %}
    {% if current_classification %}
      <a href="{% url 'books:book-list' %}">すべて</a>
    {% else %}
      すべて
    {% endif %}
  </div>
  {% if book_list %}
    <table>
      <thead>
      <tr>
        <th>タイトル</th>
        <th>分類</th>
        <th>分類詳細</th>
        <th>著者または訳者など</th>
        <th>出版社</th>
        <th>管理部署</th>
        <th>廃棄</th>
        <th />
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
          <td />
        </tr>
      {% endfor %}
      </tbody>
    </table>
  {% else %}
    <p>書籍が存在しません。</p>
  {% endif %}
{% endblock inner_body %}
```

テンプレートの`truncatechars`は、表示する文字列を指定された文字数まで出力する`テンプレートフィルタ`です。

また、`yesno`は、左を`True`と評価された場合は右の文字列をカンマ（`,`）で区切ったときの最初の文字列、`False`と評価された場合は2番目の文字列を出力するテンプレートフィルタです。
なお、オプションで`None`と評価された場合は3番目の文字列を出力します。

> Djangoにビルトインされているテンプレートフィルタについては、[ここ](https://docs.djangoproject.com/en/4.2/ref/templates/builtins/#built-in-filter-reference)を参照してください。

書籍一覧リストビューをディスパッチするために、`./books/urls.py`に次を追加します。

```python
# ./books/urls.py
    # 書籍一覧ページ (ex: /books/)
    path("", views.BookListView.as_view(), name="book-list"),
```

開発サーバーを起動後、ブラウザで`http://localhost:8000/books/`にアクセスして、書籍一覧ページが正常に表示されたら変更をリポジトリにコミットします。

```bash
git add ./books/
git commit -m '書籍一覧ページを実装'
```

> commit cd09805fa6c6810291eb7252e60a9ee1d5972dcb

### 書籍詳細ページの実装

書籍詳細ビューを次の通り実装します。

```python
# ./books/views.py
class BookDetailView(
    BookViewMixin,
    PageTitleMixin,
    generic.DetailView,
):
    """書籍詳細クラスビュー"""

    title = "書籍詳細"
```

書籍詳細テンプレートにおいて、書籍の詳細を`table`要素で表示ます。
この書籍の詳細を表示する`table`要素は、後で実装する書籍削除テンプレートにも表示したいため、`./books/templates/books/_book_detail.html`テンプレートに実装してインクルード(`include`)します。

```html
<!-- ./books/templates/books/_book_detail.html -->
<table>
  <tr>
    <th>タイトル</th>
    <td>{{ book.title }}</td>
  </tr>
  <tr>
    <th>分類</th>
    <td>{{ book.classification_detail.classification }}</td>
  </tr>
  <tr>
    <th>分類詳細</th>
    <td>{{ book.classification_detail }}</td>
  </tr>
  <tr>
    <th>著者または訳者</th>
    <td>{% if book.authors %}{{ book.authors }}{% else %}ー{% endif %}
  </tr>
  <tr>
    <th>ISBN</th>
    <td>{{ book.isbn | default_if_none:"ー" }}</td>
  </tr>
  <tr>
    <th>出版社</th>
    <td>{{ book.publisher | default_if_none:"ー" }}</td>
  </tr>
  <tr>
    <th>発行日</th>
    <td>{{ book.published_at | default_if_none:"ー" }}</td>
  </tr>
  <tr>
    <th>管理部署</th>
    <td>{{ book.division }}</td>
  </tr>
  <tr>
    <th>廃棄</th>
    <td>{{ book.disposed | yesno:"済,ー" }}</td>
  </tr>
  {% if book.disposed %}
    <tr>
      <th>廃棄日</th>
      <td>{{ book.disposed_at }}</td>
    </tr>
  {% endif %}
  <tr>
    <th>登録日時</th>
    <td>{{ book.created_at }}</td>
  </tr>
  <tr>
    <th>更新日時</th>
    <td>{{ book.updated_at }}</td>
  </tr>
</table>
```

書籍詳細テンプレートを次の通り実装します。

```html
<!-- ./books/templates/books/book_detail.html -->
{% extends 'base.html' %}

{% block inner_body %}
  <h1>書籍詳細</h1>
  {% include "books/_book_detail.html" %}
  <div>
    <a href="{% url 'books:book-list' %}">書籍一覧</a>
  </div>
{% endblock inner_body %}
```

書籍詳細ビューをディスパッチするために、`./books/urls.py`に次を追加します。

```python
    # 書籍詳細ページ (ex: /books/<id>/)
    path("<str:pk>/", views.BookDetailView.as_view(), name="book-detail"),
```

また、書籍一覧テンプレートに書籍詳細ページへのリンクを追加します。

```html
<!-- ./books/templates/books/book_list.html -->
        {% for book in book_list %}
          <tr>
            <td>{{ book.title }}</td>
            <td>{{ book.classification_detail.classification }}</td>
            <td>{{ book.classification_detail }}</td>
            <td>{% if book.authors %}{{ book.authors | truncatechars:40 }}{% else %}ー{% endif %}</td>
            <td>{{ book.publisher | default_if_none:"ー" }}</td>
            <td>{{ book.division }}</td>
            <td>{{ book.disposed | yesno:"済,ー" }}</td>
-           <td />
+           <td>
+             <a href="{% url 'books:book-detail' pk=book.id %}">詳細</a>
+           </td>
          </tr>
        {% endfor %}
```

開発サーバーを起動後、ブラウザで`http://localhost:8000/books/`にアクセスして、書籍詳細ページへのリンクをクリックして、書籍詳細ページが正常に表示されたら変更をリポジトリにコミットします。

```bash
git add ./books/
git commit -m '書籍詳細ページを実装'
```

> commit 97dde10a459266f89f61080ffe1d072a18875f7b

### 書籍フォームの実装

`./books/form.py`ファイルを作成して、次の通り書籍フォームを実装します。
書籍フォームは、書籍モデルのモデルフィールドに対応しない、書籍分類フォームフィールドを追加します。
書籍分類フォームフィールドは、書籍フォームの書籍分類詳細フォームフィールドの選択肢を、書籍分類フォームフィールドに入力された書籍分類でフィルタするために使用します。

```python
# ./books/forms.py
from django import forms

from .models import Book, Classification, ClassificationDetail


class BookForm(forms.ModelForm):
    """書籍フォーム"""

    classification = forms.ModelChoiceField(
        label="書籍分類", queryset=Classification.objects.all(), empty_label=None
    )

    class Meta:
        model = Book
        fields = (
            "title",
            "classification",
            "classification_detail",
            "authors",
            "isbn",
            "publisher",
            "published_at",
            "division",
            "disposed",
            "disposed_at",
        )

    def __init__(self, *args, **kwargs) -> None:
        """イニシャライザ"""
        super().__init__(*args, **kwargs)
        self.fields["classification_detail"].empty_label = None
        self.fields["division"].empty_label = None

```

書籍フォームテンプレートを次の通り実装します。
コメントで記述した説明を確認してください。

```html
<!-- ./books/templates/books/book_form.html -->
{% extends 'base.html' %}

{% block inner_body %}
  <h1>書籍{{ action }}</h1>
  <form method="post">
    {% csrf_token %} {{ form.as_p }}
    <button type="submit">{{ action }}</button>
  </form>
  <div>
    <a href="{% url 'books:book-list' %}">書籍一覧</a>
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
    }

    // 書籍分類フォームフィールドの選択が変更された後に実行する関数を登録
    document.getElementById("{{ form.classification.auto_id }}")
        .addEventListener("change", (ev) => {
          setClassificationDetailOptions();
        });
  </script>
{% endblock inner_body %}
```

部署詳細ページのレンダリング結果は、ChromeのDevToolなどで確認してください。

### 書籍登録ページの実装

書籍登録ビューを次の通り実装します。

```python
# ./books/views.py
class BookCreateView(
    BookViewMixin,
    PageTitleMixin,
    FormActionMixin,
    generic.CreateView,
):
    """書籍登録ビュー"""

    form_class = BookForm
    title = "書籍登録"
    action = "登録"

    def get_context_data(self, **kwargs: Any) -> Dict[str, Any]:
        ctx = super().get_context_data(**kwargs)
        ctx["classification_details"] = list(
            ClassificationDetail.objects.all().values("code", "classification", "name")
        )
        return ctx

    def get_success_url(self) -> str:
        return reverse("book-detail", kwargs={"pk": self.object.id})
```

書籍一覧ページに書籍登録ページへのリンクを設置するために、書籍一覧テンプレートを次の通り変更します。

```html
<!-- ./books/templates/books/book_list.html -->
    {% else %}
      <p>書籍が存在しません。</p>
    {% endif %}
+   <div>
+     <a href="{% url 'books:book-create' %}">書籍登録</a>
+   </div>
  {% endblock inner_body %}
```

書籍詳細ページに書籍登録ページへのリンクを設置するために、書籍一覧テンプレートを次の通り変更します。

```html
<!-- ./books/templates/books/book_detail.html -->
    <div>
      <a href="{% url 'books:book-list' %}">書籍一覧</a>
+     <a href="{% url 'books:book-create' %}">書籍登録</a>
    </div>
{% endblock inner_body %}
```

書籍登録ビューを書籍詳細ビューの上の行でディスパッチします。

```python
# ./books/urls.py
      # 書籍一覧ページ (ex: /books/)
      path("", views.BookListView.as_view(), name="book-list"),
+     # 書籍登録ページ (ex: /books/create/)
+     path("create/", views.BookCreateView.as_view(), name="book-create"),
      # 書籍詳細ページ (ex: /books/<id>/)
      path("<str:pk>/", views.BookDetailView.as_view(), name="book-detail"),
```

書籍登録ビューが正常に機能したら、次の通り変更をコミットします。

```bash
git add --all
git commit -m '書籍登録ビューを実装'
```

> commit 1ae1343726f9c7f8c5c0f98a5692831e9624735a

### 書籍更新ページの実装

書籍更新ビューを次の通り実装します。

```python
class BookUpdateView(
    BookViewMixin,
    PageTitleMixin,
    FormActionMixin,
    generic.UpdateView,
):
    """書籍更新ビュー"""

    form_class = BookForm
    title = "書籍更新"
    action = "更新"

    def get_initial(self) -> Dict[str, Any]:
        initial = super().get_initial()
        initial["classification"] = self.object.classification_detail.classification
        return initial

    def get_context_data(self, **kwargs: Any) -> Dict[str, Any]:
        ctx = super().get_context_data(**kwargs)
        ctx["classification_details"] = list(
            ClassificationDetail.objects.all().values("code", "classification", "name")
        )
        return ctx

    def get_success_url(self) -> str:
        return reverse("book-detail", kwargs={"pk": self.object.id})
```

書籍フォームテンプレートを次の通り2箇所変更します。

```html
    <div>
      <a href="{% url 'books:book-list' %}">書籍一覧</a>
+     {% if action == "更新" %}
+       <a href="{% url 'books:book-detail' form.instance.id %}">書籍詳細</a>
+     {% endif %}
    </div>

  (...省略...)

      // ページがロードされた後に実行される関数を登録
      window.onload = () => {
        setClassificationDetailOptions();
+       // 書籍分類詳細フォームフィールドを取得
+       const select = document.getElementById("{{ form.classification_detail.auto_id }}");
+       // 書籍更新ページの場合は、setClassificationDetailOption関数によって、書籍の書籍分類詳細がクリアされるため
+       // 書籍分類詳細を再設定
+       if ("{{ action }}" === "更新") {
+         select.value = "{{ book.classification_detail.code }}";
+       }
      }
```

書籍一覧ページに書籍更新ページへのリンクを設置するために、書籍一覧テンプレートを次の通り変更します。

```html
<!-- ./books/templates/books/book_list.html -->
            <td>
              <a href="{% url 'books:book-detail' pk=book.id %}">詳細</a>
+             <a href="{% url 'books:book-update' pk=book.id %}">更新</a>
            </td>
```

書籍詳細ページに書籍更新ページへのリンクを設置するために、書籍詳細テンプレートを次の通り変更します。

```html
<!-- ./books/templates/books/book_detail.html -->
    <div>
      <a href="{% url 'books:book-list' %}">書籍一覧</a>
      <a href="{% url 'books:book-create' %}">書籍登録</a>
+     <a href="{% url 'books:book-update' pk=book.id %}">書籍更新</a>
    </div>
```

書籍更新ビューをディスパッチするために、`./books/urls.py`に次を追加します。

```python
    # 書籍更新ページ (ex: /books/update/<id>/)
    path("update/<str:pk>/", views.BookUpdateView.as_view(), name="book-update"),
```

書籍更新ビューが正常に機能したら、次の通り変更をコミットします。

```bash
git add --all
git commit -m '書籍更新ページを実装'
```

> commit 6ba5d64c530eb1aebff12f5c4e338a5ac1717e44

### 書籍削除ページの実装

書籍削除ビューを次の通り実装します。

```python
# ./books/views.py
class BookDeleteView(
    BookViewMixin,
    PageTitleMixin,
    generic.DeleteView,
):
    """書籍削除ビュー"""

    title = "書籍削除"
    success_url = reverse_lazy("book-list")
```

書籍削除テンプレートを次の通り実装します。

```html
<!-- ./books/templates/books/book_confirm_delete.html -->
{% extends 'base.html' %}

{% block inner_body %}
  <h1>書籍削除</h1>
  <p>この書籍を削除しますか?</p>
  <form method="post">
    {% csrf_token %}
    {% include "books/_book_detail.html" %}
    <button type="submit">削除</button>
  </form>
  <div>
    <a href="{% url 'books:book-list' %}">書籍一覧</a>
    <a href="{% url 'books:book-detail' pk=book.id %}">書籍詳細</a>
    <a href="{% url 'books:book-update' pk=book.id %}">書籍更新</a>
  </div>
{% endblock inner_body %}
```

書籍一覧ページから書籍削除ページへのリンクを次の通り設置します。

```html
<!-- books/templates/books/book_list.html -->
            <td>
              <a href="{% url 'books:book-detail' pk=book.id %}">詳細</a>
              <a href="{% url 'books:book-update' pk=book.id %}">更新</a>
+             <a href="{% url 'books:book-delete' pk=book.id %}">削除</a>
            </td>
```

書籍詳細ページから書籍削除ページへのリンクを次の通り設置します。

```html
<!-- books/templates/books/book_detail.html -->
    <div>
      <a href="{% url 'books:book-list' %}">書籍一覧</a>
      <a href="{% url 'books:book-create' %}">書籍登録</a>
      <a href="{% url 'books:book-update' pk=book.id %}">書籍更新</a>
+     <a href="{% url 'books:book-delete' pk=book.id %}">書籍削除</a>
    </div>
```

書籍更新ページから書籍削除ページへのリンクを次の通り設置します。

```html
<!-- books/templates/books/book_update.html -->
      <a href="{% url 'books:book-list' %}">書籍一覧</a>
      {% if action == "更新" %}
        <a href="{% url 'books:book-detail' form.instance.id %}">書籍詳細</a>
+       <a href="{% url 'books:book-delete' form.instance.id %}">書籍削除</a>
      {% endif %}
```

書籍削除ビューをディスパッチするために、`./books/urls.py`に次を追加します。

```python
# ./books/urls.py
      path("<str:pk>/", views.BookDetailView.as_view(), name="book-detail"),
      # 書籍更新ページ (ex: /books/update/<id>/)
      path("update/<str:pk>/", views.BookUpdateView.as_view(), name="book-update"),
+     # 書籍削除ページ (ex: /books/book/<id>/)
+     path("delete/<str:pk>/", views.BookDeleteView.as_view(), name="book-delete"),
  ]
```

書籍削除ページで書籍が削除できること、またそれぞれの書籍ページに設置したリンクが機能することを確認できたら、次の通り変更をリポジトリにコミットします。

```bash
git add --all
git commit -m '書籍削除ページを実装'
```

> commit 3c5c298cd616ae9eed54682b0976ac77b2d22736

## トランザクション

それぞれのモデルインスタンスの登録、更新及び削除をアトミックにするためにトランザクションを次の通り設定します。

次は、書籍登録ビューをトランザクションを実行する例ですが、書籍更新及び書籍削除ビューでも、また、それぞれのモデルについても同様です。

```python
# ./books/views.py
  from typing import Any, Dict, Optional

  from django import forms
+ from django.db import transaction
  from django.db.models import QuerySet
- from django.http import HttpRequest
+ from django.http import HttpRequest, HttpResponse
  from django.urls import reverse, reverse_lazy
  from django.views import generic

(...省略...)

  class BookCreateView(
      BookViewMixin,
      PageTitleMixin,
      FormActionMixin,
      generic.CreateView,
  ):
      """書籍登録ビュー"""

      form_class = BookForm
      title = "書籍登録"
      action = "登録"

+     @transaction.atomic
+     def post(self, request: HttpRequest, *args: str, **kwargs: Any) -> HttpResponse:
+         return super().post(request, *args, **kwargs)
+
      def get_context_data(self, **kwargs: Any) -> Dict[str, Any]:
          ctx = super().get_context_data(**kwargs)
          ctx["classification_details"] = list(
              ClassificationDetail.objects.all().values("code", "classification", "name")
          )
          return ctx

      def get_success_url(self) -> str:
          return reverse("book-detail", kwargs={"pk": self.object.id})
```

`@transaction.atomic`は`デコレーター｀です。
デコレーターは、関数やメソッドをラップ（`wrap`、覆って）、ラップした関数またはメソッドの前後で、何らかの処理を実行できます。

上記では、`@transaction.atomic`デコレーターは、ラップした`post`メソッドを処理する前にトランザクションを開始して、処理した後にトランザクションをコミットまたはロールバックします。

トランザクジョンを設定後、それぞのモデルを登録、更新及び削除するページが正常に機能したら、次の通り変更をリポジトリにコミットします。

```bash
git add books/views.py
git add divisions/views.py
git commit -m 'それぞれのビューでトランザクションを設定'
```

> commit 7eaa3a4bba336b1839d43c0c96c8d2e0846a5935

## まとめ

本書では、書籍を一覧及び詳細表示して、書籍を登録、更新及び削除するページを実装しました。
また、書籍を更新するページでは、トランザクションを開始することで、書籍の操作をアトミックにしました。

次の章は、本章で実装した書籍ページを[Bootstrap](https://getbootstrap.jp/)を使用して装飾及びレイアウトします。
