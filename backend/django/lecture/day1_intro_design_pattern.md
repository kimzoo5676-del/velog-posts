# Django 1장 복습 🐍
> Django Q&A 복습 정리

---

## 📌 웹 서비스란?

**Q. 웹 서비스가 뭐야?**

인터넷을 통해 사용자에게 제공되는 소프트웨어. 다양한 디바이스에서 웹 브라우저만 있으면 접근 가능하다.

> **소프트웨어 그 자체**가 웹 서비스!  


---

## 📌 클라이언트 & 서버

**Q. 클라이언트와 서버가 뭐야?**

- **클라이언트**: 서비스를 요청하는 주체 (웹 브라우저, 모바일 앱)
- **서버**: 클라이언트의 요청에 응답하는 주체 (웹 서버)

```
클라이언트 → 웹 서버(Django) → 데이터베이스
               ↑                    ↓
               └────── 응답 ◀───────┘
```

> 클라이언트는 웹 서버에만 요청하고, 웹 서버가 필요하면 DB에 따로 물어보는 구조!

---

## 📌 웹 페이지를 보게 되는 과정

1. 브라우저에서 `google.com` 입력
2. **DNS 서버**에 "google.com의 실제 IP 주소가 뭐야?" 물어봄
3. IP 주소를 받아 구글 서버에 요청
4. 서버가 HTML 파일을 응답
5. 브라우저가 HTML을 해석해서 화면에 표시

> **DNS**: 도메인(google.com)을 IP 주소로 번역해주는 역할  
> **정적 페이지** → 서버 파일에서 바로 응답  
> **동적 페이지** → 서버가 DB 조회 후 HTML 생성해서 응답 ← Django가 하는 일!

---

## 📌 Frontend & Backend

| 구분 | 역할 | 기술 |
|---|---|---|
| **Frontend** | 사용자 인터페이스 구성, 상호작용 | HTML, CSS, JavaScript, Vue/React 등 |
| **Backend** | 서버 측 처리, DB 상호작용 | Python/Java, Django, DB, API, 보안 등 |

```
Frontend (HTML/CSS/JS)
      ↕ API
Backend → Django (Python 백엔드 프레임워크)
      ↕
   Database (SQLite, PostgreSQL 등)
```

---

## 📌 웹 프레임워크란?

**Q. 프레임워크가 뭐야? 라이브러리랑 뭐가 달라?**

```
라이브러리  =  벽돌, 못, 망치 (도구들) → 내가 원하는 대로 골라서 씀
프레임워크  =  이미 뼈대가 세워진 골조  → 정해진 구조 안에서 내가 살을 채워넣음
```

핵심 차이는 **제어의 주체**:
- **라이브러리** → 내가 코드 흐름을 제어
- **프레임워크** → 프레임워크가 흐름을 제어, 내가 그 안에 끼워넣음 (IoC, Inversion of Control, 제어의 역전)

**웹 프레임워크가 해주는 것:**
- HTTP 요청/응답 처리
- URL 라우팅
- DB 연결 및 쿼리
- 인증/보안
- HTML 렌더링

> **Frame + Work = 틀 안에서 일한다**
> **Routing : 최적의 경로를 선택하고 데이터를 전달하는 과정**
> **Rendering : 추상적인 데이터를 화면으로 변환하는 과정**

---

## 📌 Django를 사용하는 이유

| 특징 | 설명 |
|---|---|
| **다양성** | Python 기반으로 웹, 모바일 앱 백엔드, API, 빅데이터 등 광범위하게 활용 |
| **확장성** | 대량의 데이터에 빠르고 유연하게 확장 가능 |
| **보안** | SQL Injection, XSS, CSRF 등 보안 기능 기본 내장 |
| **커뮤니티** | 활성화된 커뮤니티, 풍부한 문서 (docs.djangoproject.com) |

> Django는 **"배터리 포함(Battery Included)"** 철학 — 필요한 게 기본으로 다 들어있음!

---

## 📌 가상환경 (Virtual Environment)

**Q. 가상환경이 왜 필요해?**

```
프로젝트 A → Django 3.2 필요
프로젝트 B → Django 4.2 필요
→ 가상환경 없으면 버전 충돌!
```

> 가상환경 = 프로젝트마다 따로 쓰는 **개인 사물함**  
> 글로벌 환경 = 모두가 공유하는 **공용 창고**

**가상환경 명령어:**

```bash
# 생성
python -m venv venv

# 활성화
source venv/Scripts/activate  # Windows
source venv/bin/activate       # Mac/Linux

# 비활성화
deactivate
```

> `-m` = module의 약자. "이 모듈을 스크립트처럼 실행해줘"  
> 활성화는 현재 터미널에만 영향 → 새 터미널 열면 다시 활성화 필요!

**venv를 .gitignore에 넣는 이유:**
- 용량이 매우 큼 (수백 MB)
- OS마다 달라서 올려도 못 씀
- 대신 `requirements.txt`를 공유해서 각자 환경 구성

---

## 📌 의존성 (Dependencies) & requirements.txt

**Q. 의존성이 뭐야?**

하나의 소프트웨어가 동작하기 위해 필요로 하는 다른 소프트웨어나 라이브러리.

```
requests 설치
    ↓
requests가 동작하려면 다른 패키지들도 필요해
    ↓
pip가 자동으로 줄줄이 같이 설치해줌
```

**관련 명령어:**

```bash
pip list                          # 현재 패키지 목록 확인
pip freeze > requirements.txt     # 패키지 목록을 파일로 저장
pip install -r requirements.txt   # 파일에서 패키지 한번에 설치
```

> `>` = CLI Redirection Operator. 출력 방향을 화면 → 파일로 바꿈  
> `>>` = 이어쓰기 / `>` = 덮어쓰기

**팀 프로젝트 흐름:**
```
나        →  pip freeze > requirements.txt → Git push
팀원/서버  →  git pull → pip install -r requirements.txt
→ 똑같은 환경 완성!
```

> ☁️ **DevOps 연결**: CI/CD, Docker 배포 시에도 `requirements.txt` 그대로 사용!
> ```dockerfile
> COPY requirements.txt .
> RUN pip install -r requirements.txt
> ```

---

## 📌 Django 프로젝트 시작 전체 흐름

```bash
# 1. 가상환경 생성 및 활성화
python -m venv venv
source venv/Scripts/activate   # Windows
source venv/bin/activate        # Mac/Linux

# 2. Django 설치
pip install django

# 3. 프로젝트 생성
django-admin startproject config .

# 4. 앱 생성 (반드시 생성 먼저!)
python manage.py startapp articles

# 5. settings.py에 앱 등록
INSTALLED_APPS = [..., 'articles']

# 6. 서버 실행
python manage.py runserver
```

> `.` 없이 하면 폴더가 중첩됨 (`config/config/`) → 반드시 `.` 붙이기!  
> 앱 등록은 반드시 **생성 후** 에 할 것!

---

## 📌 프로젝트 & 앱 구조

```
프로젝트 루트/
├── manage.py            ← Django 프로젝트 관리 명령어 도구
├── config/              ← 프로젝트 설정 폴더
│   ├── settings.py      ← 프로젝트 모든 설정 관리
│   ├── urls.py          ← URL 연결 관리
│   ├── wsgi.py          ← 배포 관련 (동기)
│   ├── asgi.py          ← 배포 관련 (비동기)
│   └── __init__.py      ← 패키지 인식 파일
└── articles/            ← 앱 폴더
    ├── admin.py         ← 관리자 페이지 설정
    ├── apps.py          ← 앱 정보 (자동 생성)
    ├── models.py        ← 데이터 모델 정의 (MTV의 M)
    ├── views.py         ← HTTP 요청 처리 (MTV의 V)
    ├── tests.py         ← 테스트 코드
    └── migrations/      ← DB 변경사항 기록
```

**프로젝트 vs 앱:**
- **프로젝트** = 애플리케이션의 집합. DB 설정, URL 연결, 전체 앱 설정 담당
- **앱** = 독립적으로 작동하는 기능 단위 모듈

---

## 📌 디자인 패턴 - MTV

**MVC vs MTV 비교:**

| 역할 | MVC | MTV (Django) |
|---|---|---|
| 데이터/비즈니스 로직 | Model | Model |
| 사용자에게 보이는 화면 | View | Template |
| 요청 처리/교통정리 | Controller | View |

**Django MTV 흐름:**
```
사용자 요청
    ↓
urls.py  →  "이 URL은 어디로?"
    ↓
views.py →  "데이터 가져오고, 템플릿에 전달"
    ↓
models.py ↔ views.py ↔ templates
    ↓
사용자에게 HTML 응답
```

---

## 📌 새 페이지 만들 때 3단계 패턴

> Django의 핵심 패턴! 새 페이지마다 이 3단계 반복

**1. urls.py - 경로 등록**
```python
from django.contrib import admin
from django.urls import path
from articles import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('articles/', views.index),  # URL 끝은 반드시 /
]
```

**2. views.py - 함수 작성**
```python
from django.shortcuts import render

def index(request):  # 첫 번째 인자는 반드시 request
    return render(request, 'articles/index.html')
```

**3. templates/articles/index.html - HTML 작성**
```html
<!DOCTYPE html>
<html>
  <body>
    <h1>Hello Django!</h1>
  </body>
</html>
```

> **templates 안에 앱 이름 폴더를 만드는 이유**: 앱이 여러 개일 때 템플릿 이름 충돌 방지!

---

## 📌 동기 vs 비동기

| | 동기 (WSGI) | 비동기 (ASGI) |
|---|---|---|
| 방식 | 요청 하나씩 순서대로 처리 | 여러 요청 동시 처리 |
| 언제 | 일반 웹사이트 | 실시간 채팅, 웹소켓 |

> Django 기본은 **WSGI**  
> `manage.py runserver`는 개발용 → 실제 배포엔 **gunicorn + wsgi.py** 사용!

---

## ☁️ DevOps 연결 포인트 정리

| Django 개념 | DevOps 연결 |
|---|---|
| `requirements.txt` | Docker 이미지 빌드, CI/CD 자동 설치 |
| `settings.py` | 환경변수로 개발/배포 환경 분리 (`DEBUG=False`) |
| `tests.py` | CI/CD 파이프라인에서 자동 테스트 후 배포 |
| `wsgi.py` | gunicorn과 연동해서 실제 서버 배포 |
| MTV 구조 분리 | Docker 컨테이너 역할 분리 |