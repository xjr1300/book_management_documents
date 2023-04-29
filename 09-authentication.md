# 認証

- [認証](#認証)
  - [アカウントアプリの追加](#アカウントアプリの追加)
  - [独自ユーザーモデルの実装](#独自ユーザーモデルの実装)
  - [ユーザーモデルを管理サイトに登録](#ユーザーモデルを管理サイトに登録)
  - [ログインページの実装](#ログインページの実装)
  - [ユーザーの表示とログアウト機能の実装](#ユーザーの表示とログアウト機能の実装)
  - [アクセス制限の実装](#アクセス制限の実装)
    - [カスタムLoginRequiredMixinの実装](#カスタムloginrequiredmixinの実装)
    - [書籍登録、更新及び削除ページのアクセス制限](#書籍登録更新及び削除ページのアクセス制限)
    - [書籍一覧及び書籍詳細ページのリンクの整理](#書籍一覧及び書籍詳細ページのリンクの整理)
  - [まとめ](#まとめ)

Djangoは、`django.contrib.auth.models.User`でユーザーモデルを提供しています。
Djangoが提供するユーザーモデルは、ユーザー名（`username`）モデルフィールドが含まれています。
しかし、ユーザーの識別にEメールアドレスを使用するWebアプリケーションにとって、ユーザー名は必要ありません。

よって、本Webアプリケーションは、独自のユーザーモデルを定義します。

Djangoが提供する機能を最大限に利用して独自にユーザーモデルを定義する場合、Userモデルが継承する`django.contrib.auth.models.AbstractUser`または、`django.contrib.auth.models.AbstractBaseUser`を継承してユーザーモデルを実装します。

`AbstractBaseUser`は、認証機能を提供しており、`password`及び`last_login`モデルフィールドを持っています。
なお、名前が`password`となっていますが、Djangoはパスワードをそのままデータベースに記録しません。
`password`フィールドは、`ソルト（塩）`と呼ばれるランダムな文字列をパスワードに追加した文字列を数回ハッシュ化したハッシュ値を記録しています。
これにより、データベースがハッキングされても、そのハッシュ値からパスワードに戻すことを難しくしています。
同様な理由で、システム管理者（`is_superuser = True`）は、ユーザーのパスワードを知ることができません。

> たった1ビット`0`から`1`に変更してもハッシュ値は大きく異なる（そのような特性を持たなくてはならない）ため、総当たりでハッシュか前のデータを得ることも困難です。

`AbstractUser`は`AbstractBaseUser`を継承しており、`AbstractBaseUser`が持つ属性に加えて次の属性を持ちます。

- `username`
- `first_name`
- `last_name`
- `email`
- `is_staff`
- `is_active`
- `data_joined`

`AbstractUser`が持つ属性に独自な属性を追加して、ユーザー名とパスワードで認証するなどする場合は、`AbstractUser`を継承した独自ユーザーモデルを実装します。

たとえば、Eメールアドレスとパスワードで認証するなど`AbstractUser`が提供する機能以外を実装したい場合は、`AbstractBaseUser`を継承した独自ユーザーモデルを実装します。

本アプリケーションでは、Eメールアドレスを使用してユーザー認証をしたいため、`AbstractBaseUser`を継承したユーザーモデルを実装します。

> 独自ユーザーモデルの実装は、[ここ](https://dev.to/joshwizzy/customizing-django-authentication-using-abstractbaseuser-llg)を参考にしています。

## アカウントアプリの追加

独自ユーザーモデルを実装するアカウントアプリを次の通り追加します。

```bash
python manage.py startapp accounts
```

`AccountsConfig`を次の通り変更します。

```python
# ./accounts/apps.py
  from django.apps import AppConfig


  class AccountsConfig(AppConfig):
      default_auto_field = "django.db.models.BigAutoField"
      name = "accounts"
+     verbose_name = "アカウントアプリ"
```

プロジェクト設定ファイルの`INSTALLED_APPS`を次の通り変更します。

```python
# ./book_management/settings.py
  INSTALLED_APPS = [
      "django.contrib.admin",
      "django.contrib.auth",
      "django.contrib.contenttypes",
      "django.contrib.sessions",
      "django.contrib.messages",
      "django.contrib.staticfiles",
      "django_bootstrap5",
      "debug_toolbar",
+     "accounts.apps.AccountsConfig",
      "divisions.apps.DivisionsConfig",
      "books.apps.BooksConfig",
  ]
```

次の通り変更をリポジトリにコミットします。

```bash
git add accounts/
git add book_management/settings.py
git commit -m 'アカウントアプリを追加'
```

> 14fd533 (tag: 055-add-accounts-app)

## 独自ユーザーモデルの実装

独自ユーザーモデル（`User`）を次の通り実装します。
なお、以後、本Webアプリケーションで定義した独自ユーザーモデルを単にユーザーモデルと呼びます。

<!-- cspell: disable -->
```python
# ./accounts/models.py
from django.contrib.auth.models import (
    AbstractBaseUser,
    BaseUserManager,
    PermissionsMixin,
)
from django.db import models
from django.utils import timezone


class UserManager(BaseUserManager):
    use_in_migrations = True

    def _create_user(self, email, password, **extra_fields):
        if not email:
            raise ValueError("Eメールを設定してください。")
        email = self.normalize_email(email)
        user = self.model(email=email, **extra_fields)
        user.set_password(password)
        user.save(using=self._db)
        return user

    def create_user(self, email, password=None, **extra_fields):
        extra_fields.setdefault("is_staff", False)
        extra_fields.setdefault("is_superuser", False)
        return self._create_user(email, password, **extra_fields)

    def create_superuser(self, email, password, **extra_fields):
        extra_fields.setdefault("is_staff", True)
        extra_fields.setdefault("is_superuser", True)
        if extra_fields.get("is_staff") is not True:
            raise ValueError("スーパーユーザーは`is_staff`を`True`に設定してください。")
        if extra_fields.get("is_superuser") is not True:
            raise ValueError("スーパーユーザーは`is_superuser`を`True`に設定してください。")
        return self._create_user(email, password, **extra_fields)


class User(AbstractBaseUser, PermissionsMixin):
    """ユーザーモデル

    下記フィールドは、AbstractBaseUserで実装されている。
    - password
    - last_login
    下記フィールドはPermissionMixinで実装されている。
    - is_superuser
    - groups
    - user_permissions
    """

    email = models.EmailField("Eメールアドレス", primary_key=True, max_length=255)
    name = models.CharField("名前", max_length=80)
    is_active = models.BooleanField("アクティブ", default=True)
    is_staff = models.BooleanField("スタッフ", default=False)
    date_joined = models.DateTimeField("登録日時", default=timezone.now)

    objects = UserManager()

    USERNAME_FIELD = EMAIL_FIELD = "email"
    REQUIRED_FIELDS = ("name",)

    class Meta:
        db_table = "users"
        verbose_name = verbose_name_plural = "ユーザー"
        ordering = ("email",)

    def __str__(self) -> str:
        """名前を返却する。

        Returns:
            名前
        """
        return self.name

    @property
    def username(self) -> str:
        """ユーザー名(メールアドレス)を返却する。

        Returns:
            ユーザー名(メールアドレス)。
        """
        return self.email
```
<!-- cspell: enable -->

ユーザーモデルを認証で使用するユーザーとするために、プロジェクト設定ファイルに次を追加します。

```python
# ./book_management/settings.py

# 認証で使用するユーザーモデル
AUTH_USER_MODEL = "accounts.User"
```

現在のデータベースにユーザーモデルをマイグレーションしても、最初に実行したマイグレーションと一貫性がなくなるため、マイグレーションに失敗します。
よって、次の通りデータベースを削除して、ユーザーモデルのマイグレーションファイルを作成した後、マイグレーションを再実行します。
その後、フィクスチャーを登録します。

```bash
rm db.sqlite3
python manage.py makemigrations
python manage.py migrate
python manage.py loaddata --format=yaml ./divisions/fixtures/divisions.yaml
python manage.py loaddata --format=yaml ./books/fixtures/classifications.yaml
python manage.py loaddata --format=yaml ./books/fixtures/classification_details.yaml
python manage.py loaddata --format=yaml ./books/fixtures/books.yaml
```

次の通りスーパーユーザー（システム管理者ユーザー）を作成します。

```bash
python manage.py createsuperuser
Eメールアドレス: spam@example.com
名前: 徳川　家康
Password:
Password (again):
このパスワードは短すぎます。最低 8 文字以上必要です。
このパスワードは一般的すぎます。
Bypass password validation and create user anyway? [y/N]: y
Superuser created successfully.
```

次の通り、Pythonインタラクティブシェルで、ユーザーを登録してパスワードをチェックできます。

```bash
python manage.py shell
```

<!-- cspell: disable -->
```python
from accounts.models import User
user = User()
user.id
# UUID('6b204ff6-03bb-45e3-8d71-2ba0768a1095')
user.email = "grok@example.com"
user.name = "徳川　秀忠"
user.set_password("django")
user.save()
from django.contrib.auth.hashers import check_password
check_password("django", user.password)
# True
```
<!-- cspell: enable -->

上記で登録したユーザーのパスワードが、データベースに記録されている様子を次に示します。

<!-- cspell: disable -->
```bash
sqlite3 db.sqlite3
sqlite> .headers on
sqlite> SELECT * FROM users;
password|last_login|is_superuser|email|name|is_active|is_staff|date_joined
pbkdf2_sha256$600000$VytyASTBvAkrnxDyC7oA1T$OaZlnr+dPIhjcW4ze99tBiP8/qm3f01JbmUI2q0dhB8=||1|spam@example.com|徳川　家康|1|1|2023-04-26 10:29:02.372759
pbkdf2_sha256$600000$lbz4XHb2PQHFGKYvEoxDYO$FPpgoW8hjYESNhdjxoSKzH9FQp+ognOsLF5QKwL4b2E=||0|grok@example.com|徳川　秀忠|1|0|2023-04-26 10:29:21.428121
```
<!-- cspell: enable -->

`password`カラムには、`<algorithm>$<iterations>$<salt>$<hash>`の形式で値が記録されています。
上記、2ユーザーは同じパスワードを設定ましたが、ソルトが異なっているため、ハッシュ値も異なっています。

次の通り変更をリポジトリにコミットします。

```bash
git add accounts/
git add book_management/settings.py
git commit -m 'ユーザーモデルを実装'
```

> 1bed130 (tag: 056-implement-user-model)

## ユーザーモデルを管理サイトに登録

次の通り管理サイトにユーザーモデルを登録します。

<!-- cspell: disable -->
```bash
# ./accounts/admin.py
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin as DjangoUserAdmin
from django.contrib.auth.forms import UserChangeForm as DjangoUserChangeForm
from django.contrib.auth.forms import UserCreationForm as DjangoUserCreationForm
from django.utils.translation import gettext_lazy as _

from .models import User


class UserCreationForm(DjangoUserCreationForm):
    """ユーザー作成フォーム"""


class UserChangeForm(DjangoUserChangeForm):
    """ユーザー作成フォーム"""


class UserAdmin(DjangoUserAdmin):
    """ユーザーアドミン"""

    fieldsets = (
        (None, {"fields": ("email", "password", "name")}),
        (
            _("Permissions"),
            {
                "fields": (
                    "is_active",
                    "is_staff",
                    "is_superuser",
                    "groups",
                    "user_permissions",
                )
            },
        ),
        (_("Important dates"), {"fields": ("last_login", "date_joined")}),
    )
    add_fieldsets = (
        (
            None,
            {
                "classes": ("wide",),
                "fields": ("email", "password1", "password2"),
            },
        ),
    )
    add_form = UserCreationForm
    form = UserChangeForm
    list_display = (
        "email",
        "name",
        "is_active",
        "date_joined",
    )
    list_filter = (
        "is_active",
        "is_staff",
        "is_superuser",
        "groups",
    )
    search_fields = ("email", "name")
    ordering = ("email",)


admin.site.register(User, UserAdmin)
```
<!-- cspell: enable -->

管理サイトにアクセスして、ユーザーを登録または変更してください。
なお、ユーザーを登録するとき、Eメールアドレスとパスワードを入力して`保存`ボタンをクリックした後、次に表示されるページにユーザーの名前を入力して`保存`ボタンをクリックします。
管理サイトでユーザーを登録及び変更できた場合は、次の通り変更をリポジトリにコミットします。

```bash
git add ./accounts/admin.py
git commit -m 'ユーザーモデルを管理サイトに登録'
```

> d24c7da (tag: 057-register-user-model-to-admin-site)

## ログインページの実装

ログインビューを実装するため、`./accounts/views.py`の内容を次で置き換えます。

```python
# ./accounts/views.py
from django.contrib.auth.views import LoginView as DjangoLoginView

from core.mixins import PageTitleMixin


class LoginView(PageTitleMixin, DjangoLoginView):
    """ログインビュー"""

    title = "ログイン"
    template_name = "accounts/login.html"
```

`./accounts/templates/accounts`ディレクトリを作成後、ログインテンプレートを次の通り実装します。

```bash
mkdir -p ./accounts/templates/accounts
code ./accounts/templates/accounts/login.html
```

```html
<!-- ./accounts/templates/accounts/login.html -->
{% extends 'bootstrap_base.html' %}
{% load django_bootstrap5 %}

{% block bootstrap5_content %}
  <div class="container-fluid vh-100">
    <div class="" style="padding-top:200px">
      <div class="rounded d-flex justify-content-center">
        <div class="col-md-5 col-sm-12 shadow-lg p-5 bg-light">
          <div class="text-center pb-3">
            <h3 class="text-primary">書籍管理アプリ</h3>
          </div>
          <form method="post">
            {% csrf_token %}
            {% bootstrap_form_errors form %}
            {% bootstrap_field form.username size='sm' show_label=False placeholder='Eメール' %}
            {% bootstrap_field form.password size='sm' show_label=False placeholder='パスワード' %}
            <div class="d-grid gap-2">
              {% bootstrap_button title button_type='submit' button_class='btn-primary' extra_classes='btn-block' size='sm' %}
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
{% endblock bootstrap5_content %}
```

ログインビューをディスパッチするため、`./accounts/urls.py`を作成して次を入力します。

```bash
code ./accounts/urls.py
```

```python
# ./accounts/urls.py
from django.urls import path

from . import views

app_name = "accounts"

urlpatterns = [
    # ログインページ
    path("login/", views.LoginView.as_view(), name="login"),
]
```

ルートURLconfにアカウントアプリのURLconfを次の通り追加します。

```python
# ./book_management/urls.py
  from django.urls import include, path

  urlpatterns = [
+     path("accounts/", include("accounts.urls")),
      path("divisions/", include("divisions.urls")),
      path("books/", include("books.urls")),
      path("admin/", admin.site.urls),
```

Djangoにログインページとログインに成功した後にリダイレクトするURLを伝えるために、プロジェクト設定ファイルに次を追加します。

```python
# ./book_management/settings.py
# ログイン
LOGIN_URL = "/accounts/login/"
LOGIN_REDIRECT_URL = "/books/"
```

開発サーバーを起動してブラウザで`http://localhost:8000/accounts/login/`にアクセスします。
前に登録したユーザーのEメールアドレスとパスワードを入力して、書籍一覧ページにリダイレクトすることを確認できたら、次の通り変更をリポジトリにコミットします。

```bash
git add ./accounts/
git add ./book_management/
git commit -m 'ログインページを実装'
```

## ユーザーの表示とログアウト機能の実装

それぞれの書籍アプリページのページヘッダに、ユーザーの名前を表示して、ユーザーをログインさせるボタンを追加します。
なお、書籍一覧及び書籍詳細ページには、認証済み、または認証されていないユーザーがアクセスすることを許可します。
ただし、書籍登録、書籍更新及び書籍削除ページには、認証済みユーザーのみアクセスすることを許可します。

Djangoにおいて、認証されていないユーザーは`django.contrib.auth.models.AnonymousUser`で表現され、認証済みユーザーは上記で実装した`User`で表現されます。

次の通りログアウトビューを実装します。

```python
- from django.contrib.auth.views import LoginView as DjangoLoginView
+ from django.contrib.auth.mixins import LoginRequiredMixin
+ from django.contrib.auth.views import (
+     LoginView as DjangoLoginView,
+     LogoutView as DjangoLogoutView,
+ )

  from core.mixins import PageTitleMixin

  (...省略...)
+ class LogoutView(PageTitleMixin, DjangoLogoutView):
+     """ログアウトビュー"""
+
+     title = "ログアウト"
+     template_name = "accounts/logout.html"
```

次の通りログアウトテンプレートを実装します。

```html
<!-- ./accounts/templates/accounts/logout.html -->
{% extends 'bootstrap_base.html' %}

{% block bootstrap5_content %}
  <div class="container-fluid vh-100">
    <div class="" style="padding-top:200px">
      <div class="rounded d-flex justify-content-center">
        <div class="col-md-5 col-sm-12 shadow-lg p-5 bg-light">
          <div class="text-center pb-3">
            <h3 class="mb-5 text-primary">書籍管理アプリ</h3>
            <p>書籍管理アプリからログアウトしました。</p>
            <a href="{% url 'accounts:login' %}">ログインページに移動</a>
          </div>
        </div>
      </div>
    </div>
  </div>
{% endblock bootstrap5_content %}
```

次の通りログアウトビューをディスパッチします。

```python
# ./accounts/urls.py
  urlpatterns = [
      # ログインページ
      path("login/", views.LoginView.as_view(), name="login"),
+     # ログアウトページ
+     path("logout/", views.LogoutView.as_view(), name="logout"),
  ]
```

それぞれの書籍ページで表示するページヘッダテンプレートを次の通り実装します。
ページヘッダテンプレートは、認証済みユーザーがアクセスしたときにユーザーの名前とログインページへのリンクを表示して、認証していないユーザーがアクセスしたときにログインページへのリンクを表示します。

```html
<!-- ./books/templates/books/_book_page_header.html -->
<div class="container-fluid">
  <div class="row">
    <div class="col">
      <h2 class="float-start">{{ title }}</h2>
      <div class="float-end" style="font-size: 0.85rem;">
        {% if user.is_anonymous %}
          <div style="padding-top: 1.25rem;">
            <a href="{% url 'accounts:login' %}">ログイン</a>
          </div>
        {% else %}
          <p class="mb-0">こんにちは、<b>{{ user.name }}</b> さん。</p>
          <a class="mb-0 float-end" href="{% url 'accounts:logout' %}">ログアウト</a>
        {% endif %}
      </div>
    </div>
  </div>
</div>
```

それぞれの書籍テンプレートが派生する書籍ベーステンプレートを次の通り実装します。

```html
<!-- ./books/templates/books/book_base.html -->
{% extends 'bootstrap_base.html' %}

{% block bootstrap5_before_content %}
  {% include "books/_book_page_header.html" %}
{% endblock bootstrap5_before_content %}
```

それぞれの書籍テンプレートが書籍ベーステンプレートから派生するように修正します。

```html
<!-- ./books/templates/books/book_list.html -->
- {% extends 'bootstrap_base.html' %}
-
- {% block bootstrap5_before_content %}
-   <div class="container-fluid">
-     <h2>書籍一覧</h2>
-   </div>
- {% endblock bootstrap5_before_content %}
+ {% extends 'books/book_base.html' %}

  {% block bootstrap5_content %}
    <div class="container-fluid">
```

```html
<!-- ./books/templates/books/book_detail.html -->
- {% extends 'bootstrap_base.html' %}
-
- {% block bootstrap5_before_content %}
-   <div class="container-fluid">
-     <h2>書籍詳細</h2>
-   </div>
- {% endblock bootstrap5_before_content %}
+ {% extends 'books/book_base.html' %}

  {% block bootstrap5_content %}
    <div class="container-fluid">
```

```html
<!-- ./books/templates/books/book_form.html -->
- {% extends 'bootstrap_base.html' %}
+ {% extends 'books/book_base.html' %}

  {% load django_bootstrap5 %}

  (...省略...)

- {% block bootstrap5_before_content %}
-   <div class="container-fluid">
-     <h2>書籍{{ action }}</h2>
-   </div>
- {% endblock bootstrap5_before_content %}
-
{% block bootstrap5_content %}
  <div class="container-fluid">
    <form method="post">
```

```html
<!-- ./books/templates/books/book_confirm_delete.html -->
- {% extends 'bootstrap_base.html' %}
+ {% extends 'books/book_base.html' %}
  {% load django_bootstrap5 %}

  (...省略...)

- {% block bootstrap5_before_content %}
-   <div class="container-fluid">
-     <h2>書籍削除</h2>
-   </div>
- {% endblock bootstrap5_before_content %}
-
  {% block bootstrap5_content %}
    <div class="container-fluid">
      <div>この書籍を削除しますか?</div>
```

本Webアプリケーションからログアウトできて、またそれぞれの書籍ページが正常に機能することを確認できたら、次の通り変更をコミットします。

```bash
git add ./accounts/
git add ./books/
git commit -m 'ユーザーを表示して、ログアウトできるように実装'
```

## アクセス制限の実装

次の通り書籍登録、更新及び削除ページに認証済みユーザーしかアクセスできないように制限します。
認証済みユーザーしがアクセスを許可していないページに、認証していないユーザーがアクセスしたとき、その旨を示すメッセージをDjangoが提供するセッションフレームワークでログインページに渡すようにします。

### カスタムLoginRequiredMixinの実装

Djangoが提供する`django.contrib.auth.mixins.LoginRequiredMixin`をビューが継承すると、そのビューは認証済みユーザーのアクセスを許可して、認証していないユーザーに対してはログインページ（`LOGIN_URL`に設定したページ）を表示します。
Djangoが提供する`LoginRequiredMixin`はユーザーに通知するメッセージを提供する機能がないため、Djangoのセッションフレームワークでメッセージを提供する独自の`LoginRequiredMixin`を`./core/mixins.py`に次の通り実装します。

```python
# ./core/mixins.py

class LoginRequiredMixin(DjangoLoginRequiredMixin):
    """ログイン要求ミックスイン

    Djangoが提供するLoginRequiredMixinを拡張して、未認証でページにアクセスする権限がないことを
    示すメッセージを、Djangoが適用するセッションフレームワークで通知できるようにします。
    このミックスインを継承するビューは、permission_denied_messageをオーバーライドすることで、
    ビューにあわせたメッセージを通知できます。
    """

    permission_denied_message = "ページにアクセスする権限がありません。"

    def dispatch(self, request: HttpRequest, *args: Any, **kwargs: Any) -> HttpResponse:
        if not request.user.is_authenticated:
            messages.add_message(
                request, messages.WARNING, self.permission_denied_message
            )
            return self.handle_no_permission()
        return super().dispatch(request, *args, **kwargs)
```

### 書籍登録、更新及び削除ページのアクセス制限

書籍登録、更新及び削除ビューを先ほど実装した`LoginRequiredMixin`を継承するように変更します。

```python
# ./books/views.py
  from django.http import HttpRequest, HttpResponse
  from django.urls import reverse, reverse_lazy
  from django.views import generic

- from core.mixins import FormActionMixin, PageTitleMixin
+ from core.mixins import FormActionMixin, PageTitleMixin, LoginRequiredMixin

  from .forms import BookForm
  from .models import Book, Classification, ClassificationDetail

  (...省略...)

  class BookCreateView(
      BookViewMixin,
      PageTitleMixin,
      FormActionMixin,
+     LoginRequiredMixin,
      generic.CreateView,
  ):
      """書籍登録ビュー"""

  (...省略...)

  class BookUpdateView(
      BookViewMixin,
      PageTitleMixin,
      FormActionMixin,
+     LoginRequiredMixin,
      generic.UpdateView,
  ):
      """書籍更新ビュー"""

  (...省略...)

  class BookDeleteView(
      BookViewMixin,
      PageTitleMixin,
+     LoginRequiredMixin,
      generic.DeleteView,
  ):
      """書籍削除ビュー"""
```

開発サーバーを起動して、ブラウザで、ユーザーがログアウトした状態で書籍登録、更新及び削除ページにアクセスしてください。
正しく実装していれば、当該ページは表示されず、アクセスする権限がないことを示すメッセージと共に、ログインページが表示されるはずです。

次の通り変更をリポジトリにコミットしてください。

```bash
git add ./accounts/
git add ./core/
git add ./books/
git commit -m '書籍登録、更新及び削除ページのアクセス制限を実装'
```

> commit 3776f0e37be8359202073529a6f5fbef5f82c100

### 書籍一覧及び書籍詳細ページのリンクの整理

書籍登録、更新及び削除ページへのリンクを認証済みユーザーに対してのみ表示するように、書籍一覧及び書籍詳細テンプレートを変更します。

```html
<!-- ./books/templates/books/book_list.html -->
                <a href="{% url 'books:book-detail' pk=book.id %}">詳細</a>
+               {% if user.is_authenticated %}
                  <a href="{% url 'books:book-update' pk=book.id %}">更新</a>
                  <a href="{% url 'books:book-delete' pk=book.id %}">削除</a>
+               {% endif %}
              </td>
            </tr>
          {% endfor %}

  (...省略...)

      {% else %}
        <div class="ms-3">書籍が存在しません。</div>
      {% endif %}
+     {% if user.is_authenticated %}
        <div>
          <a class="ms-3" href="{% url 'books:book-create' %}">書籍登録</a>
        </div>
+     {% endif %}
    </div>
```

```html
<!-- ./books/templates/books/book_detail.html -->
    <div class="container-fluid">
      {% include "books/_book_detail.html" %}
      <div>
        <a href="{% url 'books:book-list' %}">書籍一覧</a>
+       {% if user.is_authenticated %}
          <a href="{% url 'books:book-create' %}">書籍登録</a>
          <a href="{% url 'books:book-update' pk=book.id %}">書籍更新</a>
          <a href="{% url 'books:book-delete' pk=book.id %}">書籍削除</a>
+       {% endif %}
      </div>
    </div>
  {% endblock bootstrap5_content %}
```

認証されていないユーザーが、書籍一覧及び書籍詳細ページにアクセスしたとき、書籍登録、更新及び削除ページへのリンクが表示されず、認証済みユーザーに表示されることを確認でいたら、次の通り変更をリポジトリにコミットします。

```bash
git add ./books/
git commit -m '書籍一覧及び詳細ページのリンクを整理'
```

## まとめ

これまでの実装により、書籍を管理するために必要な最小限な機能を持ったWebアプリケーションを作成できました。

次の章では、書籍アプリが管理するデータを操作する`REST API`を実装します。
