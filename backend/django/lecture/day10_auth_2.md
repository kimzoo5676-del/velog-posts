# Django Authentication System 02

## 목차

1. [학습 목표](#1-학습-목표)
2. [사용자 생명주기 (User Lifecycle)](#2-사용자-생명주기-user-lifecycle)
3. [Logout (로그아웃)](#3-logout-로그아웃)
4. [Signup (회원가입)](#4-signup-회원가입)
5. [회원가입 에러 발생 및 해결](#5-회원가입-에러-발생-및-해결)
6. [get_user_model()](#6-get_user_model)
7. [Delete (회원탈퇴)](#7-delete-회원탈퇴)
8. [인증된 사용자에 대한 접근 제한](#8-인증된-사용자에-대한-접근-제한)
9. [login_required 데코레이터](#9-login_required-데코레이터)
10. [User 관련 개념 총정리](#10-user-관련-개념-총정리)

---

## 1. 학습 목표

| # | 키워드 | 설명 |
|---|--------|------|
| 1 | `logout` | 로그아웃 기능 구현 |
| 2 | `UserCreationForm` | 커스텀 User 모델에 맞는 회원가입 폼 작성 |
| 3 | `get_user_model` | 활성화된 User 모델을 안전하게 참조 |
| 4 | 커스텀 회원가입 폼 | 직접 만든 폼으로 회원가입 기능 구현 |
| 5 | `request.user.delete` | 회원 탈퇴 기능 구현 |
| 6 | `is_authenticated` | 로그인 여부에 따라 접근 제어 |
| 7 | `login_required` | 데코레이터로 인증된 사용자만 접근 허용 |

---

## 2. 사용자 생명주기 (User Lifecycle)

단순히 '로그인' 기능 하나만 있으면 될까?

사용자가 서비스에 들어오고 나가는 **전체 흐름**을 생각해야 한다.

| 사용자 유형 | 필요한 기능 |
|------------|------------|
| 비회원 (Anonymous User) | 회원가입 (새 계정 생성) |
| 비회원 (Anonymous User) | 로그인 (인증 후 접속) |
| 회원 (Logged-in User) | 로그아웃 (연결 종료) |
| 회원 (Logged-in User) | 회원탈퇴 (정보 삭제) |

> **이번 학습의 핵심**
> 사용자가 **"누구인지"를 확인**하고,
> 데이터의 **생성(가입) → 소멸(탈퇴)** 까지
> **사용자 전체 생명주기(Lifecycle)** 를 관리하는 것이 Authentication이다.

CRUD 관점으로 보면 이렇다.

```
회원가입  →  Create  (User 객체 생성)
로그인    →  Read    (User 인증)
정보수정  →  Update  (User 객체 수정)
회원탈퇴  →  Delete  (User 객체 삭제)
```

---

## 3. Logout (로그아웃)

### 핵심 개념

> **로그아웃 = 세션 삭제 과정**

로그인 상태는 서버의 세션 데이터와 클라이언트의 쿠키로 유지된다.
로그아웃은 이 둘을 모두 삭제하는 과정이다.

`auth_logout(request)` 이 하는 일:

| 순서 | 처리 내용 |
|------|-----------|
| 1 | 서버 DB에서 현재 요청에 대한 Session Data 삭제 |
| 2 | 클라이언트 쿠키에서 Session ID 삭제 |
| 3 | `request.user` 를 `AnonymousUser` 로 교체 |

> ✅ 세션만 삭제하면 서버/클라이언트에서 로그인 정보는 사라지지만
> 현재 요청(request) 안의 user 객체가 남아있을 수 있다.
> 그래서 `AnonymousUser` 로 덮어써서 현재 요청에서도 완전히 로그아웃 처리한다!

---

### Login vs Logout 비교

| | Login | Logout |
|--|-------|--------|
| 하는 일 | 세션 **생성** | 세션 **삭제** |
| 함수 | `auth_login(request, user)` | `auth_logout(request)` |
| 인자 수 | **2개** (request + user 객체) | **1개** (request만) |

> logout은 `request` 안에 이미 현재 로그인된 유저 정보가 들어있기 때문에
> 별도로 user를 인자로 받을 필요가 없다.

---

### 구현

### Step 1 — `accounts/urls.py` URL 등록

`/accounts/logout/` 경로로 요청이 들어오면 `views.logout` 함수를 호출하도록 URL을 연결한다.

```python
# accounts/urls.py
urlpatterns = [
    path('login/', views.login, name='login'),
    path('logout/', views.logout, name='logout'),  # 추가
]
```

> 💡 `views.logout` 을 호출하도록 했으니, 이제 `views.py` 에 `logout` 함수를 만들어야 한다!

### Step 2 — `accounts/views.py` logout 함수 작성

DB에서 현재 요청에 대한 Session Data를 삭제하고,
클라이언트의 쿠키에서도 Session ID를 삭제하는 내장 logout 함수를 활용한다.

```python
# accounts/views.py
from django.contrib.auth import logout as auth_logout

def logout(request):
    auth_logout(request)
    return redirect('articles:index')
```

> 💡 login 함수와 마찬가지로 `as auth_logout` 으로 alias를 사용한다.
> 우리가 정의한 함수 이름도 `logout`, Django 내장 함수 이름도 `logout` 이라서
> 이름 충돌 방지를 위해 별칭을 붙여주는 것이다!
> views.py 에서 로그아웃 처리를 했으니, 이제 템플릿에 버튼을 추가해야 한다!

### Step 3 — `articles/index.html` 로그아웃 버튼 추가

로그아웃은 데이터를 변경하는 작업이므로 반드시 `method="POST"` 로 처리한다.

```html
<!-- articles/index.html -->
<form action="{% url 'accounts:logout' %}" method="POST">
  {% csrf_token %}
  <input type="submit" value="Logout">
</form>
```

> ⚠️ 로그아웃은 반드시 `method="POST"` + `{% csrf_token %}` 세트!
> `<a>` 태그(GET 요청)로 로그아웃하면 악성 사이트의 이미지 태그 하나만으로도
> 사용자를 강제 로그아웃 시킬 수 있어 CSRF 공격에 취약해진다.

---

### 🔍 개발자 도구로 세션 확인하기

`Application 탭 → Cookies → http://127.0.0.1:8000`

| 쿠키 | 역할 | 로그아웃 후 |
|------|------|------------|
| `sessionid` | 로그인 상태 유지 | ❌ 삭제됨 |
| `csrftoken` | CSRF 공격 방지 | ✅ 유지됨 |

> `csrftoken` 은 로그인/로그아웃과 무관하게 브라우저-서버 간 위조 요청 방지용이라
> 세션이 삭제되어도 남아있다.

---

## 4. Signup (회원가입)

### 핵심 개념

> **회원가입 = User 객체를 Create 하는 과정**

사용자로부터 아이디, 비밀번호 등의 정보를 입력받아
DB에 새로운 User 객체를 생성하고 저장한다.

---

### 구현

### Step 1 — `accounts/urls.py` URL 등록

`/accounts/signup/` 경로로 요청이 들어오면 `views.signup` 함수를 호출하도록 URL을 연결한다.

```python
# accounts/urls.py
urlpatterns = [
    path('login/', views.login, name='login'),
    path('logout/', views.logout, name='logout'),
    path('signup/', views.signup, name='signup'),  # 추가
]
```

> 💡 `views.signup` 을 호출하도록 했으니, 이제 `views.py` 에 `signup` 함수를 만들어야 한다!

### Step 2 — `articles/index.html` 회원가입 링크 추가

회원가입 페이지로 이동하는 링크를 추가한다.

```html
<!-- articles/index.html -->
<a href="{% url 'accounts:signup' %}">회원가입</a>
```

> 💡 링크를 추가했으니, 이제 회원가입 로직을 처리할 `views.py` 를 작성해야 한다!

### Step 3 — `accounts/views.py` signup 함수 뼈대 작성

실제 코드를 채우기 전에 함수의 뼈대를 먼저 잡는다.

```python
# accounts/views.py
def signup(request):
    if request.method == 'POST':
        pass
    else:
        pass
```

> 💡 뼈대를 잡았으니, GET 요청 처리 부분(else)부터 채워나가자!

### Step 4 — `accounts/views.py` else + context + render 채우기

GET 요청일 때 빈 폼을 생성해서 템플릿에 넘겨준다.

```python
# accounts/views.py
from django.contrib.auth.forms import AuthenticationForm, UserCreationForm

def signup(request):
    if request.method == 'POST':
        pass
    else:
        form = UserCreationForm()
    context = {
        'form': form,
    }
    return render(request, 'accounts/signup.html', context)
```

> 💡 `render(request, 'accounts/signup.html', context)` 로 폼을 템플릿에 넘겨주고 있으니,
> 이제 `accounts/signup.html` 템플릿을 만들어야 한다!

### Step 5 — `accounts/templates/accounts/signup.html` 작성

views.py 에서 넘겨받은 `UserCreationForm` 을 화면에 출력한다.

```html
{% extends 'base.html' %}

{% block content %}
  <h1>회원가입</h1>
  <hr>
  <form action="{% url 'accounts:signup' %}" method="POST">
    {% csrf_token %}
    {{ form.as_p }}
    <input type="submit" value="SignUp">
  </form>
{% endblock content %}
```

> 💡 `UserCreationForm` 이 자동으로 제공하는 기본 필드:
> - `Username` : 150자 이하, 영문/숫자/@./+/-/_ 만 허용
> - `Password` : 8자 이상, 흔한 비밀번호 불가, 숫자만 불가
> - `Password confirmation` : 비밀번호 확인 (일치 여부 자동 검사)
>
> 템플릿을 만들었으니, 이제 POST 요청을 처리하는 로직을 채워야 한다!

### Step 6 — `accounts/views.py` POST 요청 처리

폼 데이터를 받아 유효성 검사 후 DB에 저장한다.

```python
# accounts/views.py
def signup(request):
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('articles:index')
    else:
        form = UserCreationForm()
    context = {
        'form': form,
    }
    return render(request, 'accounts/signup.html', context)
```

> ✅ `UserCreationForm` 은 **ModelForm** 이기 때문에
> 데이터를 넘겨주면 알아서 User 객체와 연결되고, `form.save()` 로 DB에 저장된다.
> 그런데 지금 이 코드를 실행하면 에러가 발생한다!

---

## 5. 회원가입 에러 발생 및 해결

### 에러 메시지
```
AttributeError: Manager isn't available; 
'auth.User' has been swapped for 'accounts.User'
```

### 원인

`UserCreationForm` 은 Django 기본 `auth.User` 를 바라보도록 작성된 클래스다.

```python
# Django 내부 UserCreationForm 코드
class Meta:
    model = User  # ← auth.User (기본 유저)를 바라보고 있음
    fields = ("username",)
```

우리 프로젝트는 커스텀 `accounts.User` 를 사용하고 있어서
기본 `auth.User` 는 이미 swap 되어 사용 불가 → **에러 발생!**

> 💡 `UserCreationForm` 을 그대로 쓰는 대신,
> 상속받아서 `model` 을 커스텀 User 로 교체한 `CustomUserCreationForm` 을 만들어야 한다!

---

### 해결

### Step 1 — `accounts/forms.py` 생성

`UserCreationForm` 을 상속받아 커스텀 User 모델과 연결된 폼을 새로 만든다.

```python
# accounts/forms.py
from django.contrib.auth import get_user_model
from django.contrib.auth.forms import UserCreationForm

class CustomUserCreationForm(UserCreationForm):
    class Meta(UserCreationForm.Meta):
        model = get_user_model()  # 현재 활성화된 User 모델 반환
```

> ⚠️ `get_user_model` vs `get_user_model()`
> ```python
> model = get_user_model   # ❌ 함수 자체를 넣은 것 (호출 안 함)
> model = get_user_model() # ✅ 호출해서 User 모델을 반환받은 것
> ```
> `()` 빠뜨리지 않도록 주의!
> 폼을 만들었으니, 이제 views.py 에서 이 폼을 가져다 써야 한다!

### Step 2 — `accounts/views.py` import 수정 및 폼 교체

기존 `UserCreationForm` 을 `CustomUserCreationForm` 으로 교체하고,
회원가입 즉시 자동 로그인 처리와 로그인 상태 접근 차단도 추가한다.

```python
# accounts/views.py
from .forms import CustomUserCreationForm

def signup(request):
    if request.user.is_authenticated:       # 이미 로그인된 유저는 접근 차단
        return redirect('articles:index')

    if request.method == 'POST':
        form = CustomUserCreationForm(request.POST)
        if form.is_valid():
            user = form.save()              # 저장하면서 user 객체 반환
            auth_login(request, user)       # 가입 즉시 자동 로그인!
            return redirect('articles:index')
    else:
        form = CustomUserCreationForm()
    context = {
        'form': form,
    }
    return render(request, 'accounts/signup.html', context)
```

> 💡 **가입 후 자동 로그인**
> `form.save()` 는 저장된 user 객체를 반환한다.
> 이 객체를 받아서 바로 `auth_login()` 을 호출하면 회원가입 즉시 자동 로그인 처리가 된다!
>
> 💡 **이미 로그인된 유저 접근 차단**
> `request.user.is_authenticated` 로 이미 로그인 상태라면 메인 페이지로 튕겨낸다.
> 로그인된 상태에서 회원가입 페이지에 접근하는 건 말이 안 되니까!

---

## 6. get_user_model()

> **현재 프로젝트에서 활성화된 사용자 모델을 반환하는 함수**

```python
from django.contrib.auth import get_user_model

User = get_user_model()
```

### 왜 써야 하나?

`settings.py` 의 `AUTH_USER_MODEL` 설정에 따라
기본 `auth.User` 일 수도 있고, 커스텀 `accounts.User` 일 수도 있기 때문에
**올바른 모델을 동적으로 가져오기 위해** 사용한다.

```python
# ❌ 이렇게 쓰면 안 됨
from django.contrib.auth.models import User
# User 모델이 바뀌면 이 코드도 일일이 수정해야 함

# ✅ 이렇게 써야 함
User = get_user_model()
# User 모델이 바뀌어도 코드 수정 불필요 → 재사용성 & 유연성 ↑
```

### Django 공식 문서

> "If you reference **User** directly (for example, by referring to it in a foreign key), your code will not work in projects where the `AUTH_USER_MODEL` setting has been changed to a different user model."

→ `User` 를 직접 참조하면 `AUTH_USER_MODEL` 이 변경된 프로젝트에서 코드가 동작하지 않는다.
Django는 **공식적으로** `get_user_model()` 사용을 강조하고 있다.

> 💡 User model 참조에 대한 자세한 내용은 **추후 모델 관계**에서 다룰 예정

---

## 7. Delete (회원탈퇴)

### 핵심 개념

> **회원탈퇴 = User 객체를 Delete 하는 과정**

CRUD 관점에서 User 모델에 대한 **D(Delete)** 에 해당한다.
`request.user` 는 현재 로그인된 User 모델의 **인스턴스**이기 때문에
바로 `.delete()` 를 호출할 수 있다.

```python
# 게시글 삭제 — pk로 직접 찾아와야 함
article = Article.objects.get(pk=pk)
article.delete()

# 회원탈퇴 — request.user에 이미 인스턴스가 있으니 바로 delete!
request.user.delete()
```

### `request.user` 란?

| 상태 | `request.user` 값 |
|------|------------------|
| 로그인 상태 | 현재 로그인된 User 객체 (인스턴스) |
| 비로그인 상태 | `AnonymousUser` 객체 |

> `auth_login()` 호출 시 세션에 유저 정보를 저장하고
> 이후 모든 요청마다 Django가 세션을 읽어서 `request.user` 에 자동으로 넣어준다.

---

### 구현

### Step 1 — `accounts/urls.py` URL 등록

`/accounts/delete/` 경로로 요청이 들어오면 `views.delete` 함수를 호출하도록 URL을 연결한다.

```python
# accounts/urls.py
urlpatterns = [
    path('login/', views.login, name='login'),
    path('logout/', views.logout, name='logout'),
    path('signup/', views.signup, name='signup'),
    path('delete/', views.delete, name='delete'),  # 추가
]
```

> 💡 `views.delete` 를 호출하도록 했으니, 이제 템플릿에 탈퇴 버튼을 추가해야 한다!

### Step 2 — `articles/index.html` 회원탈퇴 버튼 추가

회원탈퇴는 DB에서 데이터를 삭제하는 작업이므로 반드시 `method="POST"` 로 처리한다.

```html
<!-- articles/index.html -->
<form action="{% url 'accounts:delete' %}" method="POST">
  {% csrf_token %}
  <input type="submit" value="회원탈퇴">
</form>
```

> 💡 버튼을 추가했으니, 이제 실제 삭제 로직을 처리할 `views.py` 를 작성해야 한다!

### Step 3 — `accounts/views.py` delete 함수 작성

```python
# accounts/views.py
@login_required
def delete(request):
    request.user.delete()    # 1. 먼저 유저 삭제
    auth_logout(request)     # 2. 그 다음 로그아웃
    return redirect('articles:index')
```

> ⚠️ **순서가 바뀌면 안 되는 이유!**
> ```python
> # ❌ 순서 바꾸면 안 됨
> auth_logout(request)     # 먼저 로그아웃 → request.user가 AnonymousUser로 교체됨
> request.user.delete()    # AnonymousUser를 삭제하려고 함 → 실제 유저 삭제 안 됨!
> ```
> `auth_logout()` 을 먼저 호출하면 `request.user` 가 `AnonymousUser` 로 바뀌어버려서
> `.delete()` 가 실제 유저가 아닌 `AnonymousUser` 에 호출된다.
> 반드시 **삭제 먼저 → 로그아웃 나중** 순서를 지켜야 한다!

---

## 8. 인증된 사용자에 대한 접근 제한

접근 제한 방법은 크게 두 가지다.

1. `is_authenticated` 속성
2. `login_required` 데코레이터

---

### `is_authenticated` 속성

> **사용자가 인증되었는지 여부를 알 수 있는 User 모델의 읽기 전용 속성**

| 상태 | 값 |
|------|-----|
| 인증된 사용자 (로그인) | 항상 `True` |
| 비인증 사용자 (비로그인 / AnonymousUser) | 항상 `False` |

### AnonymousUser 주요 속성

| 속성 | 값 |
|------|-----|
| `id` | `None` |
| `username` | 빈 문자열 `""` |
| `is_anonymous` | `True` |
| `is_authenticated` | `False` |
| `is_staff` / `is_superuser` | `False` |
| `set_password()`, `delete()` 등 | `NotImplementedError` 발생 |

> `is_authenticated` 랑 `is_anonymous` 는 반대라고 보면 된다.

---

### ⚠️ 메서드가 아닌 속성(Property)임을 주의!

```python
# ❌ 메서드로 착각해서 호출하면 안 됨
request.user.is_authenticated()

# ✅ 속성이라서 괄호 없이 사용
request.user.is_authenticated
```

Django 내부적으로 `@property` 데코레이터로 이렇게 정의되어 있다.

```python
@property
def is_authenticated(self):
    return True
```

> `@property` 가 붙으면 함수인데 **괄호 없이** 접근 가능하고, **읽기 전용**으로 동작한다.
> 즉, 함수로 정의됐지만 속성처럼 동작하는 것!

| 구분 | 예시 | 괄호 |
|------|------|------|
| 일반 메서드 | `form.is_valid()` | ✅ 괄호 필요 |
| property | `request.user.is_authenticated` | ❌ 괄호 없이 사용 |

---

### 구현

### Step 1 — `articles/index.html` 로그인 상태에 따라 메뉴 분기

`is_authenticated` 로 로그인/비로그인 상태에 따라 다른 메뉴를 보여준다.

```html
<!-- articles/index.html -->
{% if request.user.is_authenticated %}
  <p>Hello, {{ user.username }}</p>
  <a href="{% url 'articles:create' %}">NEW</a>
  <form action="{% url 'accounts:logout' %}" method="POST">
    {% csrf_token %}
    <input type="submit" value="Logout">
  </form>
  <form action="{% url 'accounts:delete' %}" method="POST">
    {% csrf_token %}
    <input type="submit" value="회원탈퇴">
  </form>
  <a href="{% url 'accounts:update' %}">회원정보 수정</a>
{% else %}
  <a href="{% url 'accounts:login' %}">Login</a>
  <a href="{% url 'accounts:signup' %}">Signup</a>
{% endif %}
```

| 상태 | 보이는 메뉴 |
|------|------------|
| 로그인 | 유저이름 / NEW / Logout / 회원탈퇴 / 회원정보 수정 |
| 비로그인 | Login / Signup |

> 💡 이제 로그인/비로그인 상태에 따라 보이는 메뉴가 달라진다.
> 다음으로 상세 페이지에서도 동일하게 접근 제어를 적용해보자!

### Step 2 — `articles/detail.html` 로그인 사용자에게만 수정/삭제 버튼 표시

```html
<!-- articles/detail.html -->
<hr>
{% if request.user.is_authenticated %}
  <a href="{% url 'articles:update' article.pk %}">EDIT</a>
  <br>
  <form action="{% url 'articles:delete' article.pk %}" method="POST">
    {% csrf_token %}
    <input type="submit" value="DELETE">
  </form>
{% endif %}
<a href="{% url 'articles:index' %}">[INDEX]</a>
```

> ✅ `{% else %}` 없이 `{% if %} ~ {% endif %}` 만 사용하면
> 비로그인 상태에서는 버튼 자체가 보이지 않는다.
> 하지만 버튼이 안 보인다고 해서 URL 직접 접근을 막을 수는 없다.
> 그래서 view 함수 자체에 접근 제한을 걸어야 하는데, 이때 사용하는 게 `login_required` 다!

---

## 9. login_required 데코레이터

> **로그인한 사용자만 해당 view에 접근할 수 있도록 제한하는 데코레이터**

```python
from django.contrib.auth.decorators import login_required

@login_required
def create(request):
    ...
```

### 비로그인 상태에서 접근하면?

자동으로 **로그인 페이지로 리다이렉트** 시켜준다.

```
/accounts/login/?next=/articles/create/
```

> `?next=` 파라미터에 **원래 가려던 URL** 을 담아서 보내준다.
> 로그인 완료 후 원래 페이지로 자동으로 돌아올 수 있다!

---

### `is_authenticated` vs `login_required` 비교

| | `is_authenticated` | `login_required` |
|--|-------------------|-----------------|
| 사용 위치 | 템플릿, views.py 내부 | views.py 함수 위 (데코레이터) |
| 용도 | 로그인 여부에 따라 **다른 화면** 보여줄 때 | 비로그인 사용자 **접근 자체를 차단**할 때 |
| 비로그인 처리 | 직접 if/else 로 처리 | 자동으로 로그인 페이지로 리다이렉트 |

---

### 구현

### Step 1 — `accounts/views.py` delete 함수에 적용

```python
# accounts/views.py
from django.contrib.auth.decorators import login_required

@login_required
def delete(request):
    request.user.delete()
    auth_logout(request)
    return redirect('articles:index')
```

> 💡 `@login_required` 를 붙이면 비로그인 상태에서 `/accounts/delete/` 에 접근할 경우
> 자동으로 로그인 페이지로 리다이렉트된다.
> 회원탈퇴뿐 아니라 로그인이 필요한 모든 view 함수에 적용할 수 있다!

### Step 2 — `articles/views.py` 게시글 관련 함수에 적용

```python
# articles/views.py
from django.contrib.auth.decorators import login_required

@login_required
def create(request):      # 로그인해야만 글 작성 가능
    ...

@login_required
def delete(request, pk):  # 로그인해야만 글 삭제 가능
    ...
```

> ✅ 템플릿에서 버튼을 숨기는 것(`is_authenticated`)과
> view 함수 접근 자체를 차단하는 것(`login_required`) 을 함께 사용하는 것이 올바른 방법이다.
> 버튼이 없어도 URL 직접 입력으로 접근할 수 있기 때문!

---

## 10. User 관련 개념 총정리

| | 종류 | 뭐냐 | 언제 씀 |
|--|------|------|---------|
| `request.user` | 인스턴스 | 현재 로그인된 유저 객체 | 로그인한 사람 정보 접근할 때 |
| `get_user_model()` | 클래스 | 활성화된 User 모델 반환 | forms.py, models.py에서 모델 참조할 때 |
| `User` (직접 import) | 클래스 | 기본 auth.User 모델 | ❌ 쓰지 말 것 |
| `AbstractUser` | 클래스 | 커스텀 User 모델 만들 때 상속 | models.py에서 커스텀 모델 정의할 때 |
| `AnonymousUser` | 인스턴스 | 비로그인 유저 객체 | 로그아웃 후 request.user에 자동 할당 |

---

### 실제 코드에서 등장하는 위치

```python
# models.py — AbstractUser 상속해서 커스텀 모델 만들 때
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    pass
```

```python
# forms.py — 커스텀 모델 참조할 때
from django.contrib.auth import get_user_model

class CustomUserCreationForm(UserCreationForm):
    class Meta:
        model = get_user_model()  # 설계도(클래스) 참조
```

```python
# views.py — 로그인된 유저 정보 쓸 때
request.user.username          # 현재 유저 이름
request.user.delete()          # 현재 유저 삭제
request.user.is_authenticated  # 로그인 여부
```

```python
# 로그아웃 후
request.user  # → AnonymousUser (자동으로 교체됨)
```

---

### 붕어빵으로 비유하면

```
AbstractUser     → 붕어빵 틀 설계도 (내가 커스텀해서 만드는 것)
get_user_model() → 완성된 붕어빵 틀 (설계도로 만들어진 클래스)
request.user     → 실제 붕어빵 (틀로 찍어낸 실제 유저 객체)
AnonymousUser    → 빈 붕어빵 (로그인 안 된 상태)
```

---

## 최종 정리

```
사용자 생명주기
  → 가입(Create) / 로그인(Read) / 정보수정(Update) / 탈퇴(Delete)

Logout
  → auth_logout(request) 한 줄로 세션 삭제 + 쿠키 삭제 + AnonymousUser 교체
  → 반드시 POST 요청으로만 처리 (CSRF 공격 방지)

Signup
  → UserCreationForm 을 그대로 쓰면 커스텀 User 모델과 충돌 → 에러 발생
  → CustomUserCreationForm 만들어서 get_user_model() 로 커스텀 User 연결
  → form.save() 반환값으로 user 받아서 auth_login() 호출 → 가입 즉시 자동 로그인

get_user_model()
  → AUTH_USER_MODEL 에 설정된 현재 활성화된 User 모델을 동적으로 반환
  → User 모델을 직접 import 하지 말고 항상 이걸 써야 함

Delete
  → request.user 는 이미 User 인스턴스 → 바로 .delete() 호출 가능
  → 순서 중요! request.user.delete() 먼저 → auth_logout() 나중

is_authenticated
  → @property 로 정의된 읽기 전용 속성 → 괄호 없이 사용
  → 템플릿/뷰에서 로그인 상태에 따라 다른 화면 분기 처리

login_required
  → 데코레이터로 비로그인 사용자의 접근 자체를 차단
  → 비로그인 접근 시 /accounts/login/?next=원래URL 로 자동 리다이렉트
  → is_authenticated(화면 분기) + login_required(접근 차단) 함께 써야 완전한 보호!
```