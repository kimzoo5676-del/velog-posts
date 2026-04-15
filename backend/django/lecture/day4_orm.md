# Django ORM - Model 사용하기 🗄️
> Django QuerySet API로 데이터베이스와 대화하는 법

---

## 📌 오늘의 흐름

- **어제 (3장)** → Model **정의** (클래스 작성, 필드 설정, 마이그레이션)
- **오늘 (4장)** → Model **사용** (QuerySet API로 실제 데이터 CRUD)

---

## 📌 ORM이란?

**Object Relational Mapping** 

> **객체 지향 프로그래밍 언어 객체(Object)**
> **데이터베이스 데이터(Data)**
> **매핑(Mapping)**하는 기술

> Django는 파이썬 문법만으로 데이터베이스와 대화할 수 있다

여기서 **R = Relational**, 즉 **관계형 데이터베이스(RDB)** 를 의미한다.
ORM은 파이썬 객체 ↔ RDB 테이블을 매핑해주는 기술이다.

```
[파이썬 세계]          [RDB 세계]
클래스            ←→   테이블
객체              ←→   행 (row / 레코드)
속성              ←→   열 (column / 필드)
```

> 💡 **RDB란?** 데이터를 테이블(표) 형태로 저장하고, 테이블 간 관계로 데이터를 연결하는 DB.
> MySQL, PostgreSQL, SQLite 등이 모두 RDB!

---

## 📌 QuerySet API

> 복잡한 SQL 쿼리문을 직관적인 파이썬 코드로 다룰 수 있게 해주는 강력한 번역기

ORM은 Django 개발자를 위해 **QuerySet API** 라는 특별한 도구를 제공한다.
ORM의 기능을 개발자가 Python 코드 안에서 **객체 지향적이고 직관적인 방식**으로
DB를 조작할 수 있도록 제공하는 **인터페이스**다.

**SQL vs QuerySet API 비교**

```sql
SELECT *                    -- 모든 컬럼을
FROM articles               -- articles 테이블에서
WHERE title = '안녕'        -- title이 '안녕'인 것만
ORDER BY created_at DESC;   -- created_at 기준 내림차순 정렬
```

```python
Article.objects                    # Article 모델(테이블)에서
  .filter(title='안녕')            # title이 '안녕'인 것만
  .order_by('-created_at')         # '-' = 내림차순, created_at 기준 정렬
```

> 💡 `.order_by('-created_at')` 에서 **`-` (마이너스)** 가 붙으면 내림차순(최신순), 없으면 오름차순!

**QuerySet API가 코드의 가독성을 높이고 개발 생산성을 극대화하는 Django ORM의 핵심 기능인 이유가 바로 이것!**

---

## 📌 ORM의 동작 방식

```
[Django 코드]
Article.objects.filter(title='안녕')
        ↓  ORM이 SQL로 변환
[SQL 쿼리]
SELECT * FROM articles WHERE title = '안녕';
        ↓  DB가 처리 후 결과 반환
[SQL 결과] (raw 데이터)
        ↓  ORM이 파이썬 객체로 변환
[Django로 반환]
 ├─ 여러 개 → QuerySet (리스트처럼 다룰 수 있는 객체)
 └─ 한 개   → Instance (파이썬 객체 하나)
```

| 방향 | 형태 | 설명 |
|------|------|------|
| Django → DB | QuerySet API | 파이썬 코드로 요청 |
| ORM → DB | SQL | ORM이 번역해서 전달 |
| DB → ORM | SQL 결과 | DB가 raw 데이터 반환 |
| ORM → Django | QuerySet or Instance | 파이썬 객체로 변환해서 반환 |

> 💡 **핵심**: 개발자는 QuerySet API만 쓰면 되고, SQL 변환은 ORM이 알아서 해준다!

---

## 📌 QuerySet API 기본 구조

```python
Article.objects.all()
#  ↑       ↑      ↑
# 모델    매니저  QuerySet API 메서드
# 클래스
```

**구조 공식**
```
Model Class . Manager . QuerySet API 메서드
```

**① Article (모델 클래스)**
- DB 테이블을 파이썬 클래스로 표현한 것
- `articles_article` 테이블의 스키마(필드, 데이터 타입 등)를 정의
- Django ORM이 DB와 상호작용할 때 사용하는 기본 구조체

**② .objects (매니저, Manager)**
- DB 조회 작업을 위한 **기본 진입점**
- Django가 모든 모델에 **자동으로 추가**해주는 매니저
- 이 매니저를 통해 `.all()`, `.filter()` 등의 쿼리 메서드 호출 가능

> 💡 매니저 = "이 모델로 DB에 접근하려면 나를 통해!" 하는 문지기 역할

**③ .all() (QuerySet API 메서드)**
- 해당 모델과 연결된 테이블의 **모든 레코드(rows)를 조회**

```python
Article.objects.all()
```
```sql
SELECT * FROM articles_article;
-- 가져와라  모든 컬럼을  articles_article 테이블에서
```

---

## 📌 Query 관련 용어 정리

헷갈리기 쉬운 Query 관련 용어들, 한 번에 정리!

| 용어 | 의미 | 한 줄 설명 |
|------|------|-----------|
| **Query** | 질의 | DB에 데이터 달라고 보내는 **요청** 자체 |
| **QuerySet** | 데이터 꾸러미 | DB에서 받아온 **데이터 목록** (유사 리스트) |
| **QuerySet API** | 메서드 모음 | QuerySet을 다루는 **기능들** (`.all()`, `.filter()` 등) |

```
개발자가 QuerySet API 로  →  Query 를 날림  →  결과로 QuerySet 을 받음
     (도구)                   (요청)              (데이터 꾸러미)
```

> 💡 **Query** 는 행위, **QuerySet** 은 결과물, **QuerySet API** 는 그 행위를 가능하게 해주는 도구!

---

## 📌 QuerySet이란?

> 데이터베이스에서 전달받은 **객체 목록 (데이터 모음)**

- Django ORM을 통해 만들어진 자료형
- 순회 가능한 데이터로, 1개 이상의 데이터를 불러와 사용 가능
- 파이썬 리스트와 유사하지만 완전히 같지는 않음

```python
articles = Article.objects.all()
# <QuerySet [<Article: Article object (1)>, <Article: Article object (2)>, ...]>

# 리스트처럼 순회 가능
for article in articles:
    print(article.title)
```

> 💡 **유사 리스트인 이유?**
> 리스트처럼 순회(for문), 인덱싱(`articles[0]`)은 되지만,
> `.filter()`, `.order_by()` 같은 **추가 쿼리 조작**이 가능한 게 일반 리스트와의 차이!

**⚠️ 예외: 단일 객체 반환**

DB가 **단일 객체**를 반환할 때는 QuerySet이 아닌 **모델 인스턴스**로 반환된다!

| 상황 | 반환 타입 | 예시 |
|------|-----------|------|
| 여러 개 조회 | QuerySet | `Article.objects.all()` |
| 단일 객체 조회 | Instance | `Article.objects.get(id=1)` |

```python
# QuerySet (여러 개) → 대괄호 [ ] 있음
articles = Article.objects.all()
# <QuerySet [<Article object (1)>, <Article object (2)>]>

# Instance (한 개) → 대괄호 없이 객체 하나
article = Article.objects.get(id=1)
# <Article object (1)>
print(article.title)    # 바로 필드 접근 가능
print(article.content)
```

> ⚠️ `.get()` 은 **무조건 1개**를 반환한다고 확신할 때만 써야 한다!
> 결과가 0개거나 2개 이상이면 에러 발생!

---

## 📌 CRUD 실습 준비

### 실습 환경 세팅

```bash
# 1. 가상환경 생성 & 활성화
python -m venv venv
source venv/bin/activate        # Mac/Linux
venv\Scripts\activate           # Windows

# 2. 의존성 설치
pip install -r requirements.txt

# 3. IPython 설치 (더 편한 shell 환경)
pip install ipython

# 4. requirements.txt 업데이트 (중요!)
pip freeze > requirements.txt

# 5. DB 생성 (마이그레이션)
python manage.py makemigrations
python manage.py migrate

# 6. Django Shell 실행
python manage.py shell -v 2
```

> 💡 **`pip freeze > requirements.txt` 왜 하냐?**
> 새로 설치한 패키지를 기록해두지 않으면, 나중에 다른 환경에서 `pip install -r requirements.txt` 했을 때 빠져있어서 에러 발생!

> CI/CD 파이프라인이나 Docker 빌드할 때도 이 파일 기준으로 패키지를 설치하기 때문에 항상 업데이트하는 습관이 중요하다.

> 💡 **`python manage.py shell` vs `python manage.py shell -v 2` 차이**
> ```bash
> # -v 2 없이 실행
> $ python manage.py shell
> 7 objects imported automatically (use -v 2 for details).
> # → 자동 import는 됐는데 뭐가 import됐는지 안 보여줌
>
> # -v 2 옵션 추가
> $ python manage.py shell -v 2
> 7 objects imported automatically:
> from articles.models import Article   # ← 뭐가 import됐는지 상세히 보여줌!
> ```
> `-v 2` = verbosity(상세도) 옵션. 자동 import된 목록을 눈으로 직접 확인할 수 있어서 학습할 때 유용!

### CRUD란?

> 대부분의 소프트웨어가 가지는 **기본적인 데이터 처리 기능 4가지**

| 이름 | 의미 | SQL | QuerySet API |
|------|------|-----|--------------|
| **C**reate | 생성 | `INSERT` | `.create()`, `.save()` |
| **R**ead | 조회 | `SELECT` | `.all()`, `.filter()`, `.get()` |
| **U**pdate | 수정 | `UPDATE` | `.save()`, `.update()` |
| **D**elete | 삭제 | `DELETE` | `.delete()` |

> 💡 **`.save()` 가 Create랑 Update 두 곳에 있는 이유?**
> `.save()` 는 내부적으로 id 유무를 보고 알아서 판단한다!
> - id 없음 → `INSERT` (새로 생성)
> - id 있음 → `UPDATE` (기존 수정)

---

## 📌 CREATE - 데이터 생성 3가지 방법

### 방법 1 - 빈 객체 생성 후 값 할당 및 저장

```python
# ① 빈 객체 생성 직후
article = Article()
article             # <Article: Article object (None)>  ← id 없음
article.title       # ''                                ← 빈 문자열
article.pk          # None                              ← pk도 없음

# ② 값 할당 후 (save 전)
article.title = 'first'
article.content = 'django!'
article.title       # 'first'                           ← 값은 들어있음
article.pk          # None                              ← 아직 DB에 없음!

# ③ save 후
article.save()
article             # <Article: Article object (1)>     ← id 부여됨
article.pk          # 1
article.title       # 'first'
article.content     # 'django!'
article.created_at  # 2026-04-15 09:00:00+00:00        ← 자동 저장됨!

# ④ DB 전체 확인
Article.objects.all()
# <QuerySet [<Article: Article object (1)>]>            ← DB에 1개 있음
```
```sql
INSERT INTO articles_article (title, content) VALUES ('first', 'django!');
-- 삽입해라  articles_article 테이블에  title 컬럼엔 'first',  content 컬럼엔 'django!'
```

> 💡 **핵심 포인트**
> - `save()` 전 → 파이썬 메모리에만 존재, pk = None
> - `save()` 후 → DB에 저장, pk 부여, `created_at` 자동 채워짐
> - `Article.objects.all()` → DB에 실제로 저장된 것만 보임

---

### 방법 2 - 초기값과 함께 객체 생성 및 저장

```python
# ① 값과 함께 객체 생성 (save 전)
article = Article(title='second', content='django!')
article             # <Article: Article object (None)>  ← 아직 None!
article.title       # 'second'                          ← 값은 들어있음
article.pk          # None                              ← 아직 DB에 없음

# ② save 후
article.save()
article             # <Article: Article object (2)>     ← id 부여됨
article.pk          # 2
article.created_at  # 2026-04-15 09:01:00+00:00        ← 자동 저장됨!

# ③ DB 전체 확인
Article.objects.all()
# <QuerySet [<Article: Article object (1)>, <Article: Article object (2)>]>
```
```sql
INSERT INTO articles_article (title, content) VALUES ('second', 'django!');
-- 삽입해라  articles_article 테이블에  title 컬럼엔 'second',  content 컬럼엔 'django!'
```

> 💡 **1번 vs 2번** → 값을 넣는 시점만 다를 뿐, `save()` 전까지 pk = None 인 건 똑같다!

---

### 방법 3 - create() 메서드로 한 번에 생성 및 저장

```python
# ① create() 한 줄로 생성 + 저장 완료
article = Article.objects.create(title='third', content='django!')
article             # <Article: Article object (3)>     ← 바로 id 부여됨!
article.pk          # 3
article.title       # 'third'
article.created_at  # 2026-04-15 09:02:00+00:00        ← 자동 저장됨!

# ② DB 전체 확인
Article.objects.all()
# <QuerySet [<Article: Article object (1)>, <Article: Article object (2)>, <Article: Article object (3)>]>
```
```sql
INSERT INTO articles_article (title, content) VALUES ('third', 'django!');
-- 삽입해라  articles_article 테이블에  title 컬럼엔 'third',  content 컬럼엔 'django!'
```

> 💡 **3가지 방법 최종 비교**
>
> | | 방법 1 | 방법 2 | 방법 3 |
> |--|--------|--------|--------|
> | 줄 수 | 3줄 | 2줄 | 1줄 |
> | 값 할당 시점 | 객체 생성 후 | 객체 생성 시 | create() 안에서 |
> | save() 필요 | ✅ | ✅ | ❌ (자동) |
> | 저장 전 추가처리 | ✅ 가능 | ✅ 가능 | ❌ 불가 |
> | 실무 사용 | 추가처리 필요할 때 | 추가처리 필요할 때 | 가장 많이 사용 ⭐ |

### .save() 메서드란?

> 객체를 데이터베이스에 저장하는 **인스턴스 메서드**

`.save()` 는 id 유무를 보고 내부적으로 알아서 판단한다:
- id 없음 → `INSERT` (새로 생성)
- id 있음 → `UPDATE` (기존 수정)

**`.save()` 가 필요한 경우** — 저장 전에 추가 처리가 필요할 때!
```python
article = Article(title='first', content='django!')
article.user = request.user   # 다른 데이터와 관계 설정
article.full_clean()          # 유효성 검사
article.save()                # 처리 다 끝난 후 저장
```

> 💡 반면 `.create()` 는 중간 처리 없이 바로 저장해버리기 때문에,
> 추가 처리가 필요한 상황에선 1번, 2번 방식 + `.save()` 조합이 더 적합!

### pk (Primary Key)

```python
article = Article()
article   # <Article: Article object (None)>  ← 저장 전, id 없음

article.save()
article   # <Article: Article object (1)>     ← 저장 후, id 부여됨!
```

> 💡 **pk = Primary Key (기본키)**
> 테이블에서 각 행(row)을 **유일하게 식별**하는 값.
> Django는 id 필드를 자동으로 만들고, 이게 곧 pk!
> ```python
> article.id  # 1
> article.pk  # 1  ← 같은 값!
> ```

---

## 📌 READ - 데이터 조회

| 메서드 | 반환 타입 | 설명 |
|--------|-----------|------|
| `.all()` | QuerySet | 전체 조회 |
| `.filter()` | QuerySet | 조건 조회 (0~n개) |
| `.get()` | Instance | 단일 객체 조회 (반드시 1개) |

### .all() - 전체 조회

```python
Article.objects.all()
# <QuerySet [<Article object (1)>, <Article object (2)>, <Article object (3)>]>
```
```sql
SELECT * FROM articles_article;
-- 가져와라  모든 컬럼을  articles_article 테이블에서
```

> 💡 DB 상태를 모른다면 `.all()` 결과는 **0 ~ n개** 라고 예상하면 된다!
> 빈 QuerySet `[]` 이거나 데이터가 있거나 — 에러는 절대 안 남.

### .filter() - 조건 조회

```python
# content가 'django!'인 것 전부 조회
Article.objects.filter(content='django!')
# <QuerySet [<Article object (1)>, <Article object (2)>, <Article object (3)>]>
```
```sql
SELECT * FROM articles_article WHERE content = 'django!';
-- 가져와라  모든 컬럼을  테이블에서  content가 'django!'인 것만
```

```python
# 없는 데이터 조회 → 에러 없이 빈 QuerySet 반환
Article.objects.filter(title='abc')
# <QuerySet []>

# 1개만 있어도 QuerySet으로 반환!
Article.objects.filter(title='first')
# <QuerySet [<Article object (1)>]>  ← 대괄호 있음!
```

> 💡 **filter() 핵심** — 결과가 0개든, 1개든, n개든 무조건 **QuerySet** 반환!

### .get() - 단일 객체 조회

```python
Article.objects.get(pk=1)
# <Article: Article object (1)>  ← 대괄호 없음! Instance 반환
```
```sql
SELECT * FROM articles_article WHERE id = 1;
-- 가져와라  모든 컬럼을  테이블에서  id가 1인 것
```

> 💡 **get() 조건이 엄격한 이유!**
>
> | 상황 | 결과 |
> |------|------|
> | 1개 존재 | Instance 반환 ✅ |
> | 0개 (없음) | `DoesNotExist` 에러 💥 |
> | 2개 이상 | `MultipleObjectsReturned` 에러 💥 |
>
> 그래서 **고유성이 보장된 pk(=id)** 로 조회할 때 주로 사용!

### filter() vs get() 비교

```python
# filter → 1개여도 QuerySet에 담김
Article.objects.filter(title='first')
# <QuerySet [<Article object (1)>]>  ← 대괄호 [ ] 있음!

# get → 바로 Instance로 반환
Article.objects.get(pk=1)
# <Article object (1)>  ← 대괄호 없음!
```

| 메서드 | 언제 써? |
|--------|---------|
| `.all()` | 전체 다 가져올 때 |
| `.filter()` | 조건 있는데 결과가 여러 개일 수 있을 때 |
| `.get()` | **고유성이 보장된** 조건으로 딱 1개 가져올 때 |

---

## 📌 UPDATE - 데이터 수정

```python
# 1. 수정할 인스턴스 조회
article = Article.objects.get(pk=1)
```
```sql
SELECT * FROM articles_article WHERE id = 1;
-- 가져와라  모든 컬럼을  테이블에서  id가 1인 것
```

```python
# 2. 인스턴스 변수 변경 (아직 DB 반영 안 됨!)
article.title = 'byebye'

# 3. 저장
article.save()
```
```sql
UPDATE articles_article SET title = 'byebye' WHERE id = 1;
-- 수정해라  articles_article 테이블에서  title을 'byebye'로  id가 1인 행을
```

> 💡 수정할 대상은 **딱 1개** 를 정확히 집어야 하니까 `get(pk=__)` 을 주로 활용!

---

## 📌 DELETE - 데이터 삭제

```python
# 1. 삭제할 인스턴스 조회
article = Article.objects.get(pk=1)
```
```sql
SELECT * FROM articles_article WHERE id = 1;
-- 가져와라  모든 컬럼을  테이블에서  id가 1인 것
```

```python
# 2. 삭제
article.delete()
# (1, {'articles.Article': 1})
```
```sql
DELETE FROM articles_article WHERE id = 1;
-- 삭제해라  articles_article 테이블에서  id가 1인 행을
```

> 💡 **반환값 `(1, {'articles.Article': 1})` 의 의미**
> - 앞의 `1` → 삭제된 행의 **총 개수**
> - `{'articles.Article': 1}` → Article 테이블에서 1개 삭제됐다는 **상세 내역**

> ⚠️ 삭제된 데이터는 **복구 불가!**
> 삭제 후 `Article.objects.get(pk=1)` 하면 `DoesNotExist` 에러 발생

---

## 📌 ORM with View - 전체 게시글 조회

Shell에서 연습한 ORM을 실제 views.py에 연결하는 단계!

### urls.py 설정

```python
# crud/urls.py
path('articles/', include('articles.urls'))
# → articles/ 로 시작하는 요청은 articles/urls.py 로 넘겨라

# articles/urls.py
path('', views.index, name='index')
# → articles/ + '' = articles/ 로 접속하면 views.index 함수 실행
```

### views.py

```python
from django.shortcuts import render
from .models import Article          # Article 모델 import

def index(request):
    articles = Article.objects.all() # 모든 게시글 QuerySet으로 가져오기
    context = {
        'articles': articles,        # template에 넘겨줄 데이터
    }
    return render(request, 'articles/index.html', context)
```
```sql
SELECT * FROM articles_article;
-- 가져와라  모든 컬럼을  articles_article 테이블에서
```

### index.html

```html
<body>
  <h1>Article</h1>
  <hr>
  {% for article in articles %}
  <div>
    <p>글 번호: {{ article.pk }}</p>
    <p>글 제목: {{ article.title }}</p>
    <p>글 내용: {{ article.content }}</p>
    <hr>
  </div>
  {% endfor %}
</body>
```

```
[브라우저 화면 - 127.0.0.1:8000/articles/]

Articles
────────────────────
글 번호: 2
글 제목: second
글 내용: django!
────────────────────
글 번호: 3
글 제목: third
글 내용: django!
────────────────────
```

> 💡 pk=1 은 DELETE 실습에서 삭제했으니까 2, 3만 출력됨!

> 💡 **전체 흐름**
> ```
> 사용자가 articles/ 접속
>     ↓
> index 함수 실행
>     ↓
> Article.objects.all() 로 DB에서 데이터 가져옴
>     ↓
> context에 담아서 index.html로 전달
>     ↓
> template에서 for문으로 하나씩 꺼내서 출력
> ```

---

## 📌 Field Lookups - 데이터 필터링의 마법

> 단순 동치 비교(`=`)를 넘어 **더 상세한 조건**으로 데이터를 조회할 수 있도록 Django ORM이 제공하는 기능

**사용 방법**
```
필드이름 __ 조회유형 = 값
   ↑    ↑      ↑
 title  __  startswith
```

```python
# 내용에 'dja'가 포함된 모든 게시글 조회
Article.objects.filter(content__contains='dja')
```
```sql
SELECT * FROM articles_article WHERE content LIKE '%dja%';
-- 가져와라  모든 컬럼을  테이블에서  content에 'dja'가 포함된 것만
```

```python
# 제목이 'he'로 시작하는 모든 게시글 조회
Article.objects.filter(title__startswith='he')
```
```sql
SELECT * FROM articles_article WHERE title LIKE 'he%';
-- 가져와라  모든 컬럼을  테이블에서  title이 'he'로 시작하는 것만
```

**자주 쓰는 Field Lookups**

| Lookup | 의미 | 예시 |
|--------|------|------|
| `__contains` | 포함 | `title__contains='django'` |
| `__startswith` | ~로 시작 | `title__startswith='he'` |
| `__endswith` | ~로 끝남 | `title__endswith='ng'` |
| `__gt` | 초과 | `pk__gt=2` |
| `__gte` | 이상 | `pk__gte=2` |
| `__lt` | 미만 | `pk__lt=2` |
| `__lte` | 이하 | `pk__lte=2` |

> 💡 `filter()`, `exclude()`, `get()` 전부 Field Lookups 사용 가능!
> 복잡한 SQL 조건도 파이썬 코드로 간결하게 처리할 수 있어.

---

## ☁️ DevOps 연결 포인트

| Django 개념 | DevOps 연결 |
|---|---|
| `pip freeze > requirements.txt` | 배포 환경에서 동일한 패키지 보장 |
| `venv` 가상환경 | Docker 컨테이너와 같은 환경 격리 개념 |
| `makemigrations` → `migrate` | 배포 파이프라인에서 자동 실행 가능 |