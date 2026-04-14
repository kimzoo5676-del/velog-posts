# Django 3장 - Model 복습 🗄️
> Django Model, Migration, Admin 정리

---

## 📌 Model이란?

**Q. Model이 뭐야?**

데이터베이스를 Python 클래스(객체)로 **추상화**해서 다루는 것.  
SQL 없이 파이썬 문법만으로 DB를 조작할 수 있게 해주는 레이어.

```
파이썬 클래스  =  DB 테이블
파이썬 객체    =  DB 행(row) 하나
```

**왜 쓰냐?**

- **접근성**: DB 깊은 지식 없이도 데이터 관리 가능
- **유지보수**: DB가 바뀌어도 (SQLite → PostgreSQL) 코드 수정 최소화
- **확장성**: 재사용 가능한 모델로 개발 효율성 향상

> ☁️ **DevOps 연결**: 개발/스테이징/프로덕션 환경마다 다른 DB를 써도 모델 코드는 그대로!

---

## 📌 MVT 요청 처리 흐름

```
사용자 요청
    ↓
urls.py       →  어떤 view로 보낼지 라우팅
    ↓
views.py      →  요청 처리 + models.py로 데이터 조작
    ↓
models.py     →  DB 구조 정의 & DB와 직접 상호작용
    ↓
views.py      →  DB에서 받은 데이터를 template에 전달
    ↓
templates     →  사용자에게 화면으로 출력
```

| 파일 | 역할 |
|------|------|
| `urls.py` | 요청의 시작점, 라우팅 |
| `views.py` | 요청 처리의 중심, 로직 담당 |
| `models.py` | DB 구조 정의 + DB 상호작용 |
| `templates` | 화면 구성, 사용자에게 출력 |

> 이번 장은 이 흐름 중에서 **`models.py` 파트에 집중**!

---

## 📌 SQLite & db.sqlite3

- Django 기본 DB 엔진 = **SQLite**
- 파일 하나(`db.sqlite3`)로 관리 → 별도 설치 불필요
- 타입 검사가 느슨함 → Django가 폼 레벨에서 미리 유효성 검사해줌 (아래 참고)

**`db.sqlite3`를 `.gitignore`에 넣는 이유:**
- 개인 데이터가 담겨있음
- 팀원 간 충돌 위험
- 환경마다 달라짐

---

## 📌 Model Class

**Q. Model 클래스가 뭐야?**

`models.Model`을 **상속받은** 파이썬 클래스.  
DB 테이블의 구조를 설계하는 **청사진(설계도)** 역할.

```python
from django.db import models

# 앱 이름은 복수형(articles), 클래스는 단수형(Article)
class Article(models.Model):
    title   = models.CharField(max_length=10)  # 짧은 문자열, 길이 제한 있음
    content = models.TextField()               # 긴 문자열, 길이 동적으로 결정
```

**왜 `models.Model`을 상속받냐?**

- `models.Model` 안에 DB 연동에 필요한 **모든 코드가 이미 작성**되어 있음
- `.save()`, `.delete()`, `.objects.all()` 같은 CRUD 메서드를 물려받음
- 개발자는 **"테이블 구조를 어떻게 설계할지"만 집중**하면 됨

**이렇게 쓰면 자동으로 만들어지는 테이블:**

| id | title | content |
|----|-------|---------|
| 1  | ...   | ...     |

> `id`는 Django가 자동 생성 → 직접 안 써도 됨  
> 테이블 이름도 자동으로 `앱이름_클래스이름` 형식으로 만들어짐 → `articles_article`

---

## 📌 Model Field

> 데이터베이스 테이블의 **열(column)을 나타내는** 중요한 구성요소  
> `models` 모듈의 클래스로 정의되어 있음

```python
class Article(models.Model):
    title   = models.CharField(max_length=10)  # ← 이게 Model Field (열 하나)
    content = models.TextField()               # ← 이게 Model Field (열 하나)
```

Model Field는 두 가지를 조합해서 정의해:

```
Field Type   +   Field Options
(데이터 종류)     (동작 & 제약 조건)

CharField    +   max_length=10, blank=True ...
```

**유효성 검사(Validation)와 Model Field의 관계**

SQLite는 타입 검사가 느슨해서 엉뚱한 값도 그냥 저장해버림.  
그래도 Field type/option을 명시적으로 써야 하는 이유:

```
사용자가 폼에 입력
    ↓
[Django 폼 레벨] → Field type/option 기반으로 유효성 검사
    → CharField(max_length=10) : 10자 넘으면 여기서 차단!
    → blank=False : 빈 값이면 여기서 차단!
    ↓
[DB 레벨] → 통과한 데이터만 저장
```

> DB가 안 따져줘도 **Django가 DB 가기 전에 먼저 걸러줌**  
> 나중에 SQLite → PostgreSQL로 바꿔도 Field 정의가 그대로 호환됨  
> 코드만 봐도 "이 컬럼엔 어떤 데이터가 들어오는지" 명확하게 알 수 있음

---

## 📌 Field Types

> 데이터베이스에 저장될 **데이터의 종류**를 정의  
> `models` 모듈의 클래스로 정의되어 있음

**문자열**

| 필드 | 설명 |
|------|------|
| `CharField(max_length=n)` | 제한된 길이의 문자열, `max_length` 필수 |
| `TextField()` | 대용량 텍스트, 길이 제한 없음 (DBMS마다 다름) |

**숫자**

| 필드 | 설명 |
|------|------|
| `IntegerField()` | 정수 |
| `FloatField()` | 실수 |

**날짜/시간**

| 필드 | 설명 |
|------|------|
| `DateField()` | 날짜 |
| `TimeField()` | 시간 |
| `DateTimeField()` | 날짜+시간 |

**파일**

| 필드 | 설명 |
|------|------|
| `FileField()` | 일반 파일 |
| `ImageField()` | 이미지 파일 |

> 모든 필드 외울 필요 없음 → 필요할 때 **Django 공식 문서** 참고!

---

## 📌 Field Options

> 필드의 **동작**과 **제약 조건**을 정의할 때 사용

**제약 조건(Constraint)이란?**  
테이블의 열/행에 적용되는 규칙이나 제한사항  
(숫자만 저장, 문자 100자 제한 등)

**주요 Field Options**

| 옵션 | 설명 | 기본값 |
|------|------|--------|
| `null` | DB에서 NULL 허용 여부 | `False` |
| `blank` | Form에서 빈 값 허용 여부 | `False` |
| `default` | 필드 기본값 설정 | - |
| `auto_now` | 저장될 때마다 현재 시간 자동 저장 | - |
| `auto_now_add` | 처음 생성될 때만 현재 시간 자동 저장 | - |

```python
class Article(models.Model):
    title      = models.CharField(max_length=10)
    content    = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)  # 생성 시간
    updated_at = models.DateTimeField(auto_now=True)      # 수정 시간
```

**💡 null vs blank 차이**

| | 어디서 작동? | 의미 |
|--|------|------|
| `null` | **DB 레벨** | NULL 저장 허용 |
| `blank` | **Django 폼 레벨** | 빈 칸 제출 허용 |

```
NULL   =  컵 자체가 없는 상태
blank  =  컵은 있는데 비어있는 상태 ("")
```

> 문자열 필드(`CharField`, `TextField`)는 `null=True` 잘 안 씀  
> 빈 값이면 `NULL` 대신 **빈 문자열(`""`)**로 저장하는 게 Django 관례!

---

## 📌 CRUD

> 데이터를 다루는 4가지 기본 동작  
> 지금은 개념만! 실제 코드로 다루는 건 views에서 배움

| 이름 | 의미 | DB 명령 |
|------|------|---------|
| **C**reate | 생성 | INSERT |
| **R**ead   | 조회 | SELECT |
| **U**pdate | 수정 | UPDATE |
| **D**elete | 삭제 | DELETE |

Django에서는 SQL 없이 파이썬으로 다 됨:

```python
Post.objects.create(title="안녕")    # Create
Post.objects.all()                   # Read
post.title = "수정됨"; post.save()   # Update
post.delete()                        # Delete
```

---

## 📌 Migrations

> `models.py`의 변경사항을 **실제 DB에 반영**하는 과정  
> 모든 변경사항이 파일로 관리되어 협업 시 추적과 공유가 수월

**언제 Migration이 필요할까?**

```
model class에 변경사항(1)이 생겼다면
반드시 새로운 설계도를 생성(2)하고
이를 DB에 반영(3)해야 한다

(1) models.py 생성/수정  →  (2) makemigrations  →  (3) migrate
```

**전체 흐름:**

```
models.py 작성         ←  설계도 초안 (내가 씀)
    ↓
python manage.py makemigrations
    →  migrations/0001_initial.py 생성 (최종 설계도 파일)
    ↓
python manage.py migrate
    →  설계도를 읽고 실제 DB에 테이블 생성/수정
```

> `models.py` 건드렸으면 **무조건 makemigrations → migrate** 두 방이면 끝!

**협업 관점에서 중요한 이유:**

```
개발자 A 로컬
    models.py 수정
        ↓
    makemigrations  →  0002_xxx.py 생성  ← 이게 Git에 올라가는 것
        ↓
    migrate         →  A의 db.sqlite3에만 반영
                       (db.sqlite3는 .gitignore라 Git에 안 올라감!)
        ↓
    Git push

개발자 B 로컬
    git pull        →  0002_xxx.py (설계도 파일)만 받음
        ↓
    migrate         →  B의 로컬 DB에 반영됨
        ↓
    동일한 DB 구조 유지 완료!
```

> `db.sqlite3`는 `.gitignore`에 있어서 **Git에 절대 안 올라감**  
> Git으로 공유되는 건 **migration 파일(설계도)만**  
> 각자 `migrate`를 직접 실행해야 본인 DB에 반영됨

> ☁️ **DevOps 연결**: migration 파일이 Git으로 공유되어 배포 환경에서도 동일한 DB 구조 보장!

---

## 📌 Migration 경고 메시지

**① 프로젝트 처음 만들고 migrate 안 했을 때**

```
You have 18 unapplied migration(s). Your project may not work properly...
```

Django는 프로젝트 생성 시점부터 기본 앱들의 migration 파일이 이미 존재함:

```
django.contrib.auth     ← 로그인/유저 관리
django.contrib.admin    ← 관리자 페이지
django.contrib.sessions ← 세션 관리
...
```

이것들이 DB에 반영이 안 된 상태라 뜨는 경고.  
`python manage.py migrate` 한 번 실행하면 바로 사라짐.

---

**② 기존 테이블에 새 필드 추가할 때**

```
It is impossible to add a non-nullable field 'created_at'
to article without specifying a default.
```

> "기본값 없이는 created_at 필드를 추가할 수 없어"

열(column)을 추가하면 **기존 row에도 그 칸이 생겨버려:**

```
기존 테이블:
| id | title | content |
| 1  | 안녕  | 내용    |
| 2  | 반가워 | 내용2  |

created_at 열 추가하면?
| id | title | content | created_at |
| 1  | 안녕  | 내용    | ???        |  ← 기존 row!
| 2  | 반가워 | 내용2  | ???        |  ← 기존 row!
```

`null=False`가 기본값이라 빈 칸으로 둘 수 없으니까  
Django가 "기존 row에 뭘 채울거야?" 라고 물어보는 것!

**선택지 두 개:**

```
1) Provide a one-off default now
   → 지금 바로 임시 기본값 정해줄게
   → 1 입력 후 엔터
   → [default: timezone.now] >>> 엔터
   → 현재 시간으로 기존 row 채워줌

2) Quit and manually define a default value in models.py
   → 나가서 models.py에 직접 설정할게
```

**해결 방법 2가지:**

| 방법 | 설명 | 추천 상황 |
|------|------|----------|
| models.py에 옵션 미리 설정 | 경고 자체가 안 뜸 | 실제 서비스, 의도를 코드에 명확히 |
| makemigrations 후 1번 선택 | 경고 뜨면 그때 처리 | 빠른 테스트, 급할 때 |

```python
# 방법 1 - 처음부터 옵션 명시 (권장)
created_at = models.DateTimeField(auto_now_add=True)  # 경고 안 뜸
updated_at = models.DateTimeField(auto_now=True)      # 경고 안 뜸
```

> `auto_now_add=True` = "이 필드는 현재 시간으로 채우면 돼"라는 규칙을 Django한테 미리 알려주는 것  
> 규칙이 있으니까 물어볼 필요 없이 알아서 채워줌!

---

## 📌 Migration 기타 명령어

```bash
# 적용 여부 확인
python manage.py showmigrations
```
```
articles
 [X] 0001_initial      ← X = 적용됨
 [ ] 0002_xxx          ← 빈칸 = 아직 migrate 안 됨
```

```bash
# Django가 실제로 어떤 SQL을 날리는지 확인
python manage.py sqlmigrate articles 0001
```
```sql
CREATE TABLE "articles_article" (
    "id" integer NOT NULL PRIMARY KEY,
    "title" varchar(10) NOT NULL,
    "content" text NOT NULL
);
```

> ☁️ **DevOps 연결**: `sqlmigrate`로 배포 전 DB 영향도 미리 검토 가능!

---

## 📌 Django Admin (관리자 인터페이스)

> 설치/설정 없이 Django가 **자동으로 제공**하는 관리 화면  
> `/admin` 으로 접속

**활용 상황**

| 상황 | 설명 |
|------|------|
| 빠른 프로토타이핑 | 뷰/템플릿 없이 데이터 바로 테스트 |
| 비개발자 데이터 관리 | 기획자/운영자가 직접 데이터 관리 |
| 내부 시스템 구축 | 사내 관리 도구로 활용 |

---

### ⚠️ Admin 사용을 위한 필수 2단계

**반드시 둘 다 해야 함! 하나라도 빠지면 admin에서 모델이 안 보임**

**Step 1. 슈퍼유저 생성**

```bash
python manage.py createsuperuser
```
```
Username: admin
Email address: (엔터 스킵 가능)
Password: ****
Password (again): ****

Superuser created successfully.
```

> `createsuperuser` = `auth_user` 테이블에 관리자 계정 row 하나를 INSERT 하는 것  
> 즉, 유저도 결국 DB에 저장되는 데이터!

**Step 2. admin.py에 모델 등록**

```python
# articles/admin.py
from django.contrib import admin
from .models import Article     # ← 반드시 import

admin.site.register(Article)    # ← 반드시 등록
```

> 슈퍼유저만 만들고 등록 안 하면 → `/admin` 들어가도 Article 메뉴 없음  
> 등록만 하고 슈퍼유저 없으면 → `/admin` 로그인 자체가 안 됨

**결과:**
```
/admin 접속
  ├── AUTHENTICATION AND AUTHORIZATION
  │     └── Users    ← 슈퍼유저 계정이 여기 저장됨
  └── ARTICLES
        └── Articles ← 등록한 모델이 여기 나타남 ✅
```

→ 웹 UI에서 CRUD 가능 (SQL 한 줄 없이!)

---

**Admin 커스터마이징 (공식 문서 참고):**

```python
@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    list_display  = ['id', 'title', 'created_at']  # 목록에 보일 컬럼
    list_filter   = ['created_at']                  # 필터 사이드바
    search_fields = ['title', 'content']            # 검색 기능
```

---

## 📌 DB 초기화 방법

> ⚠️ **반드시 서버 종료 후 진행!**

```
articles/
  └── migrations/
        ├── __init__.py        ← ❌ 삭제 금지!
        ├── 0001_initial.py    ← ✅ 삭제
        └── 0002_xxx.py        ← ✅ 삭제

db.sqlite3                     ← ✅ 삭제
```

초기화 후 다시 처음부터:

```bash
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
# admin.py 모델 등록도 다시 확인!
```

**실수로 `__init__.py`나 `migrations/` 폴더까지 지웠다면?**

```
migrations/ 폴더 직접 생성
    └── __init__.py 파일 생성 (내용 비워도 됨)
```

그 다음 평소처럼 `makemigrations` → `migrate` 하면 됨.

> `migrations/` 폴더와 `__init__.py`는 Django가 migration 파일을 찾는 위치!  
> 없으면 `makemigrations` 자체가 안 됨

---

## 📌 프로젝트 세팅 전체 흐름

```bash
# 1. 가상환경 & 패키지
python -m venv venv
source venv/Scripts/activate
pip install django
pip freeze > requirements.txt

# 2. .gitignore 설정
touch .gitignore
# venv/ 추가
# gitignore.io → windows + vscode + python + django 검색해서 붙여넣기

# 3. 프로젝트 & 앱 생성
django-admin startproject config .
python manage.py startapp articles

# 4. settings.py - 앱 등록
INSTALLED_APPS = [..., 'articles']

# 5. config/urls.py
from django.urls import path, include
urlpatterns = [
    path('admin/', admin.site.urls),
    path('articles/', include('articles.urls')),
]

# 6. articles/urls.py 생성
from django.urls import path
from . import views
app_name = 'articles'
urlpatterns = []

# 7. models.py 작성 후 migration
python manage.py makemigrations
python manage.py migrate

# 8. 슈퍼유저 생성
python manage.py createsuperuser

# 9. articles/admin.py 모델 등록 ← 반드시!
# from .models import Article
# admin.site.register(Article)
```

---

## 📌 서버 실행

```bash
python manage.py runserver
```

→ `http://127.0.0.1:8000` 접속하면 서버 확인  
→ `http://127.0.0.1:8000/admin` 접속하면 관리자 화면

> `runserver`는 **개발용 서버**야  
> 실제 배포할 땐 gunicorn + wsgi.py 사용!

---

## ☁️ DevOps 연결 포인트 정리

| Django 개념 | DevOps 연결 |
|---|---|
| `migrations/` 파일 | Git으로 공유 → 배포 환경에서도 동일한 DB 구조 |
| `makemigrations` → `migrate` | 배포 파이프라인에서 자동 실행 |
| `sqlmigrate` | 배포 전 DB 영향도 사전 검토 |
| SQLite → PostgreSQL | 환경변수로 DB 교체해도 모델 코드는 그대로 |
| `db.sqlite3` gitignore | 환경별 DB 분리 관리 |