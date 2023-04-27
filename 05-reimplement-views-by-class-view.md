# クラスビューによる部署アプリのビューの再実装

- [クラスビューによる部署アプリのビューの再実装](#クラスビューによる部署アプリのビューの再実装)
  - [部署一覧関数ビューを部署一覧クラスビューで再実装](#部署一覧関数ビューを部署一覧クラスビューで再実装)
    - [モデルの指定](#モデルの指定)
    - [表示するQuerySetの取得](#表示するquerysetの取得)
    - [テンプレートのパスとコンテキスト名の指定](#テンプレートのパスとコンテキスト名の指定)
    - [get\_context\_dataメソッドのオーバーライド](#get_context_dataメソッドのオーバーライド)
  - [部署詳細関数ビューを部署詳細クラスビューで再実装](#部署詳細関数ビューを部署詳細クラスビューで再実装)
  - [部署登録関数ビューを部署登録クラスビューで再実装](#部署登録関数ビューを部署登録クラスビューで再実装)
  - [部署更新関数ビューを部署更新クラスビューで再実装](#部署更新関数ビューを部署更新クラスビューで再実装)
  - [部署削除関数ビューを部署削除クラスビューで再実装](#部署削除関数ビューを部署削除クラスビューで再実装)
  - [部署アプリビューのリファクタリング](#部署アプリビューのリファクタリング)
    - [モデルの指定](#モデルの指定-1)
    - [URLキーワードの設定](#urlキーワードの設定)
    - [部署フォームの設定](#部署フォームの設定)
    - [ページタイトルの設定](#ページタイトルの設定)
    - [部署登録及び部署更新ページの操作名](#部署登録及び部署更新ページの操作名)
  - [Pythonにおける多重継承のメソッド解決順序（Method Resolution Order: MRO）](#pythonにおける多重継承のメソッド解決順序method-resolution-order-mro)
    - [メソッド解決順序](#メソッド解決順序)
    - [実際のメソッドの呼び出し](#実際のメソッドの呼び出し)
    - [\`CreateDivisionViewのMROとメソッド呼び出し](#createdivisionviewのmroとメソッド呼び出し)
  - [まとめ](#まとめ)

前の章で部署アプリのビューを関数ビューで実装しましたが、それを[クラスビュー (Class-based views)](https://docs.djangoproject.com/en/4.2/topics/class-based-views/)で再実装します。

Djangoが提供するクラスビューは、ジェリックでどのモデルにも適用できます。
クラスビューは、関数ビューと比較して実装するコードが少なくなります。

> クラスビューのデフォルトの振る舞いを変更する場合、クラスビューのメソッドをオーバーライドする必要があり、コードが増えることもあります。

## 部署一覧関数ビューを部署一覧クラスビューで再実装

[ListView](https://docs.djangoproject.com/en/4.2/ref/class-based-views/generic-display/#listview)は、特定のモデルの複数のモデルインスタンスを表示する機能を提供するクラスビューです。
部署一覧関数ビューを次の通り`ListView`で再実装します。

```python
# ./divisions/views.py
+ from typing import Any, Dict
+
  from django.db import transaction
  from django.http import HttpRequest, HttpResponse
  from django.shortcuts import get_object_or_404, redirect, render
+ from django.views import generic

  from .forms import DivisionForm
  from .models import Division


- def list(request: HttpRequest) -> HttpResponse:
-     """部署一覧関数ビュー"""
-
-     # すべての部署をデータベースから取得
-     divisions = Division.objects.all()
-     # コンテキストを作成
-     context = {
-         "title": "部署一覧",
-         "division_list": divisions,
-     }
-     # コンテキストでテンプレートをレンダリング
-     return render(request, "divisions/division_list.html", context)

+ class DivisionListView(generic.ListView):
+     """部署一覧クラスビュー"""
+     model = Division
+
+     def get_context_data(self, **kwargs: Any) -> Dict[str, Any]:
+         ctx = super().get_context_data(**kwargs)
+         ctx["title"] = "部署一覧"
+         return ctx
```

### モデルの指定

部署一覧クラスビューでは、ビューが部署モデルを扱うことをクラス変数`model`で指定しています。

### 表示するQuerySetの取得

部署一覧クラスビューに部署をデータベースから取得する実装がありませんが、それは`ListViews`（正確には`BaseListView`）が`get`メソッド内で次を処理しているからです。

- `MultipleObjectMixin`の`get_queryset`メソッドを呼び出して、表示するモデルインスタンスのQuerySetを取得して、メンバ変数`object_list`に設定します。
- `MultipleObjectMixin`の`get_context_data`メソッドを呼び出して、上記で取得したQuerySetを追加したコンテキストを取得します。

### テンプレートのパスとコンテキスト名の指定

実際のところ、部署一覧関数ビューを実装するとき、テンプレートのパス、テンプレートで使用するコンテキスト名などは**意図的に**指定しました。

`MultipleObjectMixin`の`get_context_data`メソッドは、自身の`get_context_object_name`メソッドを呼び出して、上記QuerySetを格納するコンテキスト名を取得します。
この`get_context_object_name`メソッドは、`context_object_name`クラス変数が設定されていない場合、小文字に変換したモデル名の末尾に`_list`を追加した文字列を返却して、その文字列がコンテキスト名になります。
部署一覧テンプレートにおいて、そのコンテキスト名は`division_list`です。

テンプレートのパスも、`MultipleObjectMixin`の`get_template_names`メソッドが、コンテキスト名と同様な処理で`divisions/division_list.html`になります。

### get_context_dataメソッドのオーバーライド

HTMLの`head`要素の`title`要素のコンテンツに設定したいタイトルは、本Webアプリケーション独自の仕様です。
このため、部署一覧クラスビューで`get_context_data`メソッドをオーバーライドして、`MultipleObjectMixin`の`get_context_data`メソッドを呼び出し（`super().get_context_data(**kwargs)`）て、Djangoが作成したコンテキストを取得します。
その後、取得したコンテキストに`title`という名前でタイトル名を追加しています。

> クラスビューのデフォルトの振る舞いを確認したい場合は、次を参照してください。
> 特に、`Classy Class-Based Views.`は確認しやすく、非常に便利です。
> Djangoのソースコードを確認する場合は、クラスビューのベースクラスである`Mixin`クラスを参照してください。
> Djangoのジェネリックビューの処理のほとんどは、`Mixin`クラスで実装されています。
>
> - [Classy Class-Based Views.](https://ccbv.co.uk/)
> - [Django - Document](https://docs.djangoproject.com/en/4.2/ref/class-based-views/)
> - [Django Generic View - GitHub](https://github.com/django/django/tree/main/django/views/generic)

部署一覧クラスビューがディスパッチされるように、`./divisions/urls.py`を次の通り変更します。

```python
# ./divisions/urls.py
      # 部署一覧ページ(ex: /divisions/)
-     path("", views.list, name="division-list"),
+     path("", views.DivisionListView.as_view(), name="division-list"),
```

開発用サーバーを起動して、ブラウザで部署一覧ページにアクセスしてください。
関数ビューで実装したときと同じ部署一覧ページが表示されるはずです。

部署一覧ビューをクラスビューで再実装した結果を、次の通りリポジトリにコミットします。

```bash
git add --all
git commit -m '部署一覧関数ビューを部署一覧クラスビューで再実装'
```

> commit 52685bcad7f75cf8947affeb6f3f15a472db8e33 (tag: 018-部署一覧関数ビューを部署一覧クラスビューで再実装)

## 部署詳細関数ビューを部署詳細クラスビューで再実装

[DetailView](https://docs.djangoproject.com/en/4.2/ref/class-based-views/generic-display/#detailview)は、特定のモデルインスタンスの属性を表示する機能を提供するクラスビューです。
部署詳細関数ビューを次の通り`DetailView`で再実装します。

```python
# ./divisions/views.py
- def detail(request: HttpResponse, code: str) -> HttpResponse:
-     """部署詳細関数ビュー
-
-     Args:
-         code: 部署コード
-     """
-
-     # 部署コードから部署モデルインスタンスを取得
-     division = get_object_or_404(Division, pk=code)
-     # コンテキストを渡してテンプレートをレンダリング
-     return render(
-         request,
-         "divisions/division_detail.html",
-         {"title": "部署詳細", "division": division},
-     )
+
+ class DivisionDetailView(generic.DetailView):
+     """部署詳細クラスビュー"""
+
+     model = Division
+     pk_url_kwarg = "code"
+
+     def get_context_data(self, **kwargs: Any) -> Dict[str, Any]:
+         ctx = super().get_context_data(**kwargs)
+         ctx["title"] = "部署詳細"
+         return ctx
```

部署詳細クラスビューは、部署一覧クラスビューと同様に自動的にテンプレートやコンテキストを`DetailView`が解決します。

ただし、`DetailView`が取り扱うURLパターンで扱う主キーの識別子は、`DetailView`のベースクラスである`SingleObjectMixin`で`pk`となっています。
`SingleObjectMixin`の実装をオーバーライドするため、部署詳細クラスビューのクラス変数`pk_url_kwarg`に`code`を設定しています。

部署詳細クラスビューがディスパッチされるように、`./divisions/urls.py`を変更します。

```python
# ./divisions/urls.py
      # 部署詳細ページ(ex: /divisions/<code>/)
-     path("<str:code>/", views.detail, name="division-detail"),
+     path("<str:code>/", views.DivisionDetailView.as_view(), name="division-detail"),
```

開発用サーバーを起動して、ブラウザで部署詳細ページにアクセスしてください。
関数ビューで実装したときと同じ部署詳細ページが表示されるはずです。

部署詳細ビューをクラスビューで再実装した結果を、次の通りリポジトリにコミットします。

```bash
git add ./divisions/
git commit -m '部署詳細関数ビューを部署詳細クラスビューで再実装'
```

> commit 19f606c93c7c271df3408801f2732f296f13dc42 (tag: 019-部署詳細関数ビューを部署詳細クラスビューで再実装)

## 部署登録関数ビューを部署登録クラスビューで再実装

[CreateView](https://docs.djangoproject.com/en/4.2/ref/class-based-views/generic-editing/#createview)は、特定のモデルのモデルインスタンスを登録する機能を提供するクラスビューです。
部署登録関数ビューを次の通り`CreateView`で再実装します。

```python
# ./divisions/views.py
  from django.http.request import HttpRequest
  from django.http.response import HttpResponse
  from django.shortcuts import get_object_or_404, redirect, render
+ from django.urls import reverse
  from django.views import generic

 (...省略...)

- def create(request: HttpResponse) -> HttpResponse:
-     """部署登録関数ビュー"""
-
-     if request.method == "POST":
-         # POSTパラメーターから部署フォームを構築
-         form = DivisionForm(request.POST)
-         if form.is_valid():
-             with transaction.atomic():
-                 division = form.save()
-             return redirect("division-detail", code=division.code)
-     else:  # request.method is "GET", maybe.
-         # フォームを構築
-         form = DivisionForm()
-     # コンテキストを渡してテンプレートをレンダリング
-     return render(
-         request,
-         "divisions/division_form.html",
-         {"title": "部署登録", "form": form, "action": "登録"},
-     )

+
+ class DivisionCreateView(generic.CreateView):
+     """部署登録クラスビュー"""
+
+     model = Division
+     pk_url_kwarg = "code"
+     fields = ("code", "name")
+
+     def get_context_data(self, **kwargs: Any) -> Dict[str, Any]:
+         ctx = super().get_context_data(**kwargs)
+         ctx["title"] = "部署登録"
+         ctx["action"] = "登録"
+         return ctx
+
+     def get_success_url(self) -> str:
+         return reverse("divisions:division-detail", kwargs={"code": self.object.code})
```

部署登録クラスビューは、部署一覧クラスビューと同様に自動的にテンプレートやコンテキストを`CreateView`が解決します。
また、部署登録ページに表示するフォームも`CreateView`のベースクラスである`ModelFromMixin`の`get_form_class`メソッドで解決します。

`CreateView`は、`ProcessFormView`の`post`メソッド内で、フォームの検証に成功したとき`ModelFormMixin`の`form_valid`メソッドを呼び出します。
`form_valid`メソッドでは、フォームから生成した部署モデルインスタンスをデータベースに登録して、登録した部署モデルインスタンスをメンバ変数`object (self.object)`に設定します。

部署登録クラスビューの`get_success_url`メソッドは、上記で設定されたメンバ変数`object`を参照して、`reverse`関数でリダイレクト先のURLを構築しています。

部署登録クラスクラスビューがディスパッチされるように、`./divisions/urls.py`を変更します。

```python
# ./divisions/urls.py
      # 部署登録ページ(ex: /divisions/create/)
-     path("create/", views.create, name="division-create"),
+     path("create/", views.DivisionCreateView.as_view(), name="division-create"),
```

開発用サーバーを起動して、ブラウザで部署登録ページにアクセスしてください。
関数ビューで実装したときと同様に部署を登録できるはずです。

部署登録ビューをクラスビューで再実装した結果を、次の通りリポジトリにコミットします。

```bash
git add --all
git commit -m '部署登録関数ビューを部署登録クラスビューで再実装'
```

> commit 20d60867cfbdb853ecc0d805b3c0a502bc13d793 (tag: 020-部署登録関数ビューを部署登録クラスビューで再実装)

## 部署更新関数ビューを部署更新クラスビューで再実装

[UpdateView](https://docs.djangoproject.com/en/4.2/ref/class-based-views/generic-editing/#updateview)は、特定のモデルインスタンスを更新する機能を提供するクラスビューです。
部署更新関数ビューを次の通り`UpdateView`で再実装します。

```python
# ./divisions/views.py
- from typing import Any, Dict
+ from typing import Any, Dict, Type

+ from django import forms
  from django.db import transaction
  from django.http.request import HttpRequest
  from django.http.response import HttpResponse
  from django.shortcuts import get_object_or_404, redirect, render
  from django.urls import reverse
  from django.views import generic

- from .forms import DivisionForm
  from .models import Division

  (...省略...)

- def update(request: HttpRequest, code: str) -> HttpResponse:
-     """部署更新ビュー
-
-     Args:
-         code: 部署コード
-     """
-     # 部署コードから部署モデルインスタンスを取得
-     division = get_object_or_404(Division, pk=code)
-     if request.method == "POST":
-         # POSTパラメーターから部署フォームを構築
-         form = DivisionForm(request.POST, instance=division)
-         if form.is_valid():
-             division = form.save(commit=False)
-             division.code = code
-             with transaction.atomic():
-                 division.save()
-             return redirect("division-detail", code=division.code)
-     else:  # request.method is "GET", maybe.
-         # 部署モデルインスタンスから部署フォームを構築
-         form = DivisionForm(instance=division)
-     form.fields["code"].widget.attrs["readonly"] = True
-     return render(
-         request,
-         "divisions/division_form.html",
-         {"title": "部署更新", "form": form, "action": "更新"},
-     )
+
+ class DivisionUpdateView(generic.UpdateView):
+     """部署更新クラスビュー"""
+
+     model = Division
+     pk_url_kwarg = "code"
+     fields = ("code", "name")
+
+     def get_context_data(self, **kwargs: Any) -> Dict[str, Any]:
+         ctx = super().get_context_data(**kwargs)
+         ctx["title"] = "部署更新"
+         ctx["action"] = "更新"
+         return ctx
+
+     def get_form(self, form_class: Type[forms.Form] | None = None) -> forms.Form:
+         form = super().get_form(form_class=form_class)
+         form.fields["code"].widget.attrs["readonly"] = True
+         return form
+
+     def get_success_url(self) -> str:
+         return reverse("divisions:division-detail", kwargs={"code": self.object.code})
```

部署更新クラスビューは、部署登録クラスビューと同様です。
部署更新クラスビューと部署登録クラスビューの異なる点は、部署更新クラスビューの`get_form`メソッドで、部署フォームを`UpdateView`のベースクラスである`FormMixin`の`get_form`メソッドで取得して、部署フォームの部署コードフィールドを読み込み専用に設定しています。

部署更新クラスクラスビューがディスパッチされるように、`./divisions/urls.py`を変更します。

```python
# ./divisions/urls.py
      # 部署更新ページ (ex: /divisions/update/<code>/)
-     path("update/<str:code>/", views.update, name="division-update"),
+     path(
+             "update/<str:code>/", views.DivisionUpdateView.as_view(), name="division-update"
+         ),
```

開発用サーバーを起動して、ブラウザで部署更新ページにアクセスしてください。
関数ビューで実装したときと同様に部署を更新できるはずです。

部署更新ビューをクラスビューで再実装した結果を、次の通りリポジトリにコミットします。

```bash
git add ./divisions/
git commit -m '部署更新関数ビューを部署更新クラスビューで再実装'
```

> commit 13b9086dd5e8a5a175ac6a64677985f73c5b04db (tag: 021-部署更新関数ビューを部署更新クラスビューで再実装)

## 部署削除関数ビューを部署削除クラスビューで再実装

[DeleteView](https://docs.djangoproject.com/en/4.2/ref/class-based-views/generic-editing/#django.views.generic.edit.DeleteView)は、特定のモデルインスタンスを削除する機能を提供するクラスビューです。
部署削除関数ビューを次の通り`DeleteView`で再実装します。

```python
# ./divisions/views.py
  from django import forms
- from django.db import transaction
- from django.http.request import HttpRequest
- from django.http.response import HttpResponse
- from django.shortcuts import get_object_or_404, redirect, render
  from django.urls import reverse, reverse_lazy
  from django.views import generic

  (...省略...)

- def delete(request: HttpRequest, code: str) -> HttpResponse:
-     """部署削除ビュー
-
-     Args:
-         code: 部署コード
-     """
-
-     # 部署コードから部署モデルインスタンスを取得
-     division = get_object_or_404(Division, pk=code)
-     if request.method == "POST":
-         with transaction.atomic():
-             division.delete()
-         return redirect("division-list")
-     return render(
-         request, "divisions/division_confirm_delete.html", {"division": division}
-     )
+
+ class DivisionDeleteView(generic.DeleteView):
+     """部署削除クラスビュー"""
+
+     model = Division
+     pk_url_kwarg = "code"
+     success_url = reverse_lazy("divisions:division-list")
+
+     def get_context_data(self, **kwargs: Any) -> Dict[str, Any]:
+         ctx = super().get_context_data(**kwargs)
+         ctx["title"] = "部署削除"
+         return ctx
```

`reverse_lazy`関数は、`reverse`関数の遅延評価バージョンで、クラスビューのクラス変数にURLを設定するなど、URLconfが読み込まれる前にURLパターン名からURLを逆引きしたい場合に`reverse`の代わりに使用します。

部署削除クラスクラスビューがディスパッチされるように、`./divisions/urls.py`を変更します。

```python
# ./divisions/urls.py
      # 部署削除ページ (ex: /divisions/delete/<code>/)
-     path("delete/<str:code>/", views.delete, name="division-delete"),
+     path(
+         "delete/<str:code>/", views.DivisionDeleteView.as_view(), name="division-delete"
+     ),
```

開発用サーバーを起動して、ブラウザで部署削除ページにアクセスしてください。
関数ビューで実装したときと同様に部署を削除できるはずです。

部署削除ビューをクラスビューで再実装した結果を、次の通りリポジトリにコミットします。

```bash
git add ./divisions
git commit -m '部署削除関数ビューを部署削除クラスビューで再実装'
```

> commit fc4130834fdcadc0b6446b0f79f905150527931a (tag: 022-部署削除関数ビューを部署削除クラスビューで再実装)

## 部署アプリビューのリファクタリング

`./divisions/views.py`を確認すると、それぞれのビューで実装が重複しています。
`DRY`の精神に従って、重複している実装をリファクタリングします。

### モデルの指定

それぞれのビューに、`model = Division`という実装があります。
これを[ミックスイン](https://ja.wikipedia.org/wiki/Mixin)と呼ばれる技法を使用して、クラスビューが処理するモデルの定義を`DivisionViewMixin`にまとめます。

次の通り`./divisions/views.py`を変更します。

```python
# ./divisions/views.py
+ class DivisionViewMixin:
+     """部署ビューミックスイン"""
+
+     model = Division
+

- class DivisionListView(generic.ListView):
+ class DivisionListView(DivisionViewMixin, generic.ListView):
      """部署一覧クラスビュー"""

-     model = Division

  (...省略...)


- class DivisionDetailView(generic.DetailView):
+ class DivisionDetailView(DivisionViewMixin, generic.DetailView):
      """部署詳細クラスビュー"""

-     model = Division
      pk_url_kwarg = "code"

  (...省略...)


- class DivisionCreateView(generic.CreateView):
+ class DivisionCreateView(DivisionViewMixin, generic.CreateView):
      """部署登録クラスビュー"""

-     model = Division
      pk_url_kwarg = "code"
      fields = ("code", "name")

  (...省略...)

- class DivisionUpdateView(generic.UpdateView):
+ class DivisionUpdateView(DivisionViewMixin, generic.UpdateView):
      """部署更新クラスビュー"""

-     model = Division
      pk_url_kwarg = "code"
      fields = ("code", "name")

  (...省略...)

- class DivisionDeleteView(generic.DeleteView):
+ class DivisionDeleteView(DivisionViewMixin, generic.DeleteView):
      """部署削除クラスビュー"""

-     model = Division
      pk_url_kwarg = "code"
      success_url = reverse_lazy("division-list")
```

部署一覧、詳細、登録、更新及び削除ページが正常に機能することを確認してください。
正常に機能することが確認できたら、次の通り変更をリポジトリにコミットします。

```bash
git add ./divisions/views.py
git commit -m 'DivisionViewMixinで部署モデルを指定'
```

> commit a1d264af515fcb4b0682e1fe37042bf1f0518332 (tag: 023-DivisionViewMixinで部署モデルを指定)

### URLキーワードの設定

部署詳細ビューや部署更新ビューなど、1つの部署モデルインスタンスを取り扱うビューについて、`pk_url_kwarg = "code"`という実装が重複しています。
これは、部署の部署コード（`code`）がプライマリーキーで、URLconfのパスコンバーターで`<str:code>`を指定したため、Djangoのデフォルトの実装と違うため追加したコードでした。
この実装を`DivisionSingleObjectMixin`に次の通りまとめます。

```python
# ./divisions/views.py
  class DivisionViewMixin:
      """部署ビューミックスイン"""

      model = Division


+ class DivisionSingleObjectMixin:
+     """部署シングルオブジェクトミックスイン"""
+
+     pk_url_kwarg = "code"
+
  (...省略...)

- class DivisionDetailView(DivisionViewMixin, generic.DetailView):
+ class DivisionDetailView(
+     DivisionViewMixin, DivisionSingleObjectMixin, generic.DetailView
+ ):
      """部署詳細クラスビュー"""

-     pk_url_kwarg = "code"
-
  (...省略...)

- class DivisionCreateView(DivisionViewMixin, generic.CreateView):
+ class DivisionCreateView(
+     DivisionViewMixin, DivisionSingleObjectMixin, generic.CreateView
+ ):
      """部署登録クラスビュー"""

-     pk_url_kwarg = "code"
      fields = ("code", "name")
  (...省略...)

- class DivisionUpdateView(DivisionViewMixin, generic.UpdateView):
+ class DivisionUpdateView(
+     DivisionViewMixin, DivisionSingleObjectMixin, generic.UpdateView
+ ):
      """部署更新クラスビュー"""

-     pk_url_kwarg = "code"
      fields = ("code", "name")
  (...省略...)

- class DivisionDeleteView(DivisionViewMixin, generic.DeleteView):
+ class DivisionDeleteView(
+     DivisionViewMixin, DivisionSingleObjectMixin, generic.DeleteView
+ ):
      """部署削除クラスビュー"""

-     pk_url_kwarg = "code"
      success_url = reverse_lazy("division-list")
```

部署一覧、詳細、登録、更新及び削除ページが正常に機能することを確認してください。
正常に機能することが確認できたら、次の通り変更をリポジトリにコミットします。

```bash
git add ./divisions/views.py
git commit -m 'DivisionSingleObjectMixinでパスコンバーターのキーワードを設定'
```

> commit 8bf99067d2f0bd2f6f9d0590e87180a444ad71e3 (tag: 024-DivisionSingleObjectMixinでパスコンバーターのキーワードを設定)

### 部署フォームの設定

フィールドなど部署フォームの設定を`DivisionFormMixin`にまとめます。

```python
# ./division/views.py
+ class DivisionFormMixin:
+     """部署フォームドミックスイン"""
+
+     fields = ("code", "name")
+

  (...略...)

- class DivisionCreateView(
-     DivisionViewMixin, DivisionSingleObjectMixin, generic.CreateView
- ):
+ class DivisionCreateView(
+     DivisionViewMixin, DivisionSingleObjectMixin, DivisionFormMixin, generic.CreateView
+ ):
      """部署登録クラスビュー"""

      title = "部署登録"
-     fields = ("code", "name")
      action = "登録"

  (...略...)

- class DivisionUpdateView(
-     DivisionViewMixin, DivisionSingleObjectMixin, generic.UpdateView
- ):
+ class DivisionUpdateView(
+     DivisionViewMixin, DivisionSingleObjectMixin, DivisionFormMixin, generic.UpdateView
+ ):
      """部署更新クラスビュー"""

      title = "部署更新"
-     fields = ("code", "name")
      action = "更新"
```

部署登録及び部署更新ページが正常に部署を登録、または更新できることを確認してください。
正常に機能することが確認できたら、次の通り変更をリポジトリにコミットします。

```bash
git add ./divisions/views.py
git commit -m 'DivisionFormMixinで部署フォームを設定'
```

> commit 230417362548e832188c61d2bae12a76729c44c7 (HEAD -> main, tag: 025-DivisionFormMixinで部署フォームを設定)

### ページタイトルの設定

それぞれのビューの`get_context_data`メソッドで、ページのタイトルをコンテキストに設定するように`PageTitleMixin`にまとめます。
`PageTitleMixin`は、`get_context_data`を実装する`ContextMixin`から派生します。

```python
# ./division/views.py
+ class PageTitleMixin(generic.base.ContextMixin):
+     """ページタイトルミックスイン"""
+
+     title = None
+
+     def get_context_data(self, **kwargs: Any) -> Dict[str, Any]:
+         ctx = super().get_context_data(**kwargs)
+         ctx["title"] = self.title
+         return ctx
+

- class DivisionListView(DivisionViewMixin, generic.ListView):
+ class DivisionListView(DivisionViewMixin, PageTitleMixin, generic.ListView):
      """部署一覧クラスビュー"""

+     title = "部署一覧"

-     def get_context_data(self, **kwargs: Any) -> Dict[str, Any]:
-         ctx = super().get_context_data(**kwargs)
-         ctx["title"] = "部署一覧"
-         return ctx

- class DivisionDetailView(
-     DivisionViewMixin, DivisionSingleObjectMixin, generic.DetailView
- ):
+ class DivisionDetailView(
+     DivisionViewMixin, DivisionSingleObjectMixin, PageTitleMixin, generic.DetailView
+ ):
      """部署詳細クラスビュー"""

+     title = "部署詳細"

-     def get_context_data(self, **kwargs: Any) -> Dict[str, Any]:
-         ctx = super().get_context_data(**kwargs)
-         ctx["title"] = "部署詳細"
-         return ctx

- class DivisionCreateView(
-     DivisionViewMixin, DivisionSingleObjectMixin, DivisionFormMixin, generic.CreateView
- ):
+ class DivisionCreateView(
+     DivisionViewMixin,
+     DivisionSingleObjectMixin,
+     DivisionFormMixin,
+     PageTitleMixin,
+     generic.CreateView,
+ ):
      """部署登録クラスビュー"""

+     title = "部署登録"

      def get_context_data(self, **kwargs: Any) -> Dict[str, Any]:
          ctx = super().get_context_data(**kwargs)
-         ctx["title"] = "部署登録"
          ctx["action"] = "登録"
          return ctx

      def get_success_url(self) -> str:
          return reverse("division-detail", kwargs={"code": self.object.code})


- class DivisionUpdateView(
-     DivisionViewMixin, DivisionSingleObjectMixin, DivisionFormMixin, generic.UpdateView
- ):
+ class DivisionUpdateView(
+     DivisionViewMixin,
+     DivisionSingleObjectMixin,
+     DivisionFormMixin,
+     PageTitleMixin,
+     generic.UpdateView,
+ ):
      """部署更新クラスビュー"""

+     title = "部署更新"

      def get_context_data(self, **kwargs: Any) -> Dict[str, Any]:
          ctx = super().get_context_data(**kwargs)
-         ctx["title"] = "部署更新"
          ctx["action"] = "更新"
          return ctx

  (...省略...)

- class DivisionDeleteView(
-     DivisionViewMixin, DivisionSingleObjectMixin, generic.DeleteView
- ):
+ class DivisionDeleteView(
+     DivisionViewMixin, DivisionSingleObjectMixin, PageTitleMixin, generic.DeleteView
+ ):
      """部署削除クラスビュー"""

+     title = "部署削除"
      success_url = reverse_lazy("division-list")

-     def get_context_data(self, **kwargs: Any) -> Dict[str, Any]:
-         ctx = super().get_context_data(**kwargs)
-         ctx["title"] = "部署削除"
-         return ctx
```

部署一覧、部署詳細、部署登録、部署更新及び部署削除ページのタイトルが正常に表示されることを確認してください。
正常に表示することが確認できたら、変更をリポジトリにコミットします。

```bash
git add ./divisions/views.py
git commit -m 'PageTitleMixinでページタイトルを設定'
```

> commit c1fa78bd122756c927708e5d2124ef4eaf558bdb (HEAD -> main, tag: 026-PageTitleMixinでページタイトルを設定)

### 部署登録及び部署更新ページの操作名

部署登録及び部署更新ページの操作名を`FormActionMixin`にまとめます。

```python
# ./divisions/views.py
+ class FormActionMixin(generic.base.ContextMixin):
+     """フォームアクションミックスイン"""
+
+     action = None
+
+     def get_context_data(self, **kwargs: Any) -> Dict[str, Any]:
+         ctx = super().get_context_data(**kwargs)
+         ctx["action"] = self.action
+         return ctx

  (...省略...)

- class DivisionCreateView(
-     DivisionViewMixin,
-     DivisionSingleObjectMixin,
-     DivisionFormMixin,
-     PageTitleMixin,
-     generic.CreateView,
- ):
+ class DivisionCreateView(
+     DivisionViewMixin,
+     DivisionSingleObjectMixin,
+     DivisionFormMixin,
+     PageTitleMixin,
+     FormActionMixin,
+     generic.CreateView,
+ ):
    """部署登録クラスビュー"""

      title = "部署登録"
+     action = "登録"

-     def get_context_data(self, **kwargs: Any) -> Dict[str, Any]:
-         ctx = super().get_context_data(**kwargs)
-         ctx["action"] = "登録"
-         return ctx

      def get_success_url(self) -> str:
          return reverse("division-detail", kwargs={"code": self.object.code})


- class DivisionUpdateView(
-     DivisionViewMixin,
-     DivisionSingleObjectMixin,
-     DivisionFormMixin,
-     PageTitleMixin,
-     generic.UpdateView,
- ):
+ class DivisionUpdateView(
+     DivisionViewMixin,
+     DivisionSingleObjectMixin,
+     DivisionFormMixin,
+     PageTitleMixin,
+     FormActionMixin,
+     generic.UpdateView,
+ ):
      """部署更新クラスビュー"""

      title = "部署更新"
+     action = "更新"

-     def get_context_data(self, **kwargs: Any) -> Dict[str, Any]:
-         ctx = super().get_context_data(**kwargs)
-         ctx["action"] = "更新"
-         return ctx
-
      def get_form(self, form_class: Type[forms.Form] | None = None) -> forms.Form:
          form = super().get_form(form_class=form_class)
          form.fields["code"].widget.attrs["readonly"] = True
          return form

      def get_success_url(self) -> str:
          return reverse("division-detail", kwargs={"code": self.object.code})
```

部署登録及び部署更新ページのページ名とボタンのラベルが正常に表示されることを確認してください。
正常に表示することが確認できたら、変更をリポジトリにコミットします。

```bash
git add ./divisions/views.py
git commit -m 'FormActionMixinで操作名を設定'
```

> 69c91a326e7a730e15684f6b0a15de3cfa243bcc (tag: 027-FormActionMixinで操作名を設定)

## Pythonにおける多重継承のメソッド解決順序（Method Resolution Order: MRO）

### メソッド解決順序

Pythonではクラスの多重継承を許可しています。
多重継承を許可しているプログラミング言語において、多重継承したクラスのメソッドがオーバーライドされている場合、どのクラスから呼び出すメソッドを検索するか決定する必要があります。
この順番は、`メソッド解決順序（Method Resolution Order: MRO）`と呼ばれています。
Pythonでは、直接メソッドが呼び出されたクラス、そのクラスが継承しているクラスの宣言順、その継承しているクラスが継承しているクラスの宣言順・・・で、呼び出すメソッドの順番が決まります。

> 呼び出すメソッドを解決する順番で、実際に呼び出すわけではないことに注意してください。

言葉で説明することが難しいため、次の例で説明します。

```python
class Base:
    def foo(self) -> None:
        print("Base's foo was called.")


class A(Base):
    def foo(self) -> None:
        print("A's foo was called.")
        super().foo()


class B(Base):
    def foo(self) -> None:
        print("B's foo was called.")
        super().foo()


class C(A, B):
    def foo(self) -> None:
        print("C's foo was called.")
        super().foo()
```

`C`は、`A`、`B`の順番で継承しており、`A`と`B`は`Base`を継承しています。
`C`のMROは、次のように確認できます。
なお、出力はわかりやすいように整形しています。

```python
>>> C.__mro__
class 'divisions.tests.C'
class 'divisions.tests.A'
class 'divisions.tests.B'
class 'divisions.tests.Base'
class 'object'
```

当然ながら、`C`のインスタンスのメソッドを呼び出した場合、`C`が呼び出し候補となります。
`C`にそのメソッドが定義されていない場合、次に候補となるクラスは、`C`が継承している順番が優先されるため`A`です。
`A`にそのメソッドが定義されていない場合、次に候補となるクラスは、上記理由で`B`です。
`B`にそのメソッドが定義されていない場合、次に候補となるクラスは`A`が継承している`Base`です。
`Base`にそのメソッドが定義されていない場合、次に候補となるクラスは、すべての型の基底クラスである`object`です。
そして、`object`にそのメソッドが定義されていない場合、`AttributeError: 'C' object has no attribute 'foo'`と例外が発生します。

> つまり、`C` > `A` > `B` > `Base` > `object`の順番です。

### 実際のメソッドの呼び出し

`C`の`foo`メソッドを呼び出すと次の通り出力されます。

```python
>>> C().foo()
C's foo was called.
A's foo was called.
B's foo was called.
Base's foo was called.
```

`A`と`B`は`super().foo()`を呼び出していますが、`Base`の`foo`メソッドは1回しか呼び出されていません。

先ほどの`Base`、`A`、`B`及び`C`クラスを実装しているモジュールに次のクラスを定義します。

- `Base`と同様に（`object`以外で）最も基底となる`OtherBase`
- `OtherBase`から派生した`D`
- `C`、`D`の順番で継承した`E`
- `E`とは逆に`D`、`C`の順番で継承した`F`

```python
class OtherBase:
    def foo(self) -> None:
        print("OtherBase's foo was called.")


class D(OtherBase):
    def foo(self) -> None:
        print("D's foo was called.")
        super().foo()


class E(C, D):
    def foo(self) -> None:
        print("E's foo was called.")
        super().foo()


class F(D, C):
    def foo(self) -> None:
        print("F's foo was called.")
        super().foo()
```

`E`と`F`のMROは次の通りです。

```python
>>> E.__mro__
class 'divisions.tests.E'
class 'divisions.tests.C'
class 'divisions.tests.A'     # Dではない
class 'divisions.tests.B'
class 'divisions.tests.Base'
class 'divisions.tests.D'
class 'divisions.tests.OtherBase'
class 'object'
>>> F.__mro__
class 'divisions.tests.F'
class 'divisions.tests.D'
class 'divisions.tests.OtherBase'   # Cではない
class 'divisions.tests.C'
class 'divisions.tests.A'
class 'divisions.tests.B'
class 'divisions.tests.Base'
class 'object'
```

先ほど、**継承された順番が優先**されると述べましたが、`E`と`F`では継承された順番でなく、`E`または`F`が継承したクラスが優先されています。

この理由を調べましたが、ドキュメントを見つけることができず理解できていませんが、`E`がオーバーライドしているメソッドの根源が`Base`の`foo`であり、`F`がオーバーライドしているメソッドの根元が`OtherBase`の`foo`であるからと考えています。

`E`と`F`の`foo`メソッドを呼び出すと、次の通り出力されます。

```python
>>> E().foo()
E's foo was called.
C's foo was called.
A's foo was called.
B's foo was called.
Base's foo was called.
>>>
>>> F().foo()
F's foo was called.
D's foo was called.
OtherBase's foo was called.
```

`F`で説明すると、MROは`Base`まで繋がっていますが、`OtherBase`で基本クラスの`foo`メソッドを呼び出していないため、出力された内容は当然の結果です。

次のように、`Other`、`D`及び`F`の定義を変更すると、MROにより`C`以降のメソッドが呼び出されます。

```python
class OtherBase:
    pass


class D(OtherBase):
    pass


class F(D, C):
    pass
```

```python
>>> F().foo()
C's foo was called.
A's foo was called.
B's foo was called.
Base's foo was called.
```

### `CreateDivisionViewのMROとメソッド呼び出し

`CreateDivisionView`は、`django.generic.base.ContextMixin`から派生した`PageTitleMixin`と`FormActionMixin`を継承しています。

`CreateDivisionView`のMROは次の通りです。

```python
>>> DivisionCreateView.__mro__
class 'divisions.views.DivisionCreateView'
class 'divisions.views.DivisionViewMixin'
class 'divisions.views.DivisionSingleObjectMixin'
class 'divisions.views.DivisionFormMixin'
class 'divisions.views.PageTitleMixin'
class 'divisions.views.FormActionMixin'
class 'django.views.generic.edit.CreateView'
class 'django.views.generic.detail.SingleObjectTemplateResponseMixin'
class 'django.views.generic.base.TemplateResponseMixin'
class 'django.views.generic.edit.BaseCreateView'
class 'django.views.generic.edit.ModelFormMixin'
class 'django.views.generic.edit.FormMixin'
class 'django.views.generic.detail.SingleObjectMixin'
class 'django.views.generic.base.ContextMixin'  # ContextMixin
class 'django.views.generic.edit.ProcessFormView'
class 'django.views.generic.base.View'
class 'object'
```

`ContextMixin`の`get_context_data`メソッドに`print`関数を追加して、部署登録ページを表示したときの開発サーバーの主要な出力を次に示します。

```text
ContextMixin's get_context_data was called.
[22/Apr/2023 15:41:20] "GET /divisions/create/ HTTP/1.1" 200 10256
[22/Apr/2023 15:41:31] "POST /divisions/create/ HTTP/1.1" 302 0
ContextMixin's get_context_data was called.
[22/Apr/2023 15:41:31] "GET /divisions/98/ HTTP/1.1" 200 9916
```

部署登録ページを取得（`GET`）するときと、登録する部署を送信（`POST`）するときで、`ContextMixin`の`get_context_data`の呼び出しは、前節の`C`の`foo`メソッド呼び出しと同様に1回であり、多重継承したベースクラスのメソッドの呼び出しは1回だけであることがわかります。

> Pythonで多重継承するクラスは、MROに注意して定義する必要があります。

## まとめ

前の章で部署アプリのビューを関数ビューで実装しましたが、それらをクラスビューで再実装しました。
また、それぞれの部署クラスビューで重複する項目をミックスインでリファクタリングしました。
クラスビューは関数ビューよりもDjangoからの支援を受けやすく、オブジェクト思考により実装内容がより明確になります。

次の章以降では、書籍アプリを追加して、書籍分類、書籍分類詳細及び書籍モデルを実装して、それらを操作するクラスビューを実装します。
