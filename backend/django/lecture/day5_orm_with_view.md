# Django ORM with View - 5장 🗄️
> ORM을 View에 적용하여 CRUD 기능 구현하기

---

## 📌 템플릿 상속 (base.html)

### 프로젝트 구조

```
myproject/
├── manage.py
├── templates/          ← 루트에 새로 생성 (공통 템플릿)
│   └── base.html
├── articles/
│   └── templates/
│       └── articles/
│           ├── index.html
│           ├── detail.html
│           ├── new.html
│           └── edit.html
```

### settings.py - TEMPLATES DIRS 등록

```python
TEMPLATES = [
    {
        ...
        'DIRS': [BASE_DIR / 'templates'],  # 루트 templates 폴더 인식
        ...
    }
]
```

> 💡 Django는 기본적으로 **앱 안의 templates 폴더**만 탐색한다.
> 앱 밖에 만든 폴더는 `DIRS`에 직접 등록해줘야 찾을 수 있다!

### HTML 태그 기초 (base.html 이해를 위한 필수 지식)

HTML은 **태그(tag)** 라는 꺾쇠괄호(`< >`)로 감싼 명령어들로 이루어진 언어.
대부분 여는 태그 `<태그명>` 과 닫는 태그 `</태그명>` 쌍으로 구성.

**📄 HTML 문서 뼈대**

| 태그 | 이름 | 역할 |
|------|------|------|
| `<!DOCTYPE html>` | 문서 선언 | "나 HTML5 문서야!" 라고 브라우저에게 알려주는 것. 맨 첫 줄에 항상 써줌 |
| `<html>` | HTML 루트 | 전체 HTML 문서를 감싸는 최상위 태그 |
| `<head>` | 헤드 | **화면에 안 보이는** 설정 정보 영역. CSS 연결, 탭 제목, 문자 인코딩 등 |
| `<body>` | 바디 | **화면에 실제로 보이는** 내용 영역. 우리가 보는 모든 것이 여기 들어감 |

**📝 내용 표시용 태그**

| 태그 | 이름 | 역할 |
|------|------|------|
| `<h1>` | 제목 태그 | 제목(Heading)을 나타냄. h1이 가장 크고 h6이 가장 작음. h1 → h2 → h3 순서로 크기 줄어듦 |
| `<p>` | 문단 태그 | 단락(Paragraph)을 나타냄. 글 한 줄 한 줄을 감쌀 때 주로 사용. 위아래로 자동 여백이 생김 |
| `<div>` | 구역 태그 | 여러 태그들을 하나의 **박스(구역)**로 묶을 때 사용. 그 자체로는 아무 모양 없음. CSS로 스타일 줄 때 묶음 단위로 많이 활용 |
| `<hr>` | 수평선 | 화면에 구분선(가로줄)을 그어주는 태그. 닫는 태그 없음 |

**🖱️ 상호작용 태그**

| 태그 | 이름 | 역할 |
|------|------|------|
| `<a href="주소">` | 링크 태그 | 클릭하면 다른 페이지로 이동하는 링크. `href` 속성에 이동할 주소를 넣음 |
| `<form>` | 폼 태그 | 사용자 입력 데이터를 서버로 전송하는 영역. `action`(어디로 보낼지), `method`(GET/POST) 속성을 설정 |
| `<label>` | 라벨 태그 | 입력 필드에 이름표를 붙여주는 태그. `for` 속성값과 `input`의 `id`값이 같으면 라벨 클릭 시 해당 입력창으로 포커스 이동 |
| `<input>` | 입력 태그 | 사용자가 텍스트를 입력하는 한 줄짜리 입력창. `type="text"` (일반 텍스트), `type="submit"` (제출 버튼), `type="hidden"` (숨겨진 값) 등 type에 따라 모양이 달라짐 |
| `<textarea>` | 텍스트영역 태그 | 여러 줄 입력이 가능한 입력창. `input`과 달리 닫는 태그가 있고 태그 사이에 기본값을 넣을 수 있음 |

```
a 태그 예시:
<a href="/articles/new/">NEW</a>
  ↑              ↑         ↑
a 태그       이동할 주소   화면에 보이는 텍스트
(링크)
```

```
p 태그 예시:
<p>글 제목: second</p>   ← 한 줄의 문단
<p>글 내용: django!</p>  ← 또 다른 문단 (위아래 자동 여백 생김)
```

```
div 태그 예시:
<div>                        ← 박스 시작 (여러 요소를 하나로 묶음)
  <p>글 번호: 2</p>
  <p>글 제목: second</p>
  <p>글 내용: django!</p>
</div>                       ← 박스 끝
<div>                        ← 다음 글도 하나의 박스로
  <p>글 번호: 3</p>
  <p>글 제목: third</p>
</div>

→ 각 게시글을 div로 묶으면 나중에 CSS로 한 번에 스타일 적용 가능!
```

```
form 태그 예시:
<form action="/articles/create/" method="POST">
         ↑                           ↑
    어디로 보낼지                GET or POST
  ...입력 태그들...
</form>
```

```
label + input 태그 예시:
<label for="title">Title: </label>
          ↑
      id="title" 인 input과 연결됨 (라벨 클릭하면 입력창으로 포커스 이동)

<input type="text" name="title" id="title">
         ↑               ↑          ↑
      한 줄 입력창   views.py에서    label의
                    꺼낼 때 쓰는    for와 연결
                    이름

→ label의 for 값 == input의 id 값 이어야 연결됨!
```

```
textarea 태그 예시:
<textarea name="content" id="content"></textarea>   ← 빈 입력창 (new.html)
<textarea name="content" id="content">django!</textarea>  ← 기존값 채워진 입력창 (edit.html)
                                          ↑
                               태그 사이에 값을 넣으면 기본값으로 표시됨
                               (input의 value 속성과 같은 역할)
```

> 💡 **head vs body 한 줄 요약**
> - `<head>` = 브라우저가 읽는 설정값 (사용자 눈에 안 보임)
> - `<body>` = 사용자가 실제로 보는 화면 내용

### 네비게이션 바(Navigation Bar)란?

> 웹사이트 어느 페이지에 있든 **항상 상단에 고정으로 보이는 메뉴 모음**

우리가 자주 쓰는 사이트들 상단에 있는 메뉴 바가 바로 네비게이션 바.
예) 네이버 상단의 메일 | 카페 | 블로그 |

base.html의 body 안에 네비게이션 바를 넣으면 → **모든 페이지에 공통으로 표시**됨.
각 페이지(index.html, detail.html 등)가 base.html을 상속받으니..

```
[모든 페이지에서 보임]          [페이지마다 다른 내용]
┌─────────────────────────┐
│  네비게이션 바   NEW     │  ← base.html의 body (공통)
├─────────────────────────┤
│                         │
│   여기가 block content  │  ← 각 페이지의 내용 (index, detail 등)
│                         │
└─────────────────────────┘
```

### base.html

```html
<!DOCTYPE html>              <!-- HTML5 문서 선언 -->
<html lang="en">             <!-- HTML 전체를 감싸는 루트 태그 -->
<head>                       <!-- 화면에 안 보이는 설정 영역 -->
  {% block css %}            <!-- 자식 페이지별 CSS를 여기에 끼워넣을 수 있는 구멍 -->
  {% endblock css %}
</head>
<body>                       <!-- 화면에 보이는 내용 영역 -->
  <h1>네비게이션 바</h1>     <!-- 모든 페이지 상단에 보이는 제목 -->
  <a href="{% url 'articles:new' %}">NEW</a>  <!-- NEW 클릭하면 글 작성 페이지로 이동 -->

  {% block content %}        <!-- 자식 페이지의 내용이 여기에 끼워짐 -->
  {% endblock content %}
</body>
</html>
```

> 💡 **block 이름이 같아야 한다!**
> `base.html`의 `{% block content %}`와 자식 템플릿의 `{% block content %}`
> 이름이 다르면 에러도 안 나고 조용히 내용이 사라진다 → 찾기 어려운 버그!

> 💡 **block은 여러 개 만들 수 있다!**
> - `block css` → 페이지별 스타일
> - `block content` → 페이지별 내용
> base.html에 넣은 건 **모든 페이지에 공통 적용**된다.

### 자식 템플릿 상속 방법

```html
{% extends 'base.html' %}

{% block content %}
<!-- 이 안에 페이지별 내용 작성 -->
{% endblock content %}
```

---

## 📌 READ - 단일 게시글 조회 (Detail)

### 전체 흐름

```
① 브라우저 주소창에 /articles/2/ 입력
           ↓
② urls.py → <int:article_id> 에서 숫자 2 추출 → views.detail 호출
           ↓
③ views.py → Article.objects.get(pk=2) → DB에서 2번 글 1개 조회
           ↓
④ context에 담아서 detail.html로 전달
           ↓
⑤ detail.html → 글 정보 화면에 출력
```

---

### ① urls.py

```python
# articles/urls.py
path('<int:article_id>/', views.detail, name='detail'),
#         ↑
# URL에서 숫자를 꺼내서 article_id 라는 이름으로 저장
# 예) /articles/2/ 접속하면 article_id = 2
```

---

### ② views.py

```python
def detail(request, article_id):          # URL에서 article_id(=2) 받아옴
    article = Article.objects.get(pk=article_id)  # DB에서 pk=2인 글 1개 조회
    context = {
        'article': article,               # 조회한 글을 템플릿에 넘길 준비
    }
    return render(request, 'articles/detail.html', context)  # detail.html 렌더링
```

```sql
-- Article.objects.get(pk=2)
SELECT * FROM articles_article WHERE id = 2;
-- 가져와라  모든 컬럼을  테이블에서  id가 2인 것 (딱 1개만)
```

> 💡 `all()` 은 전체 다 가져오고, `get()` 은 **조건에 맞는 1개만** 가져온다!

---

### ③ detail.html

```html
{% extends 'base.html' %}
{% block content %}
  <h2>DETAIL</h2>
  <h3>{{ article.pk }} 번째 글</h3>   <!-- views.py에서 넘긴 article 객체의 pk -->
  <hr>
  <p>제목: {{ article.title }}</p>
  <p>내용: {{ article.content }}</p>
  <p>작성일: {{ article.created_at }}</p>
  <p>수정일: {{ article.updated_at }}</p>
  <hr>
  <a href="{% url 'articles:index' %}">[back]</a>   <!-- 클릭하면 목록으로 이동 -->
{% endblock content %}
```

**📺 브라우저 화면 결과 (`127.0.0.1:8000/articles/2/`)**
```
네비게이션 바   NEW
────────────────────────────
DETAIL

2 번째 글
────────────────────────────
제목: second
내용: django!
작성일: 2026-04-15 05:55:06
수정일: 2026-04-15 05:55:06
────────────────────────────
[back]
```

---

### ④ index.html - 제목을 링크로 (목록 → 상세 연결)

```html
{% extends 'base.html' %}
{% block content %}
  <h1>Article</h1>
  <hr>
{% for article in articles %}
  <div>
    <p>글 번호: {{ article.pk }}</p>
    <p>글 제목: <a href="{% url 'articles:detail' article.pk %}">{{ article.title }}</a></p>
  <!--                                              ↑
                               article.pk를 URL에 넘겨서 해당 글 상세페이지로 이동 -->
    <hr>
  </div>
{% endfor %}
{% endblock content %}
```

**📺 브라우저 화면 결과 (`127.0.0.1:8000/articles/`)**
```
네비게이션 바   NEW
────────────────────────────
Article
────────────────────────────
글 번호: 2
글 제목: second  ← 클릭하면 /articles/2/ 로 이동!
────────────────────────────
글 번호: 3
글 제목: third   ← 클릭하면 /articles/3/ 으로 이동!
────────────────────────────
```

> 💡 목록에선 제목만 보여주고 클릭하면 상세페이지로 이동하는 패턴!

---

## 📌 HTTP Request Methods

> 서버에게 **"나 이거 하고 싶어"** 라고 알려주는 것

### GET

- 서버로부터 데이터를 **조회**할 때 사용
- **DB 변경 없음** (읽기 전용)
- 주요 사용: 검색 쿼리, 웹 페이지 요청, API 데이터 조회 (TMDB, 날씨 API 등)

| 특징 | 내용 |
|------|------|
| 데이터 위치 | URL 쿼리 문자열 (노출됨) |
| 데이터 제한 | URL 길이 제한 있음 |
| 브라우저 히스토리 | 남음 |
| 캐싱 | 가능 |

```
/articles/create/?title=제목&content=내용  ← GET은 이렇게 URL에 노출됨
```

> 💡 **캐싱이 가능한 이유?** DB를 안 건드리니까 같은 URL = 같은 결과가 보장됨!

### POST

- 서버에 데이터를 제출하여 리소스를 **변경(생성, 수정, 삭제)** 할 때 사용
- 주요 사용: 로그인 정보 제출, 파일 업로드, 새 데이터 생성, API 데이터 변경

| 특징 | 내용 |
|------|------|
| 데이터 위치 | HTTP Body (숨겨짐) |
| 데이터 제한 | 제한 없음 (더 많은 양) |
| 브라우저 히스토리 | 안 남음 |
| 캐싱 | 기본적으로 불가 |

### GET vs POST 핵심 비교

| | GET | POST |
|--|-----|------|
| 목적 | 데이터 조회 | 데이터 변경 |
| DB 변경 | ❌ | ✅ |
| 데이터 위치 | URL (노출됨) | 요청 본문 (숨겨짐) |
| 캐싱 | ✅ | ❌ |
| 예시 | index, detail | create, 로그인 |

### CRUD와 HTTP Method

| CRUD | 지금 (GET/POST만) | 나중에 (REST API) |
|------|------------------|------------------|
| Create | POST | POST |
| Read | GET | GET |
| Update | POST | PUT / PATCH |
| Delete | POST | DELETE |

> 💡 지금은 HTML `<form>` 이 GET/POST만 지원해서 POST가 수정/삭제까지 담당.
> REST API에선 역할별로 전용 메서드가 생겨 더 직관적으로 구분됨!
> - `PUT` = 전체 수정 / `PATCH` = 일부 수정 / `DELETE` = 삭제

---

## 📌 HTTP Response Status Code

> 서버가 클라이언트 요청에 대한 결과를 나타내는 **3자리 숫자**

| 코드 | 분류 | 의미 |
|------|------|------|
| 1xx | Informational | 요청 받았고 처리 중 |
| 2xx | Successful | 요청 성공 |
| 3xx | Redirection | 다른 경로로 이동 필요 |
| 4xx | Client Error | 클라이언트 잘못 (우리가 잘못 보낸 것) |
| 5xx | Server Error | 서버 잘못 (서버가 터진 것) |

> 💡 모르는 상태코드는 `mdn http status code` 로 검색하면 정확하게 나온다!

> 💡 `settings.py`의 `DEBUG = True` 일 때는 오류 상세 내용이 다 보임.
> 실제 배포 시 `DEBUG = False` 로 바꾸면 일반 사용자에게는 깔끔한 에러 페이지만 보임!

---

## 📌 CSRF (Cross-Site Request Forgery)

### 먼저 알아야 할 개념 - 쿠키와 세션

**쿠키(Cookie)**
> 로그인하면 서버가 브라우저에게 발급하는 **"나 로그인한 사람이야"** 라는 증표
> 브라우저는 이후 같은 서버에 요청할 때 이 쿠키를 **자동으로** 같이 보냄

**세션(Session)**
> 서버 쪽에서 **"이 사람 로그인 상태다"** 를 기억하는 것
> 쿠키 안에 세션 ID를 담아서 서버가 누구인지 식별

> 💡 핵심은 **"자동으로"** 보낸다는 것! 이게 CSRF 공격의 빌미가 됨

### 개념

> **사이트 간 요청 위조** - 브라우저가 쿠키를 자동으로 보내는 특성을 악용해서
> 사용자 의지와 무관하게 공격자가 의도한 요청을 특정 웹사이트에 보내게 만드는 해킹 방식

### 공격 흐름 (위조된 인감도장)

```
1. 신뢰할 수 있는 관계 (로그인)
   → 은행(bank.com) 로그인
   → 서버가 브라우저에 쿠키(세션 ID) 발급 = 나의 "인감도장"
   → 이후 bank.com 요청엔 브라우저가 이 쿠키를 자동으로 첨부

2. 악성 위임장 (악성 링크)
   → 해커가 "무료 경품 이벤트!" 미끼 링크 전송
   → 실제 내용 = "내 돈 100만원을 해커에게 송금하라" = 위조된 위임장

3. 나도 모르는 사이 (요청 전송)
   → 링크 클릭 순간 브라우저가 bank.com에 송금 요청을 자동으로 보냄
   → 브라우저는 쿠키를 "자동으로" 같이 보내버림 (인감도장이 찍힘)

4. 은행의 착각 (공격 성공)
   → 은행 입장에선 정상 인감도장이 찍혀있으니 진짜 요청인 줄 알고 송금 실행 😱
```

### Django의 CSRF 방어

```
기존 요청 형태:  요청 데이터 → 게시글 작성
변경 요청 형태:  요청 데이터 + 인증 토큰 → 게시글 작성
```

> 토큰이 없거나 다르면 Django가 **403 Forbidden**으로 막아버린다!

### 왜 POST일 때만 토큰을 확인할까?

> **DB에 조작을 가하는 요청은 반드시 인증 수단이 필요!**
> - GET 위조 → 데이터 조회만 되고 DB 안 바뀜 → 큰 피해 없음
> - POST 위조 → 내 의지와 무관하게 생성/수정/삭제 발생 → 반드시 막아야 함

### 사용법

```html
<form action="{% url 'articles:create' %}" method="POST">
  {% csrf_token %}  <!-- 반드시 form 안에 포함! -->
  ...
</form>
```

실제로 렌더링되면 `type="hidden"` 으로 토큰값이 숨겨져서 전송된다:
```html
<input type="hidden" name="csrfmiddlewaretoken" value="4RLxcFtlo8k2b...">
```

---

## 📌 CREATE 구현

> create 로직 구현에는 **2개의 view 함수**가 필요!

```
[new]                          [create]
사용자에게 입력 폼 보여줌   →   입력한 데이터 받아서 DB에 저장
(throw - 던지는 쪽)             (catch - 받는 쪽)
```

---

### ① urls.py - new 경로 등록

```python
path('new/', views.new, name='new'),
```

---

### ② views.py - new 함수

```python
def new(request):
    return render(request, 'articles/new.html')  # 빈 폼 페이지만 보여줌
```

---

### ③ new.html - 입력 폼 화면

```html
{% extends 'base.html' %}
{% block content %}
<h1>NEW</h1>
<form action="{% url 'articles:create' %}" method="POST">
<!--           ↑ 제출하면 create로 데이터 던짐! (throw)  -->
  {% csrf_token %}
  <div>
    <label for="title">Title: </label>
    <input type="text" name="title" id="title">
    <!--               ↑ views.py에서 request.POST.get('title') 로 꺼낼 때 이 name 사용 -->
  </div>
  <div>
    <label for="content">Content: </label>
    <textarea name="content" id="content"></textarea>
    <!--       ↑ views.py에서 request.POST.get('content') 로 꺼낼 때 이 name 사용 -->
  </div>
  <input type="submit">
</form>
<hr>
<a href="{% url 'articles:index' %}">[back]</a>
{% endblock content %}
```

**📺 브라우저 화면 결과 (`127.0.0.1:8000/articles/new/`)**
```
네비게이션 바   NEW
────────────────────────────
NEW

Title:   [ 입력창        ]
Content: [ 입력창        ]
         [    제출       ]
────────────────────────────
[back]
```

> 💡 제출 버튼을 누르면 `action="{% url 'articles:create' %}"` 에 의해
> `/articles/create/` 로 POST 요청이 전송됨!

---

### ④ urls.py - create 경로 등록

```python
path('create/', views.create, name='create'),
```

---

### ⑤ views.py - create 함수

```python
def create(request):
    # new.html에서 사용자가 입력하고 제출한 데이터를 꺼내옴
    title = request.POST.get('title')
    content = request.POST.get('content')
    article = Article(title=title, content=content)  # 객체 1개 생성
    article.save()                                   # DB에 저장
    return redirect('articles:detail', article.pk)  # 저장된 글 상세페이지로 이동
```

```sql
-- article.save()
INSERT INTO articles_article (title, content) VALUES ('제목', '내용');
-- 삽입해라  articles_article 테이블에  title 컬럼엔 '제목',  content 컬럼엔 '내용'
```

> ⚠️ `request.GET.get()` 으로 받으면 빈값(None)이 와서 **NOT NULL 에러** 발생!
> form의 `method="POST"` 면 반드시 `request.POST.get()` 으로 받아야 한다.

---

### ⑥ 결과 - detail 페이지로 redirect

```
redirect('articles:detail', article.pk)
        ↓
/articles/2/ 로 GET 요청 자동 전송
        ↓
detail 함수 실행 → detail.html 렌더링
```

**📺 브라우저 화면 결과 (`127.0.0.1:8000/articles/2/`)**
```
네비게이션 바   NEW
────────────────────────────
DETAIL

2 번째 글
────────────────────────────
제목: second
내용: django!
작성일: 2026-04-15 05:55:06
수정일: 2026-04-15 05:55:06
────────────────────────────
[back]
```

---

## 📌 redirect

### 문제 상황 - render로 응답할 때 발생하는 중복 저장

> ❌ **잘못된 방식 - `render` 로 응답**

```python
def create(request):
    ...
    article.save()
    return render(request, 'articles/create.html')  # ← 이게 문제!
```

```
POST로 글 작성 → DB 저장 → render로 "게시글이 작성되었습니다" 페이지 직접 반환
                                              ↓
                        새로고침 누르면? → 마지막 POST 요청 재전송 → DB에 또 저장 😱
```

> `render` 는 POST 요청에 그대로 응답하기 때문에
> 새로고침 시 브라우저가 "이전 POST 요청을 다시 보낼까요?" 하고 재전송해버림
> → **같은 글이 DB에 계속 쌓이는 문제 발생!**

### redirect 개념

> 서버가 클라이언트를 **직접** 다른 페이지로 보내는 게 아니다!
> "이 URL로 GET 요청 한 번 더 보내" 라고 응답하는 것

```
① 클라이언트 → Django : [POST] 게시글 작성 요청 + 입력 데이터
② Django : create view 함수 호출 (DB 저장)
③ Django → 클라이언트 : "detail 주소로 요청 보내라" (redirect 응답)
④ 클라이언트 → Django : [GET] detail 페이지 요청
⑤ Django : detail view 함수 호출
⑥ Django → 클라이언트 : detail 페이지 응답
```

> 요청이 총 **2번** 일어나지만 사용자 눈엔 자연스럽게 이동하는 것처럼 보임!

### 사용법

```python
from django.shortcuts import render, redirect  # redirect import 추가

return redirect('articles:detail', article.pk)
# 클라이언트가 articles:detail URL로 GET 요청을 다시 보내도록 응답
```

---

## 📌 DELETE 구현

> 삭제 = DB 변경 → **POST** 방식!

### urls.py

```python
path('delete/<int:article_id>/', views.delete, name='delete'),
```

### views.py

```python
def delete(request, article_id):
    article = Article.objects.get(pk=article_id)
    article.delete()
    return redirect('articles:index')  # 삭제 후엔 해당 글이 없으니 목록으로
```

```sql
-- article.delete()
DELETE FROM articles_article WHERE id = article_id;
-- 삭제해라  articles_article 테이블에서  id가 article_id인 행을
```

### detail.html - DELETE 버튼 추가

```html
<form action="{% url 'articles:delete' article.pk %}" method="POST">
  {% csrf_token %}
  <input type="submit" value="게시글 삭제">
</form>
<a href="{% url 'articles:edit' article.pk %}">게시글 수정</a>
<a href="{% url 'articles:index' %}">[back]</a>
```

> 💡 삭제는 DB 변경이라 `<form method="POST">` 사용
> 수정 링크는 단순 페이지 이동(GET)이라 `<a>` 태그로 충분!

---

## 📌 UPDATE 구현

> update와 create는 한 끗 차이!
> - **create** = `new`(빈 폼) + `create`(DB 저장)
> - **update** = `edit`(기존 데이터 채워진 폼) + `update`(DB 수정)

---

### ① urls.py - edit 경로 등록

```python
path('edit/<int:article_id>/', views.edit, name='edit'),
```

---

### ② views.py - edit 함수

```python
def edit(request, article_id):
    article = Article.objects.get(pk=article_id)
    # 수정 페이지니까 기존 데이터를 먼저 DB에서 꺼내와야
    # edit.html 입력창에 기존 내용을 채워서 보여줄 수 있음
    context = {
        'article': article,
    }
    return render(request, 'articles/edit.html', context)
```

---

### ③ edit.html - 기존 데이터가 채워진 수정 폼

```html
{% extends 'base.html' %}
{% block content %}
<h1>EDIT</h1>
<form action="{% url 'articles:update' article.pk %}" method="POST">
<!--           ↑ 제출하면 update로 데이터 던짐!  -->
  {% csrf_token %}
  <div>
    <label for="title">Title: </label>
    <input type="text" name="title" id="title" value="{{ article.title }}">
    <!--                                               ↑ 기존 제목이 입력창에 채워짐 -->
  </div>
  <div>
    <label for="content">Content: </label>
    <textarea name="content" id="content">{{ article.content }}</textarea>
    <!--                                   ↑ 기존 내용이 입력창에 채워짐 -->
  </div>
  <input type="submit">
</form>
<hr>
<a href="{% url 'articles:index' %}">[back]</a>
{% endblock content %}
```

**📺 브라우저 화면 결과 (`127.0.0.1:8000/articles/2/edit/`)**
```
네비게이션 바   NEW
────────────────────────────
EDIT

Title:   [ second        ]  ← 기존 제목이 채워져 있음
Content: [ django!       ]  ← 기존 내용이 채워져 있음
         [    제출       ]
────────────────────────────
[back]
```

> 💡 `action="{% url 'articles:update' article.pk %}"` 때문에
> 제출 버튼을 누르면 `/articles/2/update/` 로 POST 요청이 전송됨!

---

### ④ urls.py - update 경로 등록

```python
path('update/<int:article_id>/', views.update, name='update'),
```

---

### ⑤ views.py - update 함수

```python
def update(request, article_id):
    article = Article.objects.get(pk=article_id)
    # 수정할 글을 DB에서 꺼내온 뒤 새로운 값으로 덮어씌움
    article.title = request.POST.get('title')
    article.content = request.POST.get('content')
    article.save()  # UPDATE 쿼리 실행
    return redirect('articles:detail', article_id)  # 수정된 글 상세페이지로 이동
```

```sql
-- article.save() (pk가 이미 있으므로 INSERT가 아닌 UPDATE 실행)
UPDATE articles_article
SET title = '새제목', content = '새내용'
WHERE id = article_id;
-- 수정해라  articles_article 테이블에서  title, content를  id가 article_id인 행을
```

---

## 📌 CRUD 최종 URL 구조

| 기능 | URL | Method | View 함수 |
|------|-----|--------|-----------|
| 목록 조회 | `articles/` | GET | `index` |
| 상세 조회 | `articles/<int:article_id>/` | GET | `detail` |
| 작성 폼 | `articles/new/` | GET | `new` |
| 작성 저장 | `articles/create/` | POST | `create` |
| 수정 폼 | `articles/edit/<int:article_id>/` | GET | `edit` |
| 수정 저장 | `articles/update/<int:article_id>/` | POST | `update` |
| 삭제 | `articles/delete/<int:article_id>/` | POST | `delete` |

> 💡 **URL이 같아도 사용되는 메서드에 따라 서버의 동작이 달라진다!**
> 이게 나중에 배울 **REST API** 설계의 핵심 개념이다.

---

## ☁️ DevOps 연결 포인트

| Django 개념 | DevOps 연결 |
|-------------|-------------|
| `DEBUG = True/False` | 배포 환경 분리 (개발/운영 설정 다르게) |
| POST/GET 구분 | REST API 설계 원칙의 기초 |
| redirect 패턴 | PRG(Post/Redirect/Get) 패턴 - 웹 표준 |
| CSRF 토큰 | 보안 미들웨어 개념, API Gateway 인증과 연결됨 |