# Django 2장 복습 🐍
> Django Template & URLs 복습 정리

---

## 📌 Django Template System이란?

`views.py`의 함수에서 Python 데이터(context)를 HTML 문서(Template)와 결합하여 **로직과 표현을 분리**한 채 동적인 웹페이지를 생성하는 도구.

```
views.py → "어떤 데이터를 보여줄지" 결정 (Python 로직)
template → "어떻게 보여줄지" 결정 (표현)
```

**전체 흐름:**
```
사용자 요청
    ↓
urls.py → views.py 함수 호출
    ↓
views.py → context 딕셔너리 생성
    ↓
render(request, 'template.html', context)  ← 여기서 결합!
    ↓
동적 HTML 완성 → 사용자에게 응답
```

> ☁️ **DevOps 연결**: 나중에 Docker로 배포할 때 `TEMPLATES` 경로 설정이 컨테이너 내부 경로랑 맞아야 하고, Nginx 같은 웹서버가 정적 파일을 따로 서빙하는 구조가 됨!

---

## 📌 DTL (Django Template Language)

### Variable `{{ }}`

`views.py`의 context 딕셔너리에서 **key 이름**으로 접근해 값을 출력.

```python
# views.py
context = {
    'name': '주우',
    'user': {'age': 25},
    'items': ['a', 'b', 'c']
}
```

```html
{{ name }}       → 주우
{{ user.age }}   → 25      (점(.)으로 중첩 접근)
{{ items.0 }}    → a       (인덱스도 점(.)으로 접근)
```

> ⚠️ `{{ value }}` 가 아니라 **`{{ key }}`** 를 쓰는 것!

---

### Filters `{{ variable|filter }}`

변수를 **화면에 표시할 때만** 변환. 원본 데이터는 그대로 유지됨.

```html
{{ name|upper }}               → 대문자 변환
{{ name|lower }}               → 소문자 변환
{{ content|truncatewords:10 }} → 10단어까지만 자르기
{{ number|add:5 }}             → 숫자 더하기
{{ date|date:"Y-m-d" }}        → 날짜 포맷 지정

<!-- 필터 체이닝도 가능! -->
{{ name|lower|capfirst }}      → 소문자 후 첫 글자만 대문자
```

**views.py에서 똑같은 작업을 하려면?**

Filter는 HTML에서 표시용으로 쓰는 거고, `views.py`에서 Python으로 직접 가공해서 context에 담을 수도 있음.

```python
# views.py
name = '주우'

context = {
    'name_upper': name.upper(),          # {{ name|upper }} 와 동일
    'name_lower': name.lower(),          # {{ name|lower }} 와 동일
    'content_short': content[:50],       # truncatewords 유사
    'number_added': number + 5,          # {{ number|add:5 }} 와 동일
    'date_formatted': date.strftime('%Y-%m-%d'),  # {{ date|date:"Y-m-d" }} 와 동일
}
```

> 💡 **언제 뭘 쓰냐면?**
> - 단순 **화면 표시용 변환** → Template Filter (`{{ name|upper }}`)
> - **로직에서도 가공된 값이 필요**하거나 복잡한 처리 → `views.py`에서 Python으로 처리 후 context에 담기

---

### Tags `{% %}`

HTML에서 **로직을 실행**할 때 사용. 열고 닫는 태그가 필요한 것들이 있음.

```html
<!-- 조건문 -->
{% if score >= 90 %}
    <p>우수!</p>
{% elif score >= 70 %}
    <p>보통</p>
{% else %}
    <p>분발해봐</p>
{% endif %}

<!-- 반복문 -->
{% for item in items %}
    <p>{{ item }}</p>
{% endfor %}

<!-- URL 역참조 -->
{% url 'name' %}
```

---

### Comments (주석)

```html
<!-- 한 줄 주석 -->
{# 이건 화면에 안 보여! #}

<!-- 여러 줄 주석 -->
{% comment %}
    이것도 화면에 안 보이고
    HTML 소스에도 안 나타나!
{% endcomment %}
```

| 종류 | 문법 | 브라우저 소스보기에서 |
|------|------|------|
| HTML 주석 | `<!-- -->` | **보임** ⚠️ |
| DTL 주석 | `{# #}` / `{% comment %}` | **안 보임** ✅ |

---

### DTL 문법 정리

| 문법 | 용도 |
|------|------|
| `{{ }}` | **변수** 출력 |
| `{% %}` | **로직** 실행 (if, for, url...) |
| `{# #}` | **주석** |

---

## 📌 context와 Template 연결 예시

```python
# views.py
def my_view(request):
    context = {
        'user': '주우',
        'items': ['Python', 'Django', 'DevOps'],
        'score': 95,
    }
    return render(request, 'my_template.html', context)
```

```html
<!-- my_template.html -->

<!-- Variable -->
<h1>안녕 {{ user }}!</h1>

<!-- Filter -->
<p>{{ user|upper }}</p>

<!-- Tag - if -->
{% if score >= 90 %}
    <p>우수!</p>
{% else %}
    <p>분발해봐</p>
{% endif %}

<!-- Tag - for -->
<ul>
    {% for item in items %}
        <li>{{ item }}</li>
    {% endfor %}
</ul>
```

**🖥️ 결과 화면:**

```
안녕 주우!         ← {{ user }}
주우               ← {{ user|upper }} (한글이라 변화 없음, 영문이면 JOOWOO)

우수!              ← score=95 이므로 if 조건 충족

• Python           ← for 반복
• Django
• DevOps
```

---

## 📌 Template 상속 (Inheritance)

**skeleton 코드** 역할을 하는 상위 템플릿(`base.html`)을 만들고, 다른 템플릿에서 상속받아 사용.

```html
<!-- base.html (뼈대) -->
<!DOCTYPE html>
<html lang="en">
<head>...</head>
<body>
    {% block content %}
    {% endblock content %}
</body>
</html>
```

```html
<!-- dinner.html (자식) -->
{% extends 'articles/base.html' %}   ← 반드시 파일 최상단 첫 줄!

{% block content %}
    <h1>오늘의 메뉴</h1>
    <p>{{ picked }}</p>
{% endblock content %}
```

> ⚠️ `{% extends %}` 는 반드시 **파일 최상단 첫 줄**에 있어야 함!

**block 이름 규칙:**

`content` 는 그냥 이름일 뿐이야. **base.html과 자식 템플릿의 block 이름이 서로 일치해야** 연결됨!

```html
<!-- base.html -->
{% block content %}      ← 이름이 'content'
{% endblock content %}

<!-- dinner.html -->
{% block content %}      ← 똑같이 'content' 여야 연결됨!
{% endblock content %}
```

`content` 대신 다른 이름도 얼마든지 가능하고, 여러 개의 block을 만들 수도 있음:

```html
<!-- base.html -->
<head>
    {% block title %}{% endblock title %}      ← title 블록
</head>
<body>
    {% block content %}{% endblock content %}  ← content 블록
    {% block sidebar %}{% endblock sidebar %}  ← sidebar 블록
</body>
```

```html
<!-- 자식 템플릿 -->
{% extends 'articles/base.html' %}

{% block title %}<title>메뉴 페이지</title>{% endblock title %}
{% block content %}<h1>오늘의 메뉴</h1>{% endblock content %}
{% block sidebar %}<p>사이드바 내용</p>{% endblock sidebar %}
```

> ☁️ **DevOps 연결**: `base.html` 하나만 수정하면 전체 페이지에 공통 CSS, JS, CDN 링크가 반영됨. Docker 이미지 빌드할 때 정적 파일 관리가 훨씬 깔끔해짐!

---

## 📌 HTML Form

사용자와 애플리케이션 간의 상호작용을 위한 요소.

```html
<form action="/catch/" method="GET">
    <label for="food-input">메뉴 입력</label>
    <input type="text" id="food-input" name="food">
    <button type="submit">제출</button>
</form>
```

| 속성 | 역할 |
|------|------|
| `action` | 데이터를 **어디로** 보낼지 (URL) |
| `method` | 데이터를 **어떻게** 보낼지 (GET/POST) |
| `name` | 서버에서 데이터를 **꺼낼 때** 쓰는 key |
| `id` | HTML/CSS/JS가 요소를 **식별**하는 이름표 (페이지에서 유일해야 함) |

**label과 input 관계:**

```html
<label for="food-input">메뉴 입력</label>   ← for="food-input"
<input type="text" id="food-input" name="food">  ← id="food-input"
```

`for`와 `id`가 일치하면:
- 라벨 클릭 시 input 자동 포커스 ✅
- 스크린리더가 어떤 input인지 읽어줌 (웹 접근성) ✅

**GET vs POST 차이:**

| | GET | POST |
|------|------|------|
| 데이터 위치 | **URL에 노출** (`?key=value`) | **HTTP Body에 숨겨짐** |
| 보안 | 낮음 (URL에 다 보임) ⚠️ | 높음 (URL에 안 보임) ✅ |
| 용도 | **데이터 조회** (검색, 필터) | **데이터 생성/수정/삭제** |
| 북마크 | 가능 (URL에 담기므로) | 불가능 |
| 예시 | 검색창, 필터링 | 로그인, 회원가입, 게시글 작성 |

```html
<!-- GET - 검색처럼 URL에 노출돼도 괜찮을 때 -->
<form action="/search/" method="GET">
    <input type="text" name="q">
</form>
<!-- 결과: /search/?q=Django -->

<!-- POST - 비밀번호처럼 URL에 노출되면 안 될 때 -->
<form action="/login/" method="POST">
    {% csrf_token %}   ← POST는 반드시 필요! (보안)
    <input type="password" name="password">
</form>
```

> ⚠️ POST 사용 시 `{% csrf_token %}` 필수! Django의 보안 기능으로, 없으면 403 에러 발생.

> ☁️ **DevOps 연결**: 실제 배포 환경에서 GET 요청은 CDN/Nginx에서 캐싱 가능하지만, POST는 캐싱 안 됨. 이 차이가 서버 부하와 성능에 영향을 줌!

---

## 📌 throw - catch 흐름

### 1/2 - throw 페이지 접속

```
1. 브라우저에서 /throw/ 요청
        ↓
2. urls.py에서 throw/ 와 일치하는 path 찾아 views.throw 호출
        ↓
3. views.throw 함수 실행
        ↓
4. throw view 함수가 응답 객체 반환
        ↓
5. 응답 객체를 브라우저에 전달
        ↓
6. 브라우저가 응답 해석 후 throw 페이지 화면 출력
```

### 2/2 - throw에서 데이터 입력 후 catch로 이동

```
1. throw 페이지 form에 데이터 작성 후 제출
   (form action="/catch/" 으로 요청)
        ↓
2. /catch/?message=안녕! 으로 URL 변경되며 요청
        ↓
3. urls.py에서 catch/ 와 일치하는 path 찾아 views.catch 호출
        ↓
4. views.catch 함수 실행
        ↓
5. request.GET.get('message') 로 데이터 추출 후 응답 객체 반환
        ↓
6. 응답 객체를 브라우저에 전달
        ↓
7. 브라우저가 응답 해석 후 catch 페이지 화면 출력
```

### 전체 코드로 보기

**📁 urls.py**
```python
from django.urls import path
from . import views

urlpatterns = [
    path('throw/', views.throw, name='throw'),
    path('catch/', views.catch, name='catch'),
]
```

**📁 views.py**
```python
# throw 함수 - 그냥 throw.html 보여주기만 함
def throw(request):
    return render(request, 'articles/throw.html')
    # context 없음! 그냥 빈 폼 페이지만 렌더링

# catch 함수 - throw에서 보낸 데이터를 받아서 처리
def catch(request):
    # 1. URL에서 데이터 꺼내기
    #    /catch/?message=안녕! → request.GET = {'message': '안녕!'}
    message = request.GET.get('message')

    # 2. context에 담기
    context = {
        'message': message,
    }

    # 3. catch.html에 전달
    return render(request, 'articles/catch.html', context)
```

**📁 articles/throw.html**
```html
{% extends 'articles/base.html' %}

{% block content %}
    <h1>Throw</h1>
    <form action="/catch/" method="GET">
    <!--           ↑ catch/로 데이터 전송    ↑ URL에 담아서 보냄 -->
        <input type="text" id="message" name="message">
        <!--                             ↑ 이 name이 key가 됨! -->
        <input type="submit">
    </form>
{% endblock content %}
```

**📁 articles/catch.html**
```html
{% extends 'articles/base.html' %}

{% block content %}
    <h1>Catch</h1>
    <p>{{ message }}</p>   ← context의 'message' key로 출력
{% endblock content %}
```

**"message" 이름이 전체를 관통하는 흐름:**

```
throw.html  <input name="message">
                    ↓ form 제출
URL         /catch/?message=안녕!
                    ↓
views.py    request.GET = {'message': ['안녕!']}
            message = request.GET.get('message')  → '안녕!'
            context = {'message': message}
                    ↓
catch.html  {{ message }}  →  안녕!
```

**request.GET vs request.GET.get():**

| | 설명 |
|------|------|
| `request.GET` | URL `?` 뒷부분을 Django가 딕셔너리로 만든 것 (QueryDict) |
| `request.GET['message']` | key로 직접 접근 - key 없으면 **에러 💥** |
| `request.GET.get('message')` | 안전하게 꺼내기 - key 없으면 **None 반환 ✅** |

```
method="GET"  →  request.GET  에 데이터 있음
method="POST" →  request.POST 에 데이터 있음
```

---

## 📌 URL dispatcher

URL 패턴을 정의하고 해당 패턴이 일치하는 요청을 처리할 `views.py` 안의 함수와 연결(매핑)하는 역할.

> **dispatcher** = "배차원, 교통정리하는 사람"

```python
urlpatterns = [
    path('throw/', views.throw),   # /throw/ → views.throw
    path('catch/', views.catch),   # /catch/ → views.catch
]
```

`urlpatterns` 리스트를 **위에서부터 순서대로** 훑으면서 일치하는 패턴을 찾음!

> ☁️ **DevOps 연결**: Nginx도 비슷한 역할을 함. URL 패턴 보고 Django로 넘길지, 정적파일 직접 서빙할지 결정함!

---

## 📌 Variable Routing

URL 일부를 변수처럼 사용해 하나의 함수로 다양한 요청을 처리.

```python
# urls.py
path('articles/<int:num>/', views.detail)
path('hello/<str:name>/', views.greeting)
```

```python
# views.py
def detail(request, num):   # num이 키워드 인자로 전달됨!
    context = {'num': num}
    return render(request, 'articles/detail.html', context)

def greeting(request, name):
    context = {'name': name}
    return render(request, 'articles/greeting.html', context)
```

```
articles/1/ 접속 → num = 1
articles/5/ 접속 → num = 5
articles/999/ 접속 → num = 999
```

**Variable Routing 없었다면:**
```python
# 이걸 일일이 다 써야 함 😱
path('articles/1/', views.detail_1),
path('articles/2/', views.detail_2),
# ... 100개면 100줄
```

> 나중에 DB 연결하면 `num`으로 해당 게시글 데이터를 조회하는 key로 활용됨!

---

## 📌 App URL mapping (include)

각 앱마다 `urls.py`를 따로 만들어 관리하고, 프로젝트 `urls.py`에서 위임.

```python
# config/urls.py
from django.urls import path, include

urlpatterns = [
    path('articles/', include('articles.urls')),  # articles 앱한테 위임!
    path('users/', include('users.urls')),
]
```

```python
# articles/urls.py
from django.urls import path
from . import views    # . = 현재 패키지(폴더) = articles 폴더

urlpatterns = [
    path('', views.index),           # → articles/
    path('create/', views.create),   # → articles/create/
    path('<int:num>/', views.detail), # → articles/1/, articles/2/ ...
]
```

**파일 구조:**
```
프로젝트 루트/
├── config/
│   └── urls.py     ← 프로젝트 urls (관문 역할, 자동 생성)
└── articles/
    ├── urls.py     ← 앱 urls (직접 만들어야 함! ⚠️)
    └── views.py    ← 자동 생성
```

> ☁️ **DevOps 연결**: 앱별 urls.py 분리는 나중에 마이크로서비스로 쪼갤 때도 편함. 각 앱이 독립적으로 관리되니까 Docker 컨테이너로 분리하기도 수월함!

---

## 📌 Naming URL patterns

URL에 이름을 붙여서 URL 구조가 바뀌어도 HTML을 수정할 필요 없게 만듦.

```python
# articles/urls.py
app_name = 'articles'
urlpatterns = [
    path('', views.index, name='index'),
    path('<int:num>/', views.detail, name='detail'),
]
```

```html
<!-- ❌ 하드코딩 - URL 바뀌면 전부 수동으로 찾아서 고쳐야 함 -->
<a href="/articles/">목록</a>

<!-- ✅ url 태그 - URL 바뀌어도 name만 살아있으면 자동으로 따라감 -->
<a href="{% url 'articles:index' %}">목록</a>

<!-- Variable Routing 있을 때 인자 넘기기 -->
{% url 'articles:detail' 3 %}    → articles/3/
{% url 'articles:detail' num %}  → articles/{{ num }}/
```

**최종적으로 브라우저 주소창에 보이는 URL:**

```
{% url 'articles:index' %}     →  http://127.0.0.1:8000/articles/
{% url 'articles:detail' 3 %}  →  http://127.0.0.1:8000/articles/3/
{% url 'users:index' %}        →  http://127.0.0.1:8000/users/
```

즉 `{% url %}` 태그는 **HTML 렌더링 시점에 실제 URL 문자열로 변환**되는 거야.
개발환경이든 배포환경이든 도메인만 바뀌고 `/articles/` 뒷부분은 동일하게 생성됨!

**app_name이 필요한 이유:**

```python
# articles/urls.py
path('', views.index, name='index'),   # name='index' 충돌 위험!

# users/urls.py
path('', views.index, name='index'),   # name='index' 충돌 위험!
```

`app_name`으로 네임스페이스를 분리해서 충돌 방지:

```html
{% url 'articles:index' %}   → articles/
{% url 'users:index' %}      → users/
```

**한 줄 요약:**

| | 역할 |
|------|------|
| `name` | URL에 이름 붙이기 |
| 인자 | Variable Routing 있을 때 값 넘기기 |
| `app_name` | 앱 간 name 충돌 방지 |

> ☁️ **DevOps 연결**: `{% url %}` 태그 덕분에 개발환경(`127.0.0.1:8000`)이랑 운영환경(`https://mysite.com`) URL 구조가 달라져도 코드 수정 없이 그대로 동작함!

---

## ☁️ DevOps 연결 포인트 정리

| Django 개념 | DevOps 연결 |
|---|---|
| `TEMPLATES` 경로 설정 | Docker 컨테이너 내부 경로랑 일치해야 함 |
| Template 상속 구조 | 공통 CSS/JS를 `base.html` 하나로 관리 → 정적 파일 관리 효율화 |
| URL dispatcher | Nginx의 라우팅 역할과 동일한 개념 |
| App URL mapping | 마이크로서비스 분리, Docker 컨테이너 독립화에 유리 |
| `{% url %}` 태그 | 환경(개발/운영)이 바뀌어도 URL 하드코딩 없이 동작 |
| `app_name` 네임스페이스 | 마이크로서비스의 독립적 네임스페이스와 동일한 개념 |