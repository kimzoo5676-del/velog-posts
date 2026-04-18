# Django로 게시판 만들기 — 빈 폴더에서 CRUD 완성까지

> Django 1~6장을 배우며 정리한 내용입니다.
> "이 코드가 왜 필요한가?"에 초점을 맞춰, 빈 폴더에서 시작해 게시판 CRUD가 완성되는 전 과정을 담았습니다.

---

## 목차

1. [Django가 뭔가요?](#1-django가-뭔가요)
2. [프로젝트 세팅 — 빈 폴더에서 서버 켜기까지](#2-프로젝트-세팅--빈-폴더에서-서버-켜기까지)
3. [MTV 패턴 — 첫 페이지 만들기](#3-mtv-패턴--첫-페이지-만들기)
4. [Template — 동적인 HTML 만들기](#4-template--동적인-html-만들기)
5. [Model — 데이터베이스 연결하기](#5-model--데이터베이스-연결하기)
6. [ORM — 파이썬으로 DB 다루기](#6-orm--파이썬으로-db-다루기)
7. [CRUD View 구현하기](#7-crud-view-구현하기)
8. [ModelForm — 유효성 검사까지 한 번에](#8-modelform--유효성-검사까지-한-번에)
9. [최종 URL 구조 정리](#9-최종-url-구조-정리)

---

## 1. Django가 뭔가요?

Django는 Python 기반의 웹 프레임워크입니다.

**프레임워크**란 이미 뼈대가 세워진 골조입니다. 라이브러리가 "내가 필요한 도구를 골라서 쓰는 것"이라면, 프레임워크는 "정해진 구조 안에서 내가 살을 채워넣는 것"입니다. 이 차이를 **제어의 역전(IoC)** 이라고 부릅니다.

Django가 기본으로 제공하는 것들:

- HTTP 요청/응답 처리
- URL 라우팅
- DB 연결 및 쿼리 (ORM)
- 인증 및 보안 (CSRF, XSS 방어)
- 관리자 페이지

> Django는 **"배터리 포함(Batteries Included)"** 철학을 가지고 있습니다. 웹 서비스에 필요한 것들이 기본으로 내장되어 있습니다.

---

## 2. 프로젝트 세팅 — 빈 폴더에서 서버 켜기까지

### 가상환경 (Virtual Environment)

```
프로젝트 A → Django 3.2 필요
프로젝트 B → Django 4.2 필요
→ 가상환경 없으면 버전 충돌!
```

가상환경은 프로젝트마다 독립된 패키지 공간을 만들어줍니다.

```bash
# 생성
python -m venv venv

# 활성화 (Windows)
source venv/Scripts/activate

# 활성화 (Mac/Linux)
source venv/bin/activate

# 비활성화
deactivate
```

> `venv/` 폴더는 용량이 크고 OS마다 달라서 `.gitignore`에 반드시 추가합니다.
> 대신 `requirements.txt`를 공유합니다.

### 의존성 관리

```bash
pip install django
pip freeze > requirements.txt     # 패키지 목록 저장
pip install -r requirements.txt   # 목록에서 한 번에 설치
```

### 프로젝트 생성 전체 흐름

```bash
# 1. 가상환경 생성 및 활성화
python -m venv venv
source venv/Scripts/activate

# 2. Django 설치
pip install django
pip freeze > requirements.txt

# 3. 프로젝트 생성 (끝에 . 필수! 없으면 폴더 중첩 발생)
django-admin startproject config .

# 4. 앱 생성
python manage.py startapp articles

# 5. settings.py에 앱 등록
# INSTALLED_APPS = [..., 'articles']

# 6. 서버 실행
python manage.py runserver
```

### 프로젝트 구조

```
프로젝트 루트/
├── manage.py            ← Django 명령어 도구
├── config/
│   ├── settings.py      ← 프로젝트 전체 설정
│   ├── urls.py          ← URL 연결 관리
│   ├── wsgi.py          ← 배포용 (동기)
│   └── asgi.py          ← 배포용 (비동기)
└── articles/
    ├── admin.py         ← 관리자 페이지 설정
    ├── models.py        ← 데이터 모델 정의
    ├── views.py         ← 요청 처리 함수
    └── migrations/      ← DB 변경사항 기록
```

**프로젝트 vs 앱:**
- **프로젝트** = 전체 설정을 담당하는 큰 틀
- **앱** = 독립적인 기능 단위 (게시판, 유저, 댓글 등)

---

## 3. MTV 패턴 — 첫 페이지 만들기

Django는 **MTV(Model-Template-View)** 패턴을 사용합니다.

| 역할 | MVC | MTV (Django) |
|------|-----|--------------|
| 데이터/비즈니스 로직 | Model | Model |
| 사용자에게 보이는 화면 | View | Template |
| 요청 처리/교통정리 | Controller | View |

### 요청 처리 흐름

```
사용자 요청
    ↓
urls.py  →  "이 URL은 어디로?"
    ↓
views.py →  "데이터 가져오고, 템플릿에 전달"
    ↓
templates →  HTML로 렌더링해서 응답
```

### 새 페이지를 만들 때의 3단계 패턴

Django에서 새 페이지를 만들 때는 항상 이 3단계를 반복합니다.

**Step 1. `urls.py` — 경로 등록**

```python
# config/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('articles/', include('articles.urls')),  # 앱 urls로 위임
]
```

```python
# articles/urls.py (직접 생성해야 합니다!)
from django.urls import path
from . import views

app_name = 'articles'

urlpatterns = [
    path('', views.index, name='index'),
]
```

**Step 2. `views.py` — 함수 작성**

```python
from django.shortcuts import render

def index(request):  # 첫 번째 인자는 반드시 request
    return render(request, 'articles/index.html')
```

**Step 3. `templates/articles/index.html` — HTML 작성**

```html
<!DOCTYPE html>
<html>
  <body>
    <h1>Hello Django!</h1>
  </body>
</html>
```

> `templates` 안에 앱 이름 폴더를 만드는 이유: 앱이 여러 개일 때 같은 이름의 템플릿 파일이 충돌하는 것을 방지합니다.

### URL 이름 붙이기 (Naming URL patterns)

```python
# URL에 이름을 붙이면, URL 구조가 바뀌어도 HTML을 수정할 필요가 없습니다.
path('', views.index, name='index'),
```

```html
<!-- ❌ 하드코딩 - URL 바뀌면 전부 수정해야 함 -->
<a href="/articles/">목록</a>

<!-- ✅ url 태그 사용 - name만 살아있으면 자동으로 따라감 -->
<a href="{% url 'articles:index' %}">목록</a>
```

### Variable Routing

URL 일부를 변수처럼 사용해 하나의 함수로 다양한 요청을 처리합니다.

```python
# urls.py
path('<int:article_id>/', views.detail, name='detail'),
```

```python
# views.py
def detail(request, article_id):   # URL에서 꺼낸 값이 키워드 인자로 전달됨
    ...
```

```
/articles/1/ 접속 → article_id = 1
/articles/5/ 접속 → article_id = 5
```

---

## 4. Template — 동적인 HTML 만들기

### DTL (Django Template Language)

`views.py`에서 Python 데이터를 HTML과 결합해 동적인 페이지를 만드는 도구입니다.

| 문법 | 용도 |
|------|------|
| `{{ }}` | 변수 출력 |
| `{% %}` | 로직 실행 (if, for, url...) |
| `{# #}` | 주석 |

**Variable — 변수 출력**

```python
# views.py
context = {
    'name': '주우',
    'items': ['Python', 'Django'],
    'score': 95,
}
return render(request, 'template.html', context)
```

```html
{{ name }}         → 주우
{{ items.0 }}      → Python  (인덱스도 점(.)으로 접근)
```

**Filter — 화면 표시 시 변환**

```html
{{ name|upper }}               → 대문자
{{ content|truncatewords:10 }} → 10단어까지만
{{ name|lower|capfirst }}      → 체이닝 가능
```

**Tag — 로직 실행**

```html
{% if score >= 90 %}
    <p>우수!</p>
{% elif score >= 70 %}
    <p>보통</p>
{% else %}
    <p>분발해봐</p>
{% endif %}

{% for item in items %}
    <p>{{ item }}</p>
{% endfor %}
```

### 템플릿 상속 (Template Inheritance)

공통 뼈대(`base.html`)를 만들고, 다른 템플릿에서 상속받아 사용합니다.

```
프로젝트 루트/
├── templates/
│   └── base.html    ← 루트에 직접 생성 (공통 뼈대)
└── articles/
    └── templates/
        └── articles/
            ├── index.html
            └── detail.html
```

루트의 `templates` 폴더를 Django가 인식하려면 `settings.py`에 등록해야 합니다.

```python
# settings.py
TEMPLATES = [
    {
        'DIRS': [BASE_DIR / 'templates'],  # 루트 templates 폴더 인식
        ...
    }
]
```

```html
<!-- base.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  {% block css %}{% endblock css %}
</head>
<body>
  <a href="{% url 'articles:index' %}">HOME</a>
  <a href="{% url 'articles:create' %}">NEW</a>

  {% block content %}
  {% endblock content %}
</body>
</html>
```

```html
<!-- index.html (자식 템플릿) -->
{% extends 'base.html' %}   ← 반드시 파일 최상단 첫 줄!

{% block content %}
  <h1>게시글 목록</h1>
{% endblock content %}
```

> `{% extends %}` 는 반드시 **파일 최상단 첫 줄**에 있어야 합니다.

---

## 5. Model — 데이터베이스 연결하기

### Model이란?

DB를 Python 클래스로 추상화해서 SQL 없이 파이썬 문법만으로 DB를 조작할 수 있게 해주는 레이어입니다.

```
파이썬 클래스  =  DB 테이블
파이썬 객체    =  DB 행(row) 하나
```

### Model Class 작성

```python
# articles/models.py
from django.db import models

class Article(models.Model):
    title      = models.CharField(max_length=10)   # 짧은 문자열
    content    = models.TextField()                # 긴 문자열
    created_at = models.DateTimeField(auto_now_add=True)  # 생성 시간 자동
    updated_at = models.DateTimeField(auto_now=True)      # 수정 시간 자동
```

`models.Model`을 상속받으면 `.save()`, `.delete()`, `.objects.all()` 같은 CRUD 메서드를 모두 물려받습니다.

> `id` 컬럼은 Django가 자동으로 생성합니다. 직접 쓰지 않아도 됩니다.
> 테이블 이름도 자동으로 `앱이름_클래스이름` 형식으로 만들어집니다 → `articles_article`

### 주요 Field Types

| 필드 | 설명 |
|------|------|
| `CharField(max_length=n)` | 짧은 문자열, `max_length` 필수 |
| `TextField()` | 대용량 텍스트, 길이 제한 없음 |
| `IntegerField()` | 정수 |
| `DateTimeField()` | 날짜+시간 |
| `FileField()` / `ImageField()` | 파일/이미지 |

### 주요 Field Options

| 옵션 | 설명 |
|------|------|
| `null=True` | DB에서 NULL 허용 |
| `blank=True` | 폼에서 빈 값 허용 |
| `default=값` | 기본값 설정 |
| `auto_now_add=True` | 생성 시에만 현재 시간 자동 저장 |
| `auto_now=True` | 저장할 때마다 현재 시간 자동 저장 |

> `null`은 DB 레벨, `blank`는 Django 폼 레벨에서 작동합니다.
> 문자열 필드는 빈 값을 `NULL` 대신 `""` 로 저장하는 것이 Django 관례이므로, 문자열 필드에는 `null=True`를 잘 쓰지 않습니다.

### Migration — 모델 변경사항을 DB에 반영하기

```
models.py 수정  →  makemigrations  →  migrate
(설계도 초안)      (설계도 파일 생성)   (실제 DB에 반영)
```

```bash
python manage.py makemigrations   # migrations/0001_initial.py 파일 생성
python manage.py migrate          # 파일을 읽고 실제 DB 테이블 생성/수정
```

> `models.py`를 수정했다면 **항상 이 두 명령을 순서대로 실행**합니다.

**협업 시 중요한 점:**

`db.sqlite3`는 `.gitignore`에 추가해서 Git에 올리지 않습니다. 대신 **migration 파일**이 Git으로 공유됩니다. 팀원은 `git pull` 후 `migrate`를 직접 실행하면 동일한 DB 구조를 갖게 됩니다.

**유용한 migration 명령어:**

```bash
python manage.py showmigrations           # 적용 여부 확인
python manage.py sqlmigrate articles 0001 # 실제 실행되는 SQL 확인
```

### Django Admin

Django가 자동으로 제공하는 관리자 인터페이스입니다. `/admin`으로 접속합니다.

**사용을 위한 필수 2단계:**

```bash
# Step 1. 슈퍼유저 생성
python manage.py createsuperuser
```

```python
# Step 2. articles/admin.py에 모델 등록
from django.contrib import admin
from .models import Article

admin.site.register(Article)
```

> 둘 중 하나라도 빠지면 admin에서 모델이 보이지 않습니다.

---

## 6. ORM — 파이썬으로 DB 다루기

### ORM이란?

**Object Relational Mapping** — 파이썬 객체와 관계형 DB 테이블을 매핑해주는 기술입니다.

```
[파이썬 세계]     [RDB 세계]
클래스       ←→   테이블
객체         ←→   행 (row)
속성         ←→   열 (column)
```

### QuerySet API 기본 구조

```python
Article.objects.all()
#  ↑       ↑      ↑
# 모델   매니저  메서드
```

`.objects`는 Django가 모든 모델에 자동으로 추가해주는 **매니저**로, DB 접근의 진입점입니다.

### Django Shell로 ORM 연습하기

```bash
pip install ipython
python manage.py shell -v 2   # -v 2 옵션으로 자동 import 목록 확인
```

### CREATE — 데이터 생성

**방법 1, 2: 객체 생성 후 `.save()`**

```python
article = Article(title='첫 번째 글', content='내용입니다')
article.pk       # None  ← 아직 DB에 없음
article.save()
article.pk       # 1     ← DB 저장 후 id 부여
```

**방법 3: `.create()` 한 번에**

```python
article = Article.objects.create(title='두 번째 글', content='내용')
article.pk       # 2     ← 바로 id 부여됨
```

> `.save()` 는 내부적으로 id 유무를 확인해 id가 없으면 `INSERT`, 있으면 `UPDATE`를 실행합니다.

### READ — 데이터 조회

```python
Article.objects.all()              # 전체 조회 → QuerySet 반환
Article.objects.filter(title='A')  # 조건 조회 → QuerySet 반환 (0~n개)
Article.objects.get(pk=1)          # 단일 조회 → Instance 반환 (반드시 1개)
```

| 메서드 | 반환 타입 | 결과가 없거나 여러 개면? |
|--------|-----------|--------------------------|
| `.all()` | QuerySet | 빈 QuerySet `[]` |
| `.filter()` | QuerySet | 빈 QuerySet `[]` |
| `.get()` | Instance | **에러 발생** |

> `.get()`은 고유성이 보장된 `pk`로 조회할 때 주로 사용합니다.

**Field Lookups — 상세 조건 조회**

```python
Article.objects.filter(content__contains='django')    # 포함
Article.objects.filter(title__startswith='첫')        # ~로 시작
Article.objects.filter(pk__gt=2)                      # 초과
```

### UPDATE — 데이터 수정

```python
article = Article.objects.get(pk=1)  # 1. 가져오고
article.title = '수정된 제목'         # 2. 값 바꾸고
article.save()                       # 3. 저장
```

### DELETE — 데이터 삭제

```python
article = Article.objects.get(pk=1)
article.delete()
# (1, {'articles.Article': 1}) ← 삭제된 행 수 반환
```

---

## 7. CRUD View 구현하기

### HTML Form 기초

```html
<form action="/articles/create/" method="POST">
  {% csrf_token %}
  <label for="title">Title:</label>
  <input type="text" name="title" id="title">
  <textarea name="content" id="content"></textarea>
  <input type="submit">
</form>
```

| 속성 | 역할 |
|------|------|
| `action` | 데이터를 어디로 보낼지 |
| `method` | GET 또는 POST |
| `name` | 서버에서 데이터를 꺼낼 때 쓰는 key |

### GET vs POST

| | GET | POST |
|--|-----|------|
| 목적 | 데이터 조회 | 데이터 변경 |
| 데이터 위치 | URL에 노출 | HTTP Body (숨겨짐) |
| 용도 | index, detail, 검색 | create, update, delete |

### CSRF란?

**Cross-Site Request Forgery** — 사용자 의지와 무관하게 공격자가 의도한 요청을 웹사이트에 보내게 만드는 해킹 방식입니다.

브라우저는 쿠키를 **자동으로** 함께 보내는데, 악성 사이트가 이를 이용해 사용자가 모르는 사이에 요청을 위조할 수 있습니다.

```html
<!-- POST 요청이 있는 모든 form에 반드시 추가 -->
<form method="POST">
  {% csrf_token %}
  ...
</form>
```

> `{% csrf_token %}`이 없으면 Django가 **403 Forbidden**으로 막습니다.

### redirect란?

```python
# ❌ render로 응답하면 새로고침 시 POST 재전송 → DB에 중복 저장
return render(request, 'articles/create.html')

# ✅ redirect로 응답 → 새로고침해도 안전
return redirect('articles:detail', article.pk)
```

`redirect`는 클라이언트에게 "이 URL로 GET 요청을 다시 보내"라고 응답합니다. 요청이 총 2번 일어나지만, 사용자 눈에는 자연스럽게 이동하는 것처럼 보입니다. 이를 **PRG(Post/Redirect/Get) 패턴**이라고 합니다.

### CREATE 구현

```python
# articles/views.py
from django.shortcuts import render, redirect
from .models import Article

def new(request):
    return render(request, 'articles/new.html')

def create(request):
    title = request.POST.get('title')
    content = request.POST.get('content')
    article = Article(title=title, content=content)
    article.save()
    return redirect('articles:detail', article.pk)
```

### READ 구현

```python
def index(request):
    articles = Article.objects.all()
    context = {'articles': articles}
    return render(request, 'articles/index.html', context)

def detail(request, article_id):
    article = Article.objects.get(pk=article_id)
    context = {'article': article}
    return render(request, 'articles/detail.html', context)
```

### UPDATE 구현

```python
def edit(request, article_id):
    article = Article.objects.get(pk=article_id)
    context = {'article': article}
    return render(request, 'articles/edit.html', context)

def update(request, article_id):
    article = Article.objects.get(pk=article_id)
    article.title = request.POST.get('title')
    article.content = request.POST.get('content')
    article.save()
    return redirect('articles:detail', article_id)
```

### DELETE 구현

```python
def delete(request, article_id):
    article = Article.objects.get(pk=article_id)
    article.delete()
    return redirect('articles:index')
```

---

## 8. ModelForm — 유효성 검사까지 한 번에

### HTML Form의 한계

지금까지의 방식은 서버 입장에서 들어오는 데이터를 그냥 믿어야 한다는 문제가 있습니다.

- 이메일 칸에 엉뚱한 값이 들어와도 알 수 없음
- 빈 값이 들어와도 필터링 불가
- 유효성 검사 로직을 별도로 직접 구현해야 함

### Django Form → ModelForm으로

```
HTML Form → 유효성 검사 없음, 직접 구현 필요
    ↓
Django Form → 유효성 검사 자동화, but 모델과 필드 중복 발생
    ↓
Django ModelForm → 모델 기반 자동 생성, 중복 제거, DB 저장까지!
```

### ModelForm 정의

```python
# articles/forms.py (직접 생성해야 합니다!)
from django import forms
from .models import Article

class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'content']
        # fields = '__all__'       # 전체 필드 사용
        # exclude = ('title',)    # 특정 필드 제외
```

> `Meta` 클래스는 ModelForm에 대한 설정 정보를 담는 **약속된 공간**입니다. 어떤 모델과 연결할지, 어떤 필드를 사용할지를 정의합니다.

### Widget — 입력창 모양 바꾸기

```python
class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'content']
        widgets = {
            'title': forms.TextInput(attrs={
                'placeholder': '제목을 입력하세요',
                'class': 'form-control',  # Bootstrap 클래스 적용 가능
            }),
            'content': forms.Textarea(attrs={
                'rows': 5,
                'class': 'form-control',
            }),
        }
```

### CREATE / UPDATE 통합 뷰

HTTP method 차이를 활용하면 `new + create`, `edit + update`를 각각 하나의 함수로 통합할 수 있습니다.

```python
# CREATE — new + create 통합
def create(request):
    if request.method == 'POST':
        form = ArticleForm(request.POST)
        if form.is_valid():
            article = form.save()
            return redirect('articles:detail', article.pk)
    else:
        form = ArticleForm()
    context = {'form': form}
    return render(request, 'articles/create.html', context)
```

```python
# UPDATE — edit + update 통합
def update(request, pk):
    article = Article.objects.get(pk=pk)
    if request.method == 'POST':
        form = ArticleForm(request.POST, instance=article)  # instance가 핵심!
        if form.is_valid():
            article = form.save()
            return redirect('articles:detail', article.pk)
    else:
        form = ArticleForm(instance=article)  # 기존 데이터 채워서 보여줌
    context = {'article': article, 'form': form}
    return render(request, 'articles/update.html', context)
```

**create vs update의 차이는 딱 하나, `instance=article`의 유무입니다.**

```python
# create
form = ArticleForm(request.POST)

# update
form = ArticleForm(request.POST, instance=article)
```

| `instance` | 동작 | SQL |
|---|---|---|
| 없음 | 새 객체 생성 | `INSERT` |
| 있음 | 기존 객체 수정 | `UPDATE` |

### 템플릿에서의 활용

```html
<!-- create.html -->
{% extends 'base.html' %}
{% block content %}
<h1>CREATE</h1>
<form action="" method="POST">
  {% csrf_token %}
  {{ form }}       ← 이 한 줄이 label, input, 유효성 에러 메시지까지 자동 생성
  <input type="submit">
</form>
{% endblock content %}
```

`{{ form }}`이 자동으로 생성해주는 것들:

| 자동 생성 항목 | 예시 |
|---|---|
| `<label>` | `<label for="id_title">Title:</label>` |
| `name` 속성 | `name="title"` |
| `maxlength` 속성 | `maxlength="10"` (models.py의 max_length 자동 반영) |
| `required` 속성 | 자동 추가 |
| 유효성 에러 메시지 | is_valid() 실패 시 자동 표시 |

---

## 9. 최종 URL 구조 정리

```python
# articles/urls.py
from django.urls import path
from . import views

app_name = 'articles'

urlpatterns = [
    path('', views.index, name='index'),                          # 목록
    path('<int:pk>/', views.detail, name='detail'),               # 상세
    path('create/', views.create, name='create'),                 # 작성 (GET+POST 통합)
    path('<int:pk>/update/', views.update, name='update'),        # 수정 (GET+POST 통합)
    path('<int:pk>/delete/', views.delete, name='delete'),        # 삭제
]
```

| 기능 | URL | Method | View 함수 |
|------|-----|--------|-----------|
| 목록 조회 | `articles/` | GET | `index` |
| 상세 조회 | `articles/<pk>/` | GET | `detail` |
| 작성 폼 + 저장 | `articles/create/` | GET / POST | `create` |
| 수정 폼 + 저장 | `articles/<pk>/update/` | GET / POST | `update` |
| 삭제 | `articles/<pk>/delete/` | POST | `delete` |

---

## 마무리 — 전체 흐름 한눈에 보기

```
빈 폴더
  │
  ├── 가상환경 + Django 설치
  ├── 프로젝트 + 앱 생성
  ├── settings.py 앱 등록
  │
  ├── Model 작성 → makemigrations → migrate
  │
  ├── urls.py 경로 등록
  ├── views.py 함수 작성
  ├── templates HTML 작성
  │
  ├── ORM으로 DB 연동
  ├── Form → ModelForm으로 유효성 검사
  │
  └── CRUD 완성!
```

Django 1~6장에서 다룬 내용의 핵심은 결국 **"사용자 요청 → URL → View → Model → Template → 응답"** 이 흐름을 반복하는 것입니다. 

프로젝트에서는 이 흐름 위에 앱 간의 관계(ForeignKey), 로그인/인증 기능 등이 추가됩니다. 기본 흐름을 확실히 익혀두면 어떤 기능이 추가되어도 구조적으로 접근할 수 있습니다.