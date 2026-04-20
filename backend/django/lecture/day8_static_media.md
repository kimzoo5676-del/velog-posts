# Django Static & Media Files

## 목차

1. [Static Files (정적 파일)](#1-static-files-정적-파일)
2. [웹 서버와 정적 파일](#2-웹-서버와-정적-파일)
3. [로컬 환경 vs 웹 서버 환경](#3-로컬-환경-vs-웹-서버-환경)
4. [정적 파일과 URL의 관계](#4-정적-파일과-url의-관계)
5. [정적 파일 처리 과정 요약](#5-정적-파일-처리-과정-요약)
6. [Static Files 경로의 종류](#6-static-files-경로의-종류)
7. [기본 경로 CSS / 이미지 제공하기](#7-기본-경로-css--이미지-제공하기)
8. [load & static 태그 이론 정리](#8-load--static-태그-이론-정리)
9. [STATIC_URL](#9-static_url)
10. [추가 경로 — STATICFILES_DIRS](#10-추가-경로--staticfiles_dirs)
11. [Media Files (미디어 파일)](#11-media-files-미디어-파일)
12. [MEDIA_ROOT & MEDIA_URL](#12-media_root--media_url)
13. [urls.py — Media URL 설정](#13-urlspy--media-url-설정)
14. [ImageField](#14-imagefield)
15. [이미지 업로드 구현](#15-이미지-업로드-구현)
16. [업로드 이미지 제공하기](#16-업로드-이미지-제공하기)
17. [upload_to 심화 활용](#17-upload_to-심화-활용)
18. [AWS 인프라 이해하기](#18-aws-인프라-이해하기)

---

## 1. Static Files (정적 파일)

### 정의

**누가 요청하든** 서버에서 항상 **동일하게 제공**되는 고정 파일

> 💡 **비유 — 식당 메뉴판**
> 식당에 인쇄된 메뉴판은 손님이 바뀌어도 내용이 변하지 않는다.
> 정적 파일도 마찬가지로 **어떤 사용자가 요청해도 똑같은 파일**을 그대로 돌려준다.

### 대표적인 종류

| 파일 종류 | 역할 |
|----------|------|
| CSS | 페이지 스타일링 |
| JavaScript | 동적 UI 동작 |
| 이미지 | 로고, 아이콘, 배경 등 |
| 폰트 | 웹 폰트 (.woff, .ttf 등) |

### 반대 개념 — 동적 응답과 비교

```
정적 파일 → /static/style.css  → 항상 같은 내용
동적 응답 → /articles/1/       → 사용자/DB 상태에 따라 내용이 달라짐
```

---

## 2. 웹 서버와 정적 파일

### 웹 서버의 기본 역할

**요청받은 URL에 해당하는 자원을 찾아 응답**해주는 것

> 💡 **비유 — 도서관 사서**
> "A-3구역의 3번째 책을 주세요"라고 요청하면
> 사서가 해당 위치로 가서 책을 찾아 건네주는 것과 같다.
>
> | 비유 | 실제 웹 |
> |------|--------|
> | A-3구역의 3번째 (위치) | URL |
> | 책 | 자원 (파일, 데이터 등) |
> | 사서 | 웹 서버 |

### 정적 파일 응답 흐름

```
브라우저                  웹 서버
   |                        |
   |  GET /static/style.css |
   | ─────────────────────>   |
   |                        |  → 해당 경로에서 파일 탐색
   |      style.css 반환     |
   | <─────────────────────   |
```

---

## 3. 로컬 환경 vs 웹 서버 환경

### 기존 방식 (로컬에서 직접 열기)

```
HTML 파일을 직접 더블클릭
브라우저가 파일 시스템에서 바로 읽어옴

주소창: file:///C:/Users/.../index.html
```

- HTTP 요청/응답 없음
- 내 컴퓨터 파일을 직접 읽는 것

### 현재 방식 (웹 서버 환경)

```
Client                        Server
   |                             |
   |  HTTP Request (GET /...)    |
   | ─────────────────────────>    |
   |                             |  → 요청에 맞는 자원 탐색
   |  HTTP Response (200 OK)     |
   | <─────────────────────────    |
```

주소창: `http://127.0.0.1:8000/...`

> ✅ **핵심 포인트**
> 로컬에서 직접 열 땐 경로만 맞으면 됐지만,
> 웹 서버 환경에서는 **URL과 서버 설정이 맞아야** 파일을 제공할 수 있다.
> 그래서 Django에서 Static 파일 **설정**이 필요한 것!

---

## 4. 정적 파일과 URL의 관계

### 웹 서버의 기본 자원 = 정적 파일

웹 서버가 제공하는 **가장 기본적인 자원**이 바로 정적 파일이다.

### URL의 필요성

> 정적 파일이 사용자에게 **보이려면**
> 그 파일에 접근할 수 있는 **고유한 주소(URL)가 반드시 필요**하다.

```
❌ 파일이 서버에 존재해도 URL이 없으면 → 사용자가 접근 불가
✅ 파일 + URL 매핑이 되어야 → 브라우저에서 접근 가능
```

실제 예시:

```
서버 파일 경로: /static/css/style.css
접근 URL:      http://127.0.0.1:8000/static/css/style.css
```

> ✅ **핵심 포인트**
> 파일이 **존재**하는 것과 파일에 **접근 가능**한 것은 다르다.
> URL이 있어야 비로소 사용자에게 제공될 수 있다.
> → 이게 바로 Django에서 `STATIC_URL`을 설정하는 이유!

---

## 5. 정적 파일 처리 과정 요약

```
① 사용자
   브라우저에 http://example.com/images/logo.png 입력
   → 이미지 파일 요청 (HTTP Request)
            │
            ▼
② 웹 서버
   URL에서 /images/logo.png 경로 확인
   → 서버의 미리 약속된 폴더에서 logo.png 탐색
            │
            ▼
③ 웹 서버
   찾은 logo.png를 HTTP 응답에 담아 전송 (HTTP Response)
            │
            ▼
④ 사용자
   브라우저가 응답받은 이미지를 화면에 렌더링
```

> ✅ **"미리 약속된 폴더"가 핵심!**
> 웹 서버는 모든 폴더를 뒤지는 게 아니라
> **설정으로 지정된 특정 폴더**에서만 파일을 찾는다.
> → Django에서는 이게 바로 `STATICFILES_DIRS`와 `STATIC_ROOT` 설정!

---

## 6. Static Files 경로의 종류

Django가 정적 파일을 탐색하는 경로는 두 가지로 나뉜다.

| 종류 | 경로 | 설명 |
|------|------|------|
| 기본 경로 | `app폴더/static/` | 설정 없이 자동 인식 |
| 추가 경로 | `STATICFILES_DIRS`에 문자열로 설정 | 공통 파일 관리용 |

### ⚠️ 중요 — "파일 탐색"과 "파일 연결"은 다르다!

헷갈리기 쉬운 두 개념을 반드시 구분해야 한다.

| 개념 | 대상 | 설명 |
|------|------|------|
| **파일 탐색** | `app/static/` 폴더 존재 | Django가 "어디서 파일을 찾을지" |
| **파일 연결** | `{% load static %}` + `{% static %}` | HTML 템플릿에서 "그 파일을 어떻게 불러올지" |

> 💡 **비유**
> `app/static/` 폴더 = 창고에 물건을 갖다 놓은 것
> `{% static %}` 태그 = 창고에서 물건을 꺼내서 진열대에 올려놓는 것
>
> 창고에 물건이 있어도 (파일 존재)
> 진열대에 올리지 않으면 (템플릿에서 연결 안 하면)
> 손님은 볼 수 없다!

```
① app/static/ 에 파일 배치        ← Django가 파일을 찾을 수 있음
② 템플릿에서 {% static %} 로 연결  ← 브라우저가 파일을 받을 수 있음
```

**둘 다 있어야 CSS/이미지가 정상 적용된다!**

### 기본 경로

```
앱이름/
  └── static/        ← 여기에 정적 파일 넣으면 됨
        ├── css/
        │     └── style.css
        ├── js/
        │     └── script.js
        └── images/
              └── sample-1.png
```

> ✅ `app폴더/static/` 경로는 Django가 **설정 없이도 자동으로 인식**한다.
> 별도 설정 없이 이 폴더에 파일을 넣기만 하면 Django가 찾아준다.

---

## 7. 기본 경로 CSS / 이미지 제공하기

### Step 1 — 파일 배치

```
articles/
  └── static/
        ├── stylesheets/
        │     └── style.css    ← CSS 파일
        └── images/
              └── sample-1.png ← 이미지 파일
```

```css
/* style.css */
h1 {
    color: crimson;
}
```

### Step 2 — 템플릿에서 불러오기

```html
{% extends 'base.html' %}
{% load static %}   ← 반드시 extends 다음에!

<head>
    <link rel="stylesheet" href="{% static 'stylesheets/style.css' %}">
</head>
<body>
    <h1>글 목록</h1>
    <img src="{% static 'images/sample-1.png' %}" alt="sample image">
</body>
```

### Step 3 — 결과 확인 (개발자도구)

```html
<!-- Django가 변환한 실제 HTML -->
<link rel="stylesheet" href="/static/stylesheets/style.css">
<img src="/static/images/sample-1.png">
```

> ✅ `{% static %}` 태그가 `STATIC_URL + 파일경로` 를 합쳐서 URL을 자동 생성해준다.

> 💡 **`STATIC_URL`은 따로 설정 안 해도 되나?**
> Django가 프로젝트 생성 시 `settings.py` 하단에 **자동으로 작성**해준다.
> ```python
> # settings.py — 프로젝트 생성 시 자동으로 존재
> STATIC_URL = 'static/'
> ```
> 반면 `MEDIA_URL`, `MEDIA_ROOT`는 기본값이 없어서 **개발자가 직접 추가**해야 한다!
>
> | 설정 | 기본값 제공 | 직접 작성 필요? |
> |------|-----------|--------------|
> | `STATIC_URL` | ✅ `'static/'` | ❌ 이미 있음 |
> | `STATICFILES_DIRS` | ❌ | ✅ 추가 경로 쓸 때만 |
> | `MEDIA_URL` | ❌ | ✅ 필수 |
> | `MEDIA_ROOT` | ❌ | ✅ 필수 |

### ⚠️ 주의 — href 경로 방식 비교

```html
<!-- ❌ 잘못된 방법 — 상대경로 -->
<link href="stylesheets/style.css">
→ 브라우저: http://127.0.0.1:8000/articles/stylesheets/style.css (없는 URL!)

<!-- ❌ 슬래시만 붙여도 안 됨 -->
<link href="/stylesheets/style.css">
→ 브라우저: http://127.0.0.1:8000/stylesheets/style.css (static 폴더 못 찾음!)

<!-- ✅ 올바른 방법 -->
<link href="{% static 'stylesheets/style.css' %}">
→ 브라우저: http://127.0.0.1:8000/static/stylesheets/style.css ✅
```

### CSS vs 이미지 태그 비교

| 파일 종류 | 태그 | 경로 속성 |
|----------|------|---------|
| CSS | `<link>` | `href` |
| 이미지 | `<img>` | `src` |
| JS | `<script>` | `src` |

---

## 8. load & static 태그 이론 정리

### `{% load %}`

특정 라이브러리의 **태그와 필터**를 현재 템플릿에서 사용할 수 있도록 불러오는 역할

```html
{% load static %}
```

Django 템플릿 시스템에게 **"이제부터 static 관련 태그를 사용하겠다"** 고 알려주는 **선언문**

> `static` 태그는 **built-in 태그가 아니기 때문에** `{% load static %}` 으로 import 후 사용 가능하다.

| 태그 | 종류 | load 필요? |
|------|------|-----------|
| `{% if %}` | built-in | ❌ |
| `{% for %}` | built-in | ❌ |
| `{% url %}` | built-in | ❌ |
| `{% static %}` | 외부 태그 | ✅ `{% load static %}` 필요 |

> ⚠️ `{% load static %}` 빠뜨리면 `TemplateSyntaxError` 발생!

### `{% static %}`

`settings.py`의 **`STATIC_URL` 값을 기준**으로
해당 정적 파일의 **전체 URL 경로를 계산하여 생성**

```
{% static 'stylesheets/style.css' %}
        ↓ Django가 변환
/static/stylesheets/style.css
  ↑
  settings.py의 STATIC_URL = 'static/' 이 자동으로 앞에 붙음
```

### 태그 사용 순서

```html
{% extends 'base.html' %}  ← 반드시 1번째
{% load static %}          ← 반드시 2번째
```

> `{% extends %}` 가 최상단에 없으면 Django 템플릿 엔진이 에러를 발생시킨다!

---

## 9. STATIC_URL

### 정의

정적 파일의 **웹 주소 접두사(prefix)**
→ 브라우저에서 정적 파일에 접근할 때 사용할 URL의 **시작 부분**을 지정하는 설정

```python
# settings.py
STATIC_URL = 'static/'
```

### 핵심 — 실제 파일 경로 ≠ 웹 주소

```
실제 파일 경로  →  C:/Users/ssafy/project/articles/static/style.css
웹 주소 (URL)  →  http://127.0.0.1:8000/static/style.css
                                         ↑
                                    STATIC_URL = 'static/'
```

> 서버에 저장된 **실제 경로가 아니라**
> 오직 **브라우저(웹)에서만 사용되는 주소**다.

> 💡 **비유**
> 서버의 정적 파일 폴더에 **`static/` 이라는 웹 주소 별명**을 붙여주는 것.
> 브라우저가 이 별명으로 요청하면 Django가 실제 폴더에서 파일을 찾아 응답한다.

### 정적 파일 URL이 만들어지기까지

```
최종 URL = 도메인 + STATIC_URL + 정적파일 경로

http://127.0.0.1:8000  /static/  images/sample-1.png
             ↑            ↑            ↑
      IP 주소 + 포트    STATIC_URL   정적파일 경로
```

---

## 10. 추가 경로 — STATICFILES_DIRS

### 왜 필요한가?

`app/static/` 기본 경로는 해당 앱 전용이다.
**여러 앱에서 공통으로 사용**하는 CSS, JS, 이미지가 있다면
프로젝트 최상단에 별도 폴더를 만들고 `STATICFILES_DIRS`에 등록한다.

> 💡 템플릿의 `DIRS` 설정과 완전히 같은 개념!

### 설정 방법

```python
# settings.py
STATICFILES_DIRS = [
    BASE_DIR / 'static',
]
```

> ✅ 반드시 **대문자**로 작성! (`staticfiles_dirs` ❌)

### 폴더 구조

```
프로젝트/
  ├── articles/
  │     └── static/       ← 앱 전용 (기본 경로)
  ├── static/             ← 공통 파일 (추가 경로) ← 여기!
  ├── templates/
  └── manage.py
```

### 템플릿 설정과 비교

| 구분 | 템플릿 | Static 파일 |
|------|--------|------------|
| 앱 전용 | `앱/templates/` | `앱/static/` |
| 공통 | `DIRS = [BASE_DIR / 'templates']` | `STATICFILES_DIRS = [BASE_DIR / 'static']` |

> STATICFILES_DIRS → 리스트 (여러 폴더 탐색 가능)
> MEDIA_ROOT      → 단일 경로 (자세한 설명은 12섹션에서!)

---

## 11. Media Files (미디어 파일)

### 정의

**사용자가 웹사이트를 통해 직접 업로드**하는 파일

### Static vs Media 최종 비교

| 구분 | Static Files | Media Files |
|------|-------------|-------------|
| 누가 올리나 | 개발자가 미리 준비 | 사용자가 직접 업로드 |
| 변경 시점 | 배포 전 고정 | 서비스 운영 중 동적 생성 |
| 예시 | 로고, CSS, JS | 프로필 사진, 상품 후기 사진 |
| 출력 방법 | `{% static '경로' %}` | `{{ article.image.url }}` |

> 💡 **비유 — 쇼핑몰**
> 쇼핑몰 기본 로고/아이콘 → 개발자가 미리 만들어 둔 것 = **Static**
> 사용자가 상품 후기에 올리는 사진 → 운영 중 사용자가 올리는 것 = **Media**

---

## 12. MEDIA_ROOT & MEDIA_URL

### MEDIA_ROOT

사용자가 업로드한 미디어 파일들이 **서버 컴퓨터 어디에 저장될지** 지정하는 **절대 경로**

```python
# settings.py
MEDIA_ROOT = BASE_DIR / 'media'
```

- 서버 **내부에서만** 사용하는 물리적 폴더 주소
- Django가 파일을 **저장하거나 읽어올 때** 이 경로를 사용
- 브라우저는 이 경로를 직접 알 수 없음 (내부용!)
- 리스트가 아닌 **단일 경로** → 창고가 두 개면 어디 있는지 헷갈리기 때문

> 💡 **비유** : 실제 창고의 물리적 위치 — 인터넷 주소가 아니라 서버 안의 실제 폴더 경로

### MEDIA_URL

`MEDIA_ROOT`에 저장된 파일들을 웹 페이지에서 접근할 때 사용할 **URL의 시작 부분**

```python
# settings.py
MEDIA_URL = 'media/'
```

- `STATIC_URL`과 동일한 역할
- 실제 저장 위치를 숨기고 웹에서 사용할 **공개 주소 별명**을 만들어 주는 것

### 최종 URL 구성

```
http://127.0.0.1:8000/media/images/photo.png
                       ↑      ↑
                  MEDIA_URL  upload_to + 파일명
```

### Static vs Media 설정 전체 비교

| 설정 | 역할 |
|------|------|
| `STATIC_URL` | static 파일 웹 주소 접두사 |
| `STATICFILES_DIRS` | static 파일 탐색 경로 (리스트) |
| `MEDIA_URL` | media 파일 웹 주소 접두사 |
| `MEDIA_ROOT` | media 파일 저장 경로 (단일) |

---

## 13. urls.py — Media URL 설정

Static과 달리 Media는 **`urls.py`에 직접 URL 연결을 추가**해줘야 한다.

### 왜 직접 추가해야 하나?

Django 개발 서버는 기본적으로 파이썬 코드를 실행하는 **애플리케이션 서버**다.
사용자가 올린 이미지 파일 같은 미디어 파일을 어떻게 찾아 보여줘야 하는지 **스스로 알지 못한다.**
따라서 개발자가 직접 URL 경로를 만들어 "이 주소로 요청이 오면 이 폴더에서 파일을 찾아 보여줘"라는 규칙을 추가해야 한다.

### 설정 코드

```python
# config/urls.py
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('articles/', include('articles.urls')),
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
#   ↑ settings.MEDIA_URL: 'media/'로 시작하는 URL 요청이 오면
#   ↑ document_root=settings.MEDIA_ROOT: MEDIA_ROOT 폴더에서 파일을 찾아라
```

### `+ static(...)` 의 정체

`static()` 함수의 **리턴값이 리스트**이기 때문에 `urlpatterns` 리스트에 `+`로 합칠 수 있다.

```python
# static() 함수가 실제로 반환하는 것
[
    path('media/<path:path>', serve, {'document_root': MEDIA_ROOT}),
]
```

> ✅ **개발 환경 전용!**
> `+ static(...)` 은 **개발 서버에서만** 동작한다.
> 실제 배포 환경에서는 Nginx 같은 웹 서버가 이 역할을 대신한다.

---

## 14. ImageField

### 정의

이미지 파일을 업로드하기 위해 사용하는 **Django 모델 필드**

### 가장 중요한 특징 — DB에는 파일이 저장되지 않는다!

```
사용자가 이미지 업로드
        ↓
실제 파일 → 서버 폴더 (MEDIA_ROOT) 에 저장
DB      → 파일 경로(문자열)만 저장
```

```
실제 파일 위치: /media/images/sample.png   ← 서버 폴더
DB에 저장된 값: 'images/sample.png'        ← 문자열만!
```

### 왜 파일을 DB에 직접 안 저장하나?

```
이미지 파일 = 수 MB ~ 수백 MB
DB는 대용량 파일 저장에 부적합 → 느려짐 😱
→ 파일은 서버 폴더에, 경로만 DB에! ✅
```

### 코드

```python
# models.py
class Article(models.Model):
    title = models.CharField(max_length=10)
    content = models.TextField()
    # 이미지는 'MEDIA_ROOT/images/' 경로에 저장되고,
    # DB에는 'images/sample.png'와 같은 경로 문자열이 저장됨
    image = models.ImageField(upload_to='images/', blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

### `upload_to` 옵션

| 옵션 | 설명 |
|------|------|
| `upload_to='images/'` | MEDIA_ROOT 아래 images/ 폴더에 저장 |
| 선택 인자 | 없으면 MEDIA_ROOT 바로 아래에 저장 |

### `blank=True` 란?

**폼 유효성 검사 시** 해당 필드를 **선택 사항**으로 만드는 속성
→ 게시글 작성 시 **이미지 첨부 없이도** 작성 가능하도록!

| 옵션 | 적용 대상 | 의미 |
|------|---------|------|
| `blank=True` | **폼 유효성 검사** | 빈 문자열 허용 (입력 안 해도 됨) |
| `null=True` | **데이터베이스** | DB에 NULL 저장 허용 |

> ✅ `ImageField`에는 보통 `blank=True`만 사용 (문자열 기반 필드는 null 대신 빈 문자열 권장)

### Pillow 설치 필수!

`ImageField`는 내부적으로 **Pillow** 라이브러리에 의존한다.
Pillow 없이 `ImageField`를 사용하면:

```
fields.E210: Cannot use ImageField because Pillow is not installed.
```

```bash
pip install Pillow
python manage.py makemigrations
python manage.py migrate
pip freeze > requirements.txt   # 반드시 세트로!
```

> ✅ 패키지 설치 후 `pip freeze > requirements.txt` 는 **항상 한 세트**로 기억하기!
> 팀 프로젝트에서 다른 팀원이 `pip install -r requirements.txt` 할 때 동일한 환경을 보장한다.

---

## 15. 이미지 업로드 구현

### 필요한 3가지 수정

이미지 업로드가 동작하려면 아래 **3곳을 모두** 수정해야 한다.

#### 1. `create.html` / `update.html` — enctype 추가

```html
<!-- ❌ 파일 전송 안 됨 -->
<form method="POST">

<!-- ✅ 파일 전송 가능 -->
<form action="{% url 'articles:create or update' %}" method="POST" enctype="multipart/form-data">
```

`enctype="multipart/form-data"` 없으면 파일 데이터가 **서버로 아예 전송되지 않는다!**
> `enctype` 속성 : form 데이터가 서버로 제출될 떄, 해당 데이터가 어떤 형식으로 인코딩 될지 지정하는 속성

#### 2. `views.py` — request.FILES 추가

```python
# create
def create(request):
    if request.method == 'POST':
        form = ArticleForm(request.POST, request.FILES)  # FILES 추가!
        ...

# update
def update(request, article_id):
    article = Article.objects.get(pk=article_id)
    if request.method == 'POST':
        form = ArticleForm(request.POST, request.FILES, instance=article)  # FILES 추가!
        ...
```

> 💡 **왜 `request.FILES`가 따로 있나?**
> `enctype="multipart/form-data"`로 전송하면 데이터가 두 군데로 나뉘어 온다.
>
> | 데이터 | 담기는 곳 |
> |--------|---------|
> | 텍스트 (title, content) | `request.POST` |
> | 파일 (image) | `request.FILES` |

#### 3. 서버 로그로 확인

```python
# views.py - 테스트용
print(request.FILES)
# → <MultiValueDict: {'image': [<InMemoryUploadedFile: dog.jpg (image/jpeg)>]}>
```

### 이미지 업로드 완성 체크리스트

```
✅ models.py     ImageField(upload_to='images/', blank=True)
✅ settings.py   MEDIA_ROOT, MEDIA_URL 설정
✅ urls.py       + static(MEDIA_URL, document_root=MEDIA_ROOT)
✅ create.html   enctype="multipart/form-data"
✅ views.py      ArticleForm(request.POST, request.FILES)
```

> ✅ **흐름 요약**
> ```
> 보내는 곳  →  create.html (enctype)
>                    ↓
> 받는 곳    →  views.py (request.FILES)
>                    ↓
> 저장하는 곳 →  settings.py (MEDIA_ROOT)
>                    ↓
> DB         →  image 컬럼에 경로 문자열 저장
> ```

---

## 16. 업로드 이미지 제공하기

### `ImageField.url` 속성

`ImageField`나 `FileField`에 저장된 파일 객체에서 `.url` 속성을 사용하면
해당 파일을 **웹에서 접근할 수 있는 전체 URL 주소**를 얻을 수 있다.

```python
article.image      # → 'images/dog.jpg'     (DB에 저장된 경로 문자열)
article.image.url  # → '/media/images/dog.jpg' (실제 접근 가능한 URL)
```

`.url`이 하는 일:

```
MEDIA_URL + DB에 저장된 경로를 자동으로 합쳐줌
'media/'  +  'images/dog.jpg'  =  '/media/images/dog.jpg'
```

### Static vs Media 출력 방식 비교

```html
<!-- Static 파일 → {% static %} 태그 사용 -->
<img src="{% static 'images/sample.png' %}">

<!-- Media 파일 → .url 속성 사용 -->
<img src="{{ article.image.url }}">
```

> Static은 개발자가 경로를 미리 알고 있어서 `{% static %}` 태그를 쓰지만,
> Media는 경로가 DB에 저장되어 있으므로 `.url` 속성으로 접근한다.

### detail.html — 완성형 패턴

```html
{% load static %}

{% if article.image %}
    <!-- 업로드한 이미지가 있으면 → media 파일 출력 -->
    <img src="{{ article.image.url }}" alt="{{ article.image }}">
{% else %}
    <!-- 이미지 없으면 → 기본 이미지(static) 출력 -->
    <img src="{% static 'images/default.png' %}" alt="default image">
{% endif %}
```

### `alt` 속성의 역할

**Alternative text** 의 약자로, 이미지를 **대체하는 텍스트**

| 역할 | 상황 | 설명 |
|------|------|------|
| **대체 텍스트** | 이미지 로드 실패 시 | 이미지 대신 텍스트로 표시 |
| **접근성** | 시각장애인 스크린 리더 | "이 이미지는 ~입니다" 읽어줌 |
| **SEO** | 검색 엔진 | 이미지 내용을 검색엔진이 파악 |

```html
<!-- Media 파일 -->
<img src="{{ article.image.url }}" alt="{{ article.image }}">
<!--                                    ↑
                            'images/dog.jpg' 같은 파일명이 들어감 -->

<!-- Static 기본 이미지 -->
<img src="{% static 'images/default.png' %}" alt="default image">
<!--                                          ↑
                                        고정 문자열 -->
```

> ✅ 둘 다 문법적으로 문제없고 이미지 못 불러올 때 **뭘 보여줄지**의 차이만 있다.

> ✅ **왜 `if` 가 필요하냐?**
> `blank=True`로 이미지 없이도 게시글 작성이 가능하기 때문에
> 이미지가 없는 게시글에서 `.url`에 접근하면 에러가 발생한다.
>
> ```
> 이미지 업로드 O → article.image.url = '/media/images/dog.jpg' ✅
> 이미지 업로드 X → article.image = '' → .url 접근 시 에러 💥
> ```

---

## 17. upload_to 심화 활용

기본 문자열 경로 외에도 업로드 경로를 **동적으로 생성**하는 두 가지 방식이 있다.

### 1. 날짜를 이용한 경로 구성

```python
image = models.ImageField(upload_to='images/%Y/%m/%d/')
```

```
업로드 날짜에 따라 자동으로 폴더 생성
media/
  └── images/
        └── 2026/
              └── 04/
                    └── 20/
                          └── cat.jpg
```

파일이 날짜별로 자동 분류되어 한 폴더에 몰리지 않는다.

### 2. 함수를 이용한 동적 경로 생성

더 복잡한 로직으로 경로를 만들고 싶을 때 `upload_to`에 **함수를 직접 전달**할 수 있다.

이 함수는 두 가지 인자를 받는다:

| 인자 | 설명 |
|------|------|
| `instance` | 파일이 첨부된 **모델의 인스턴스** (해당 게시글 객체 등) |
| `filename` | 업로드된 파일의 **원본 이름** |

```python
# 경로 생성 함수 정의
def articles_image_path(instance, filename):
    # instance.user.username으로 게시글 작성자의 이름을 가져옴
    # 예: 'images/ssafy_user/my_photo.jpg'와 같은 경로를 반환
    return f'images/{instance.user.username}/{filename}'

class Article(models.Model):
    user = ...
    image = models.ImageField(blank=True, upload_to=articles_image_path)
```

```
media/
  └── images/
        └── ssafy_user/       ← 유저명으로 폴더 자동 생성
              └── my_photo.jpg
```

### 세 가지 방식 비교

| 방식 | 예시 | 결과 경로 |
|------|------|---------|
| 고정 문자열 | `'images/'` | `media/images/cat.jpg` |
| 날짜 | `'images/%Y/%m/%d/'` | `media/images/2026/04/20/cat.jpg` |
| 함수 | `articles_image_path` | `media/images/ssafy_user/cat.jpg` |

---

## 18. AWS 인프라 이해하기

### AWS (Amazon Web Services)

아마존이 제공하는 **클라우드 컴퓨팅 플랫폼**

서버, 스토리지, 데이터베이스 같은 IT 인프라를 직접 구매하지 않고,
**인터넷을 통해 필요한 만큼 빌려 쓰는** 서비스

### AWS 핵심 서비스 3가지

| 서비스 | 역할 | 설명 |
|--------|------|------|
| **EC2** | 가상 서버 | 클라우드에 생성하는 고성능 컴퓨터 |
| **S3** | 파일 저장소 | 이미지, 동영상 등 모든 파일을 보관하는 객체 스토리지 |
| **RDS** | 데이터베이스 | 데이터를 체계적으로 관리하는 관계형 데이터베이스 |

### Django 프로젝트 + AWS 연결

| 로컬 개발 | AWS 배포 |
|----------|---------|
| `sqlite3` | **RDS** |
| `media/` 폴더 | **S3** (MEDIA_ROOT 역할) |
| 로컬 서버 | **EC2** (Django 실행) |

### 데이터 흐름도

```
① 사용자 → EC2      : HTTP Request (웹사이트 접속)
② EC2 ↔  RDS       : DB 조회/저장 (게시글, 사용자 정보)
③ EC2 → 사용자      : HTML Response (S3 파일 주소 포함)
④ 사용자 → S3       : HTML에 명시된 S3 주소로 파일 직접 요청
⑤ S3 → 사용자       : 이미지/CSS 등 파일 직접 전송
⑥ 파일 업로드 시     : 사용자 → EC2 → S3 (Django가 받아서 S3에 저장)
```

> ✅ **역할 분리**
> EC2는 **연산(로직)** 담당
> RDS는 **데이터** 담당
> S3는 **파일** 담당
> → 역할을 분리해서 각각 **독립적으로 확장** 가능!

---

## 최종 정리

```
Static Files
  → 개발자가 미리 준비한 고정 파일
  → app/static/ 또는 STATICFILES_DIRS 에 배치
  → {% static '경로' %} 태그로 출력
  → STATIC_URL 로 웹 주소 접두사 설정

Media Files
  → 사용자가 업로드하는 동적 파일
  → MEDIA_ROOT 에 실제 저장
  → MEDIA_URL 로 웹 주소 접두사 설정
  → urls.py 에 + static(...) 으로 URL 연결
  → {{ article.image.url }} 로 출력
```

---

## ☁️ DevOps 연결 포인트

| Django 개념 | DevOps 연결 |
|-------------|-------------|
| `STATIC_URL` / `MEDIA_URL` | 배포 시 CDN 주소로 교체 → 정적 파일 서빙 분리 |
| `MEDIA_ROOT` | 로컬에선 폴더, 배포에선 **AWS S3** 로 교체 (`django-storages` 사용) |
| `+ static(...)` in urls.py | 개발 환경 전용 → 배포 시 **Nginx** 가 미디어 파일 서빙 담당 |
| `ImageField` + `Pillow` | 이미지 유효성 검사 → 잘못된 파일이 서버에 저장되지 않도록 보호 |
| `pip freeze > requirements.txt` | **CI/CD 파이프라인**에서 동일한 패키지 환경 재현의 기반 |
| `request.FILES` 분리 처리 | 파일은 별도 스토리지로 → **마이크로서비스 / 오브젝트 스토리지 아키텍처** 의 기초 |
| AWS EC2 + S3 + RDS | **역할 분리 아키텍처** → 각 서비스 독립적 확장 가능, 실제 운영 환경의 표준 구성 |

> 💡 **배포 환경 핵심 변화**
> 지금은 `media/` 폴더에 이미지를 저장하지만,
> 실제 배포에서는 `django-storages` + `boto3` 라이브러리로
> S3를 `MEDIA_ROOT`처럼 사용할 수 있다.
> Nginx는 `/media/` 경로 요청을 직접 처리해서 Django 서버 부하를 줄인다.