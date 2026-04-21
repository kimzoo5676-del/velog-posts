# Django Authentication System 01

## 목차

1. [인증 (Authentication)](#1-인증-authentication)
2. [HTTP (HyperText Transfer Protocol)](#2-http-hypertext-transfer-protocol)
3. [HTTP의 핵심 특징](#3-http의-핵심-특징)
4. [쿠키 (Cookie)](#4-쿠키-cookie)
5. [세션 (Session)](#5-세션-session)
6. [쿠키 vs 세션](#6-쿠키-vs-세션)
7. [Django Authentication System](#7-django-authentication-system)
8. [Custom User Model](#8-custom-user-model)
9. [Custom User Model 설정 순서](#9-custom-user-model-설정-순서)
10. [Login 구현](#10-login-구현)
11. [context_processors](#11-context_processors)

---

## 1. 인증 (Authentication)

### 정의

> **"이 사람이 우리 사이트의 회원인지 아닌지 확인하는 것"**

쉽게 말하면 서버가 **"너 누구야?"** 라고 물으면,
클라이언트가 **"나 이 사이트 회원이에요"** 라고 증명하는 과정이다.

### 다양한 인증 방식

| 방식 | 예시 |
|------|------|
| **아이디 / 비밀번호** | 일반 로그인 |
| **소셜 로그인** | 구글, 카카오, 네이버 로그인 |
| **생체 인증** | 지문, 얼굴 인식 |

---

## 2. HTTP (HyperText Transfer Protocol)

### 정의

**HTML 문서와 같은 리소스들을 가져올 수 있도록 해주는 규약** (웹에서의 모든 데이터 교환의 기초)

웹 브라우저와 서버가 서로 대화하기 위해 사용하는 **공통 언어 / 약속**이다.

### 동작 방식

```
브라우저  →  "이 페이지 보여줘" (Request/요청)
서버      →  HTML, 이미지 등을 응답으로 전달 (Response/응답)
```

우리가 웹사이트를 보고, 이미지를 다운로드하고, 데이터를 주고받는 **모든 과정**이 HTTP라는 규칙 위에서 이루어진다.

---

## 3. HTTP의 핵심 특징

### 비연결 지향 (Connectionless)

> 서버는 요청에 대한 응답을 보낸 후 **연결을 끊음**. 클라이언트와 서버는 계속 연결된 상태가 아님.

```
클라이언트  →  요청 (Request)
서버        →  응답 (Response)
서버        →  연결 끊음 ✂️
클라이언트  →  다시 요청하려면 재연결 필요
```

**왜 비연결형으로 설계됐을까?**

수많은 사용자와의 연결을 계속 유지하면 서버의 메모리와 자원을 모두 차지하게 된다. → **자원 낭비를 막기 위해** 비연결 방식을 채택.

> 💡 **비유**
> 편의점 알바생(서버)이 손님(클라이언트) 한 명 한 명을 결제가 끝나면 바로 잊어버리는 것과 같다. 덕분에 수백 명의 손님을 효율적으로 응대할 수 있다.

---

### 무상태 (Stateless)

> 연결을 끊는 순간 클라이언트와 서버 간의 통신이 끝나며 **상태 정보가 유지되지 않음**

**무상태의 실제 문제**
- 장바구니에 담은 상품을 **유지할 수 없음**
- 로그인 상태를 **유지할 수 없음**

아무 장치가 없으면 페이지 이동할 때마다 로그인이 풀려버린다!

**왜 무상태로 설계됐을까?**

서버가 모든 클라이언트 상태를 기억하면 두 가지 큰 문제가 생긴다.

첫째, 모든 클라이언트 상태를 저장/관리해야 하므로 **서버가 매우 복잡해진다.**

둘째, 실제 서비스는 여러 대의 서버로 운영되는데 상태를 유지하려면 서버끼리 상태를 공유해야 한다. 그러면 서버 간 연결이 필요해지고 **확장(Scale-out)이 매우 어려워진다.**

```
[Stateful 방식의 문제]
클라이언트 → 서버A (로그인 상태 저장)
다음 요청  → 서버B (서버A 상태를 모름 ❌)

[Stateless 방식의 해결]
클라이언트 → 서버A, 서버B 어디든 상관없음 ✅
```

결국 무상태는 서버의 부담을 없애고, **어떤 서버든 자유롭게 요청을 처리**할 수 있게 만들어 **대규모 웹 서비스 구축의 핵심**이 된다.

---

### Connectionless vs Stateless 비교

| | Connectionless | Stateless |
|---|---|---|
| **핵심** | 응답 후 연결을 끊음 | 상태 정보를 저장하지 않음 |
| **목적** | 서버 자원 절약 | 서버 확장성 확보 |
| **문제** | 매 요청마다 재연결 필요 | 로그인/장바구니 유지 불가 |

→ 이 두 문제를 해결하기 위해 **쿠키와 세션**이 등장한다!

---

## 4. 쿠키 (Cookie)

### 정의

> **서버가 사용자의 웹 브라우저에 전송하는 작은 데이터 조각**

쿠키는 웹사이트가 사용자의 브라우저에 남기는 작은 메모 같은 것이다. 이를 통해 서버는 사용자를 기억하고 식별할 수 있으며, 덕분에 매번 아이디/비밀번호를 다시 입력할 필요 없이 자동 로그인이 유지된다.

> 💡 **Stateless 우회**
> 서버가 상태를 저장하는 게 아니라, **클라이언트가 상태를 직접 들고 다니는 방식**으로 Stateless를 **우회(workaround)**하는 것이다. 완전한 해결이 아니라 우회라는 점이 핵심!

---

### 쿠키 저장 방식

- 브라우저(클라이언트)는 쿠키를 **KEY-VALUE** 형식으로 저장
- 이름, 값 외에도 **만료 시간, 도메인, 경로** 등의 추가 속성이 포함됨

```
예시)
session_id = abc123          ← Key : Value
Expires    = Sun, 26 Apr 2026
Domain     = .coupang.com
Path       = /
```

---

### 쿠키 동작 흐름

```
[최초 방문 — 쿠키 발급]
① 브라우저 → 서버 : 웹 페이지 요청
② 서버 → 브라우저 : 요청된 페이지 + Set-Cookie 헤더로 쿠키 전송
③ 브라우저 : 쿠키를 저장소에 저장 (만료 시간, 도메인, 경로 등 속성 포함)

[이후 방문 — 쿠키 재사용]
④ 브라우저 → 서버 : 요청 + 저장된 쿠키 자동 포함하여 전송
⑤ 서버 : 쿠키 확인 → 사용자 식별 / 세션 관리 수행
⑥ 서버 → 브라우저 : 응답 (필요 시 쿠키 수정/추가 가능)
```

> ✅ 쿠키는 **브라우저가 자동으로** 서버에 전송해준다. 단, 도메인과 경로가 일치하는 쿠키만 전송된다.

### 헤더 이름 차이 주의!

| 방향 | 헤더 이름 |
|---|---|
| 서버 → 브라우저 (쿠키 발급) | `Set-Cookie` |
| 브라우저 → 서버 (쿠키 전송) | `Cookie` |

---

### 쿠키 사용 목적

| 목적 | 설명 |
|---|---|
| **세션 관리 (Session Management)** | 로그인, 아이디 자동완성, 공지 하루 안 보기, 팝업 체크, 장바구니 |
| **개인화 (Personalization)** | 사용자 선호 설정 저장 (언어, 테마 등) |
| **추적 수집 (Tracking)** | 사용자 행동 기록 및 분석 (광고 타겟팅, 방문 패턴 분석) |

---

### 쿠키 종류별 Lifetime (수명)

| 종류 | 설명 |
|---|---|
| **Session Cookie** | 현재 세션 종료 시 삭제. 브라우저 종료와 함께 삭제됨 |
| **Persistent Cookie** | `Expires` 날짜 또는 `Max-Age` 기간이 지나면 삭제. 브라우저를 닫아도 유지됨 |

---

### 쿠키의 보안 장치

| 보안 장치 | 설명 |
|---|---|
| **제한된 정보** | 쿠키에는 중요하지 않은 정보만 저장 (민감한 정보는 서버에!) |
| **암호화** | 중요한 정보는 서버에서 암호화해서 쿠키에 저장 |
| **만료 시간** | 만료 시간 설정 → 시간이 지나면 자동 삭제 |
| **도메인 제한** | 특정 웹사이트에서만 사용할 수 있도록 설정 가능 |

> ⚠️ 쿠키는 탈취될 수 있으니, **비밀번호 등 민감한 정보는 절대 직접 저장하면 안 됨**
> 쿠키는 모든 요청에 포함되어 전송되므로, **크기를 최소화**해야 사이트 성능에 유리함

---

### 쿠키와 개인정보 보호

- 많은 국가에서 쿠키 사용에 대한 **사용자 동의를 요구하는 법규** 시행 (EU GDPR 등)
- 웹사이트는 쿠키 정책을 명시하고, 필요한 경우 **사용자의 동의를 얻어야 함**

> 우리가 웹사이트 접속할 때 자주 보이는 **쿠키 동의 팝업**이 바로 이 법규 때문!

---

### 🔍 개발자 도구로 쿠키 직접 확인하기

**Network 탭** — 쿠키가 오가는 과정 확인
```
F12 → Network 탭 → 요청 클릭 → Headers 탭
├── General          : 요청 URL, 메서드, 상태코드 등 기본 정보
├── Response Headers : 서버 → 브라우저 (Set-Cookie 확인)
└── Request Headers  : 브라우저 → 서버 (Cookie 확인)
```

**Application 탭** — 현재 브라우저에 저장된 쿠키 목록 확인
```
F12 → Application 탭 → Storage → Cookies → 사이트 선택
→ 우클릭 → Clear 로 쿠키 삭제 가능 (삭제 시 로그인 풀림!)
```

---

## 5. 세션 (Session)

### 정의

> **서버 측에서 생성되어 클라이언트와 서버 간의 상태를 유지하고, 상태 정보를 저장하는 데이터 저장 방식**

로그인 정보와 같은 **중요 데이터를 클라이언트가 아닌 서버 쪽에 저장**하고 유지하는 기술이다. 서버는 각 사용자를 구분하기 위해 **고유한 세션 ID를 발급**하고, 이 ID만 쿠키에 담아 클라이언트에 전송한다.

```
[쿠키만 사용하는 방식]
클라이언트 쿠키에 → user_id=123, name=김주우, 권한=admin ... 전부 저장 ⚠️

[세션 방식]
클라이언트 쿠키에 → session_id=abc123 (ID만!) 저장 ✅
서버에             → session_id=abc123 : {user_id=123, name=김주우 ...} 저장
```

> 💡 **열쇠와 자물쇠 비유**
> 실제 데이터(자물쇠)는 서버에만 있고, 클라이언트는 열쇠(session id)만 들고 다닌다.
> 요청할 때마다 열쇠를 제시하면 서버가 자물쇠를 열어 데이터를 확인한다.

---

### 세션 작동 원리 — 전체 흐름

```
[로그인 시]
① 클라이언트 → 서버 : 로그인 요청 (id/password)
② 서버 : 인증 성공 → session 데이터 생성 후 DB에 저장
③ 서버 → 클라이언트 : session id 만 응답 (데이터는 서버에, 열쇠만 전달)
④ 클라이언트 : session id 를 쿠키에 저장

[이후 모든 요청]
⑤ 클라이언트 → 서버 : 요청 + 쿠키(session id 포함) 자동 전송
⑥ 서버 : session id 확인 → "로그인된 사용자구나!" 식별 ✅
⑦ 서버 → 클라이언트 : 요청 처리 후 응답
```

> ✅ 쿠키는 요청 때마다 **자동으로** 서버에 전송되기 때문에, 사용자가 아무것도 하지 않아도 서버는 매 요청마다 session id를 확인해서 **로그인 상태를 계속 유지**할 수 있다.

---

### 세션 특징

- 서버의 **메모리나 데이터베이스**에 저장 → 서버 리소스를 사용하므로 효율적 관리 필요
- 쿠키에 session id를 저장하여 **매 요청마다 session id를 함께 전송**
- 세션은 **영구적으로 유지되지 않음** (브라우저 종료 or 만료 시간 도달 시 삭제)
- 중요한 데이터를 저장하므로 **보안을 신경써야 함**

> ⚠️ **세션 하이재킹 (Session Hijacking)**
> 공격자가 session id를 탈취하면, 해당 사용자인 것처럼 위장하여 서버에 접근 가능하다.
> 세션이 쿠키보다 안전하지만, **session id 자체가 탈취되면** 의미가 없어진다.
> 실제 서비스에서는 HTTPS 사용, 세션 만료 시간 설정, 로그인 시 세션 재발급 등의 추가 보안 조치를 함께 사용한다.

---

### 세션 in Django

> Django는 **`database-backed sessions`** 저장 방식을 기본값으로 사용

- 세션 정보는 DB의 **`django_session`** 테이블에 저장
- Django는 요청 안에 특정 session id를 포함하는 쿠키를 사용해서 각각의 브라우저와 연결된 session 데이터를 알아냄
- Django는 session 메커니즘(복잡한 동작원리)에 대부분을 생각하지 않게끔 **많은 도움을 줌** ✅

```
django_session 테이블
├── session_key   (세션 ID — 쿠키에 담겨 클라이언트에 전달)
├── session_data  (실제 데이터 — 암호화되어 저장)
└── expire_date   (만료 시간)
```

---

## 6. 쿠키 vs 세션

### 최종 비교

| | 쿠키 | 세션 |
|---|---|---|
| **저장 위치** | 클라이언트 (브라우저) | 서버 (메모리/DB) |
| **저장 내용** | 실제 데이터 | 실제 데이터 (session id만 쿠키에) |
| **보안** | 취약 (탈취/수정 가능) | 상대적으로 안전 (단, session id 탈취 주의) |
| **서버 부하** | 없음 | 있음 (서버 리소스 사용) |
| **만료** | 설정한 만료 시간 | 브라우저 종료 or 서버 설정 |
| **주요 용도** | 사용자 설정, 추적 | 로그인 상태 유지 |

### 공통 목적

> **클라이언트와 서버 간의 상태 정보를 유지하고, 사용자를 식별하기 위해 사용**

```
HTTP 는 기본적으로 Stateless (상태 없음)
        ↓
쿠키 등장 : 클라이언트가 데이터를 직접 들고 다님 (보안 취약)
        ↓
세션 등장 : 실제 데이터는 서버에, 열쇠(session id)만 쿠키에
        ↓
매 요청마다 session id 를 서버에 전달 → 로그인 상태 유지 ✅
```

> 결국 세션도 쿠키를 사용하긴 한다.
> 차이는 쿠키에 **실제 데이터**를 담느냐, **session id(열쇠)** 만 담느냐의 차이!

---

## 7. Django Authentication System

### 정의

> **Django에서 사용자 인증과 관련된 기능을 모아 놓은 시스템**

Django Authentication System을 활용하여 **로그인 / 로그아웃 / 회원가입 / 회원정보수정** 등 다양한 기능을 구현할 수 있다.

### 제공하는 핵심 기능

| 기능 | 설명 |
|---|---|
| **User Model** | 사용자 인증 후 연결될 User 모델 관리 |
| **Session 관리** | 로그인 상태를 유지하고 서버에 저장하는 방식을 관리 |
| **기본 인증 (ID/Password)** | 로그인 / 로그아웃 등 다양한 기능 제공 |

```python
# settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',        # ← 인증 담당 내장 앱!
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

> ✅ 단순히 로그인 여부만 확인하는 것을 넘어, **사용자별 또는 그룹별로 특정 행동에 대한 권한 부여**가 가능하다.

---

## 8. Custom User Model

### 기본 User Model의 한계

Django는 별도의 설정 없이 내장된 `auth` 앱의 User 클래스를 기본으로 사용한다. 하지만 제공되는 필드가 매우 제한적이고 (username, password, email 등), 개발자가 직접 수정하기 어렵다. 추가적인 사용자 정보 (생년월일, 주소, 나이 등)가 필요하다면 기본 User Model로는 한계가 있다.

### User Model 대체의 필요성

프로젝트의 **특정 요구사항에 맞춰** 사용자 모델을 확장할 수 있다.

| 상황 | 설명 |
|---|---|
| **필드 추가** | 이메일을 username으로 사용하거나, 생년월일/주소 등 추가 필드 포함 |
| **필드 제거** | `first_name`, `last_name` 처럼 불필요한 필드를 제거해 DB 모델을 간결하게 관리 |

> ⭐ Django는 새 프로젝트를 시작하는 경우, 비록 기본 User 모델이 충분하더라도 **커스텀 User 모델을 설정하는 것을 강력하게 권장**한다.
> 지금 당장 필요 없어도 만들어두자. 나중에 닉네임 등 필드를 추가하기 매우 쉬워진다!

---

### Abstract Base Classes (추상 기본 클래스)

몇 가지 공통 정보를 여러 다른 모델에 넣을 때 사용하는 클래스로, 데이터베이스 테이블을 만드는 데 사용되지 않고 다른 모델의 **기본 클래스로 사용**된다.

```
models.Model
      ↑
class AbstractBaseUser   ← 최소한의 인증 기능만 (비밀번호, last_login 등)
      ↑
class AbstractUser       ← 기본 User 모델의 모든 필드 포함 ✅
      ↑
class User               ← Django 기본 User (AbstractUser를 상속받음)
```

| 구분 | AbstractBaseUser | AbstractUser |
|---|---|---|
| **제공 필드** | 최소한의 인증 필드 (비밀번호, last_login 등) | 기본 User 모델의 모든 필드 (username, email 등) |
| **장점** | 최대의 유연성과 자유도 | 개발 속도가 빠르고 편리함 |
| **사용 케이스** | 전화번호 로그인 등 **완전히 새로운 인증 체계**를 만들 때 | 기존 인증 방식 유지하면서 **필드만 추가**하고 싶을 때 **(대부분의 경우)** |
| **비유** | 기본 피자 도우 | 토핑이 올려진 피자 |

> ✅ **대부분의 경우 → `AbstractUser` 상속**

---

## 9. Custom User Model 설정 순서

> ⚠️ **반드시 첫 migrate 실행 전에 완료해야 한다!**
> Django는 프로젝트 중간에 `AUTH_USER_MODEL` 변경을 **강력하게 권장하지 않음**
> 이미 진행 중인 프로젝트에서 변경해야 한다면 → **데이터베이스 초기화 후 진행**

### Step 1 — `accounts` 앱 생성

커스텀 User 모델을 담을 앱을 먼저 만든다.

```bash
python manage.py startapp accounts
```

> 💡 Django 내부적으로 auth와 관련한 경로나 키워드들을 `account` 라는 이름으로 사용하고 있기 때문에 **`accounts`** 로 생성하는 것을 공식 권장!
> 앱을 만들었으면 Django가 인식할 수 있도록 `settings.py` 에 등록해야 한다!

### Step 2 — `settings.py` 앱 등록

앱을 만들었어도 `INSTALLED_APPS` 에 등록하지 않으면 Django가 해당 앱을 인식하지 못한다.

```python
# settings.py
INSTALLED_APPS = [
    ...
    'accounts',    # ← 추가
]
```

> 💡 앱을 등록했으니, 이제 이 앱으로 들어오는 URL 경로를 프로젝트 URL에 연결해야 한다!

### Step 3 — `config/urls.py` URL 등록

`/accounts/` 로 시작하는 모든 요청을 `accounts` 앱의 `urls.py` 로 위임한다.

```python
# config/urls.py
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('articles/', include('articles.urls')),
    path('accounts/', include('accounts.urls')),    # ← 추가
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

> 💡 `accounts.urls` 로 위임했으니, 이제 `accounts/urls.py` 파일을 만들어야 한다!

### Step 4 — `accounts/urls.py` 기본 틀 작성

`accounts` 앱 내부의 URL을 관리할 파일을 만든다. 지금은 비워두고, 이후 로그인/로그아웃 등 경로를 추가할 예정이다.

```python
# accounts/urls.py
from django.urls import path
from . import views

app_name = 'accounts'
urlpatterns = [

]
```

> 💡 `app_name = 'accounts'` 를 설정해두면 템플릿에서 `{% url 'accounts:login' %}` 처럼 **앱 이름을 붙여서 URL을 구분**할 수 있다.
> URL 구조가 잡혔으니, 이제 핵심인 커스텀 User 모델을 만들어야 한다!

### Step 5 — `accounts/models.py` 커스텀 User 모델 작성

`AbstractUser` 를 상속받아 커스텀 User 모델을 정의한다. 지금은 `pass` 만 써도 기본 User 모델과 동일하게 작동한다.

```python
# accounts/models.py
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    pass
```

> ✅ 지금 당장 추가할 필드가 없어도 **커스텀 모델을 미리 만들어두는 것** 자체가 중요하다.
> 나중에 필드 추가가 필요할 때 여기에 바로 추가하면 된다.

```python
# 나중에 필드 추가 예시
class User(AbstractUser):
    bio = models.TextField(blank=True)
    birth_date = models.DateField(null=True, blank=True)
```

> 💡 모델을 만들었어도 Django는 아직 기본 `auth.User` 를 사용하고 있다.
> 우리가 만든 모델을 쓰도록 `settings.py` 에 명시적으로 알려줘야 한다!

### Step 6 — `settings.py` `AUTH_USER_MODEL` 설정

Django가 기본 User 모델 대신 우리가 만든 커스텀 User 모델을 사용하도록 설정한다.

```python
# settings.py 제일 아래에 추가
AUTH_USER_MODEL = 'accounts.User'
```

> **`AUTH_USER_MODEL`** : Django 프로젝트의 User를 나타내는데 사용하는 모델을 저장하는 속성

```
수정 전 기본값 : AUTH_USER_MODEL = 'auth.User'
수정 후        : AUTH_USER_MODEL = 'accounts.User'  ✅
```

> 💡 모델 설정이 끝났으니, 어드민 페이지에서도 커스텀 User 모델을 관리할 수 있도록 등록해야 한다!

### Step 7 — `accounts/admin.py` 커스텀 User 모델 등록

어드민 페이지에서 커스텀 User 모델을 관리할 수 있도록 등록한다.

```python
# accounts/admin.py
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
from .models import User

admin.site.register(User, UserAdmin)
```

> 💡 `UserAdmin` 을 함께 등록하면 비밀번호 변경, 권한 관리 등 User 관련 기능들이 어드민 페이지에서 제공된다.
> 이제 모든 설정이 끝났으니 DB에 반영하면 된다!

### Step 8 — migrate 실행

지금까지의 모델 변경사항을 DB에 반영한다.

```bash
python manage.py makemigrations
python manage.py migrate
```

**DB 테이블 변화**

```
변경 전 : auth_user       ← Django 기본 User 테이블
변경 후 : accounts_user   ← 커스텀 User 테이블 ✅
```

---

### Custom User Model 설정 체크리스트

**✅ 1. accounts 앱 생성**
```bash
python manage.py startapp accounts
```

**✅ 2. settings.py 앱 등록**
```python
INSTALLED_APPS = [
    ...
    'accounts',    # ← 추가
]
```

**✅ 3. config/urls.py URL 등록**
```python
urlpatterns = [
    ...
    path('accounts/', include('accounts.urls')),    # ← 추가
]
```

**✅ 4. accounts/urls.py 기본 틀 작성**
```python
from django.urls import path
from . import views

app_name = 'accounts'
urlpatterns = []
```

**✅ 5. accounts/models.py AbstractUser 상속**
```python
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    pass
```

**✅ 6. settings.py AUTH_USER_MODEL 설정**
```python
AUTH_USER_MODEL = 'accounts.User'
```

**✅ 7. accounts/admin.py 커스텀 User 모델 등록**
```python
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
from .models import User

admin.site.register(User, UserAdmin)
```

**✅ 8. migrate 실행**
```bash
python manage.py makemigrations
python manage.py migrate
```

---

## 10. Login 구현

### 로그인이란?

> **인증(id/password)을 완료하고, Session을 만들고 클라이언트와 연결하는 것**

```
① 클라이언트 → 서버 : id/password 전송 (로그인 요청)
② 서버 : id/password 검증 (로그인 인증)
③ 서버 : 인증 성공 → 세션 생성 후 DB에 저장
④ 서버 → 클라이언트 : 세션 ID를 쿠키에 담아 전달
```

---

### Step 1 — `accounts/urls.py`

`/accounts/login/` 경로로 요청이 들어오면 `views.login` 함수를 호출하도록 URL을 연결한다.

```python
from django.urls import path
from . import views

app_name = 'accounts'
urlpatterns = [
    path('login/', views.login, name='login'),
]
```

> 💡 `views.login` 을 호출하도록 했으니, 이제 `views.py` 에 `login` 함수를 만들어야 한다!

---

### Step 2 — `accounts/views.py`

URL에서 `views.login` 을 호출하도록 했으니, 실제 로그인 로직을 처리하는 함수를 작성한다.
GET 요청이면 빈 폼을 보여주고, POST 요청이면 입력값을 검증해 세션을 생성한다.

```python
from django.shortcuts import render, redirect
from django.contrib.auth import login as auth_login   # ← 이름 충돌 방지!
from django.contrib.auth.forms import AuthenticationForm

def login(request):
    if request.method == 'POST':
        form = AuthenticationForm(request, request.POST)
        # form = AuthenticationForm(request, data=request.POST)  ← 동일
        if form.is_valid():
            auth_login(request, form.get_user())
            return redirect('articles:index')
    else:
        form = AuthenticationForm()
    context = {
        'form': form,
    }
    return render(request, 'accounts/login.html', context)
```

> 💡 `render(request, 'accounts/login.html', context)` 로 폼을 템플릿에 넘겨주고 있으니,
> 이제 `accounts/login.html` 템플릿을 만들어야 한다!

---

### Step 3 — `accounts/templates/accounts/login.html`

views.py 에서 넘겨받은 `AuthenticationForm` 을 화면에 출력한다.
로그인 정보는 반드시 POST 방식으로 안전하게 전송해야 한다.

```html
{% extends 'base.html' %}

{% block content %}
  <h1>로그인</h1>
  <form action="{% url 'accounts:login' %}" method="POST">
    {% csrf_token %}
    {{ form }}
    <input type="submit">
  </form>
{% endblock %}
```

> 💡 템플릿에서 `{{ form }}` 으로 출력되는 게 바로 `AuthenticationForm` 이다.
> 이게 뭔지 좀 더 자세히 알아보자!

---

### `AuthenticationForm`

> **로그인 인증에 사용할 데이터를 입력받는 Django 빌트인 폼**

```python
from django.contrib.auth.forms import AuthenticationForm
```

| | 일반 ModelForm | AuthenticationForm |
|---|---|---|
| **상속** | `forms.ModelForm` | `forms.Form` |
| **역할** | DB에 데이터 저장 | 로그인 인증 (DB 저장 X) |
| **첫 번째 인자** | `request.POST` | `request` 자체를 받음 |

```python
# 일반 폼
form = ArticleForm(request.POST)

# AuthenticationForm ← request 가 첫 번째 인자로 추가!
form = AuthenticationForm(request, request.POST)
#                          ↑           ↑
#                      세션 접근    사용자 입력값 (username/password)
```

---

### `auth_login(request, user)`

> **AuthenticationForm을 통해 인증된 사용자를 로그인 하는 함수**

| 인자 | 역할 |
|---|---|
| **request** | 현재 사용자의 **세션 정보에 접근**하기 위해 사용 |
| **user** | 어떤 사용자가 로그인 되었는지를 **기록**하기 위해 사용 |

```python
# 왜 login as auth_login 으로 import 할까?
# 우리가 정의한 함수 이름도 login
# Django 내장 함수 이름도 login → 이름 충돌 방지를 위해 별칭 사용!
from django.contrib.auth import login as auth_login
```

### `get_user()`

> `AuthenticationForm` 의 인스턴스 메서드
> 유효성 검사를 통과했을 경우, **로그인 한 사용자 객체를 반환**

```python
# Django 내부 코드
def get_user(self):
    return self.user_cache   # 유효성 검사 시 캐시에 저장해둔 User 객체 반환
```

### 전체 흐름 요약

```
사용자가 username/password 입력 후 제출
        ↓
AuthenticationForm(request, request.POST) 으로 유효성 검사
        ↓
form.is_valid() → True
        ↓
form.get_user() → 인증된 User 객체 반환
        ↓
auth_login(request, user) → 세션 생성 + DB 저장 + 쿠키 전달
        ↓
redirect('articles:index') ✅
```

---

## 11. context_processors

> **템플릿이 렌더링 될 때 호출 가능한 컨텍스트 데이터 목록**

작성된 컨텍스트 데이터는 기본적으로 **템플릿에서 사용 가능한 변수로 포함**된다. Django에서 자주 사용하는 데이터 목록을 **미리 템플릿에 로드** 해 둔 것이다.

```python
# settings.py
TEMPLATES = [
    {
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',  # ← 핵심!
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

`django.contrib.auth.context_processors.auth` 덕분에 **views.py에서 따로 context에 넘겨주지 않아도** 모든 템플릿에서 자동으로 사용할 수 있는 변수가 생긴다.

| 변수 | 설명 |
|---|---|
| **`user`** | 현재 로그인한 사용자 객체 (`request.user`) |
| **`perms`** | 현재 사용자의 권한 목록 |

```html
<!-- views.py에서 따로 넘기지 않아도 템플릿에서 바로 사용 가능! -->
{{ user }}                   ← 로그인한 사용자 이름
{{ user.is_authenticated }}  ← 로그인 여부 (True/False)
```

---

## 최종 정리

```
HTTP 는 기본적으로 Stateless
        ↓
쿠키 등장 → Stateless 우회 (클라이언트가 상태를 들고 다님)
        ↓
세션 등장 → 실제 데이터는 서버에, session id 만 쿠키에
        ↓
Django Authentication System
  → Custom User Model (AbstractUser 상속)
  → AUTH_USER_MODEL 설정
  → AuthenticationForm + auth_login() 으로 로그인 구현
  → django_session 테이블에 세션 저장
  → context_processors 로 모든 템플릿에서 user 객체 사용 가능
```