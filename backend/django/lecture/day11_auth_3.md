# Django Authentication System 03

## 목차

1. [학습 목표](#1-학습-목표)
2. [내 비밀번호는 안전한가?](#2-내-비밀번호는-안전한가)
3. [회원정보 수정 (User Update)](#3-회원정보-수정-user-update)
4. [비밀번호 변경 (Password Change)](#4-비밀번호-변경-password-change)
5. [암호화의 중요성](#5-암호화의-중요성)
6. [우리가 사용하는 비밀번호는 어떻게 저장되고 있을까?](#6-우리가-사용하는-비밀번호는-어떻게-저장되고-있을까)
7. [해시 (Hash)](#7-해시-hash)
8. [해시 함수 (Hash Function)](#8-해시-함수-hash-function)
9. [Django의 비밀번호 저장 방식](#9-django의-비밀번호-저장-방식)
10. [SHA-256](#10-sha-256)
11. [레인보우 테이블 (Rainbow Table) 공격](#11-레인보우-테이블-rainbow-table-공격)
12. [솔트 (Salt)](#12-솔트-salt)
13. [무차별 대입 공격 (Brute-force Attack)](#13-무차별-대입-공격-brute-force-attack)
14. [키 스트레칭 (Key Stretching)](#14-키-스트레칭-key-stretching)
15. [Django에서의 비밀번호 암호화](#15-django에서의-비밀번호-암호화)
16. [비밀번호 암호화 정리](#16-비밀번호-암호화-정리)
17. [비밀번호 초기화 (Password Reset)](#17-비밀번호-초기화-password-reset)
18. [PasswordChangeForm의 인자 순서](#18-passwordchangeform의-인자-순서)

---

## 1. 학습 목표

| # | 키워드 | 설명 |
|---|--------|------|
| 1 | `UserChangeForm` | 상속받아 회원정보 수정 폼 만들기 |
| 2 | `PasswordChangeForm` | 비밀번호 변경 기능 구현 |
| 3 | `update_session_auth_hash` | 비밀번호 변경 후 세션 유지 |
| 4 | 단방향 해시 | 비밀번호 암호화의 필요성 이해 |
| 5 | Salt & Key Stretching | 암호화 강화 원리 이해 |
| 6 | Django 비밀번호 저장 방식 | 알고리즘, 솔트 등 이해 |
| 7 | Django 내장 auth URL | 비밀번호 초기화 구현 |

---

## 2. 내 비밀번호는 안전한가?

### 핵심 질문

> "우리가 만든 서비스가 해킹당해 DB가 통째로 유출된다면?"

개발자라면 반드시 고민해야 할 질문이다. 아무리 보안이 철저해도 **DB 유출은 언제든 일어날 수 있는 최악의 시나리오**고, 그 상황에서도 사용자를 보호할 수 있어야 한다.

### 비밀번호, 그냥 저장하면 안 되나?

우리가 입력한 비밀번호가 **평문(Plain Text)** 그대로 DB에 저장된다면?

```
# DB에 이렇게 저장된다면...
username: user1
password: mypassword123  ← 이게 그대로 보인다면?
```

→ DB가 유출되는 순간 **모든 사용자의 비밀번호가 즉시 노출**된다.

---

## 3. 회원정보 수정 (User Update)

### 핵심 개념

> **회원정보 수정 = User 객체를 Update 하는 과정**

1. 수정할 대상 **User 객체를 가져오고**
2. 입력받은 새로운 정보로 **기존 내용을 갱신**

GitHub의 Public Profile 페이지처럼, 이름/이메일 등을 수정할 수 있는 폼이 대표적인 예시다.

---

### UserChangeForm

**사용자 입력 데이터를 받는 built-in ModelForm**으로, 유효성 검사를 통과한 데이터로 기존 User 객체의 내용을 갱신하고 저장한다.

> ⚠️ **주로 관리자 페이지에서 사용**하도록 설계된 폼이다!

Django 내부 코드를 보면:

```python
class UserChangeForm(forms.ModelForm):
    password = ReadOnlyPasswordHashField(
        label=_("Password"),
        help_text=_(
            "Raw passwords are not stored, so there is no way to see "
            "the user's password."
        ),
    )

    class Meta:
        model = User        # 기본 auth.User (우리는 커스텀해서 accounts.User 사용!)
        fields = "__all__"  # 모든 필드 노출 (관리자용이라 그럼)
        field_classes = {"username": UsernameField}
```

| 항목 | 내용 |
|------|------|
| `model` | 기본은 `auth.User` → 우리는 **커스텀해서 `accounts.User` 사용** |
| `fields` | 기본값이 `"__all__"` → **그대로 쓰면 모든 필드 노출** (위험!) |
| `password` | `ReadOnlyPasswordHashField` → 해시된 비밀번호가 읽기 전용으로 표시됨 |

> 💡 그래서 `UserChangeForm`을 **그대로 쓰면 안 되고**, 반드시 **상속받아서 필요한 fields만 지정**해야 한다!
> `fields = "__all__"` 그대로면 권한, 그룹 등 민감한 필드까지 사용자한테 노출돼버린다.

---

### 구현

#### Step 1 — `accounts/urls.py` URL 등록

```python
# accounts/urls.py
app_name = 'accounts'

urlpatterns = [
    ...
    path('update/', views.update, name='update'),
]
```

#### Step 2 — `accounts/forms.py` CustomUserChangeForm 작성

```python
# accounts/forms.py
from django.contrib.auth.forms import UserCreationForm, UserChangeForm  # UserChangeForm 추가!
from django.contrib.auth import get_user_model


class CustomUserCreationForm(UserCreationForm):
    class Meta(UserCreationForm.Meta):
        model = get_user_model()


class CustomUserChangeForm(UserChangeForm):
    class Meta(UserChangeForm.Meta):
        model = get_user_model()
        fields = ['first_name', 'last_name', 'email']  # 노출할 필드만 지정!
```

> ⚠️ `fields` 를 꼭 지정해야 하는 이유!
> 지정하지 않으면 부모 클래스의 `fields = "__all__"` 이 그대로 상속되어
> 아래처럼 민감한 정보가 전부 노출된다.
>
> - Password 해시값 (`pbkdf2_sha256`, `salt`, `hash` 전부 노출)
> - Superuser status (권한을 사용자가 직접 수정 가능해져버림!)
> - Groups, Last login 등
>
> 일반 사용자가 자기 권한을 superuser로 바꿔버릴 수도 있는 **심각한 보안 문제**다.

#### Step 3 — `accounts/views.py` import 추가 및 update 뷰 작성

```python
# accounts/views.py
from django.contrib.auth.decorators import login_required
from .forms import CustomUserCreationForm, CustomUserChangeForm  # CustomUserChangeForm 추가!


@login_required
def update(request):
    if request.method == 'POST':
        form = CustomUserChangeForm(request.POST, instance=request.user)
        if form.is_valid():
            form.save()
            return redirect('articles:index')
    else:
        form = CustomUserChangeForm(instance=request.user)  # 기존 정보 채워서 열기
    context = {
        'form': form,
    }
    return render(request, 'accounts/update.html', context)
```

> 💡 `instance=request.user` 가 핵심!
>
> | 요청 | 설명 |
> |------|------|
> | **GET** | 현재 유저의 기존 정보가 **채워진 상태**로 폼 열기 |
> | **POST** | 제출된 데이터를 **기존 User 객체에 덮어써서** 저장 |
>
> `instance` 없으면 빈 폼이 열리고, 저장해도 새 객체가 생성돼버린다.

> ⚠️ `@login_required` 를 반드시 붙여야 한다!
> 비로그인 상태에서 접근하면 `request.user` 가 `AnonymousUser` 인데
> `CustomUserChangeForm(instance=request.user)` 에서 바로 에러가 터진다.
> ```
> AttributeError: 'AnonymousUser' object has no attribute '_meta'
> ```

#### Step 4 — `accounts/templates/accounts/update.html` 작성

```html
{% extends "base.html" %}

{% block content %}
  <h1>회원정보 수정</h1>
  <hr>
  <form action="{% url 'accounts:update' %}" method="POST">
    {% csrf_token %}
    {{ form }}
    <input type="submit" value="수정하기">
  </form>
{% endblock content %}
```

#### Step 5 — `articles/templates/articles/index.html` 링크 추가

```html
{% if request.user.is_authenticated %}
    ...
    <!-- 회원 탈퇴 -->
    <form action="{% url 'accounts:delete' %}" method="POST">
        {% csrf_token %}
        <input type="submit" value="회원탈퇴">
    </form>
    <!-- 회원 정보 수정 -->
    <a href="{% url 'accounts:update' %}">회원정보 수정</a>

{% else %}
    <a href="{% url 'accounts:login' %}">Login</a> |
    ...
{% endif %}
```

---

## 4. 비밀번호 변경 (Password Change)

### 핵심 개념

> **비밀번호 변경 = 인증된 사용자의 Session 데이터를 Update하는 과정**

- 기존 비밀번호를 통해 사용자를 인증하고
- 새로운 비밀번호를 **암호화하여 갱신**

---

### ⚠️ 암호 변경 시 세션 무효화 문제

비밀번호가 변경되면 기존 세션과의 회원 인증 정보가 일치하지 않게 되어 **로그인 상태가 유지되지 못하고 로그아웃 처리**된다.

```
비밀번호 변경 (form.save())
    ↓
DB의 password 해시값 변경
    ↓
기존 세션의 인증 정보와 불일치
    ↓
세션 무효화 → 강제 로그아웃 😱
```

이 문제를 해결하는 게 바로 **`update_session_auth_hash`** 다!

---

### update_session_auth_hash

> **암호 변경 시 세션 무효화를 막아주는 함수**

암호가 변경되면 새로운 password의 Session Data로 **기존 세션을 자동으로 갱신**해줘서 로그인 상태를 유지시켜준다.

```python
update_session_auth_hash(request, user)
#                         ↑ 요청   ↑ 변경된 유저 객체
```

---

### 구현

#### Step 1 — `accounts/urls.py` URL 등록

```python
# accounts/urls.py
app_name = 'accounts'

urlpatterns = [
    ...
    path('update/', views.update, name='update'),
    path('password/', views.password, name='password'),  # 추가!
]
```

> 💡 Django는 `UserChangeForm` 안에 비밀번호 변경 링크(`../password/`)를 자동으로 포함시켜준다.
> 현재 주소가 `accounts/update/` 라면 `../password/` → `accounts/password/` 로 이동하는 것!
> 그래서 이 URL을 직접 만들어줘야 한다.

#### Step 2 — `accounts/views.py` password 뷰 작성

```python
# accounts/views.py
from django.contrib.auth.forms import PasswordChangeForm
from django.contrib.auth import update_session_auth_hash


@login_required
def password(request):
    if request.method == 'POST':
        form = PasswordChangeForm(request.user, request.POST)  # user가 첫 번째 인자!
        if form.is_valid():
            form.save()
            update_session_auth_hash(request, form.user)  # 세션 갱신으로 로그아웃 방지!
            return redirect('articles:index')
    else:
        form = PasswordChangeForm(request.user)
    context = {
        'form': form,
    }
    return render(request, 'accounts/password.html', context)
```

> 💡 `PasswordChangeForm` 인자 순서 주의!
>
> ```python
> # POST일 때
> PasswordChangeForm(request.user, request.POST)
> #                  ↑ user 먼저   ↑ data 나중
>
> # GET일 때
> PasswordChangeForm(request.user)
> ```
>
> `ModelForm`이랑 순서가 반대다! ModelForm은 `(data, instance)` 순서지만
> `PasswordChangeForm`은 `(user, data)` 순서!

> ⚠️ `user` 인자를 빠뜨리면 바로 에러가 난다.
> ```
> TypeError: SetPasswordForm.__init__() missing 1 required positional argument: 'user'
> ```

#### Step 3 — `accounts/templates/accounts/password.html` 작성

```html
{% extends "base.html" %}

{% block content %}
  <h1>비밀번호 변경하기</h1>
  <hr>
  <form action="{% url 'accounts:password' %}" method="POST">
    {% csrf_token %}
    {{ form }}
    <input type="submit" value="변경하기">
  </form>
{% endblock content %}
```

---

## 5. 암호화의 중요성

> **많은 해킹사태가 발생하고 있고, 데이터가 유출되더라도 그 내용을 알 수 없도록 하는 암호화는 특히 중요하다**

### 실제 해킹 피해 사례

**사례 1 — 평문 저장**
시스템 관리 서버가 해킹당해 사용자들의 개인정보 **(ID, 비밀번호 등)가 평문으로 저장**되어 있었고, 동일한 정보로 다른 서버까지 연쇄 침투당한 사건

**사례 2 — BASE64 인코딩**
해커가 약 15만 건의 데이터를 탈취했고, 유출된 비밀번호 항목이 **BASE64 방식으로 저장**되어 있었다.
BASE64는 숫자나 기호를 사람이 읽기 쉬운 형태로 **단순히 바꾸는 방식**으로 누구나 쉽게 원래 내용으로 복원 가능 → **사실상 평문 저장이나 다름없음!**

---

## 6. 우리가 사용하는 비밀번호는 어떻게 저장되고 있을까?

### 방식 1 — 평문(Plain Text) 저장 ❌

> **사용자가 입력한 비밀번호를 그대로 저장하는 방식 → 보안에 매우 취약**

**문제점 3가지**

**① DB 해킹 시 즉시 노출**
데이터베이스가 해킹당하면 공격자는 아이디와 비밀번호 목록을 그대로 손에 넣게 되고, 이를 통해 **직접 로그인**하여 개인정보, 금융 정보, 주소록 등 모든 데이터를 유출하거나 서비스를 악용할 수 있다.

**② 내부자 위협**
악의적인 내부 직원이 데이터베이스에 접근하여 **모든 사용자의 비밀번호를 볼 수 있다.**

**③ Credential Stuffing 공격**
대부분의 사람들은 여러 서비스에서 **동일한 아이디와 비밀번호를 사용**하기 때문에, 탈취한 정보를 이용해 다른 사이트에 그대로 대입하여 **2차 피해를 발생**시킨다.

```
A 사이트 해킹
    ↓
id: user1 / pw: mypassword123 탈취
    ↓
B, C, D 사이트에도 동일하게 로그인 시도
    ↓
2차 피해 발생 😱 (Credential Stuffing)
```

---

### 방식 2 — 인코딩(Encoding) 후 저장 ❌

> **일정한 규칙에 따라 비밀번호를 알아볼 수 없는 문자로 '인코딩'한 후 저장 → 보안에 매우 취약**

> 💡 **인코딩(Encoding)이란?**
> 정보를 표현하는 형식을 다른 형식으로 변환하는 과정 (BASE64가 바로 이 방식!)

인코딩은 **비밀키 없이도** 정해진 규칙에 따라 누구나 **원래의 값으로 되돌릴 수 있다.**
공격자는 아주 간단한 디코딩 작업만으로 모든 사용자의 실제 비밀번호를 **즉시 알아낼 수 있다.**

→ 사실상 비밀번호를 **평문으로 저장하는 것과 동일한 수준의 위험**을 초래!

```
인코딩 (Base64 등)   →  규칙만 알면 누구나 디코딩 가능  ❌
암호화 (진짜 암호화)  →  비밀키 없으면 복호화 불가능      ✅
```

---

### 방식 3 — 해시(Hash) 후 저장 ✅

> **비밀번호를 복원이 불가능한 고정된 길이의 문자열로 변환 후 저장 → 보안에 필수**

- DB가 유출되어도 공격자는 복잡하게 얽힌 문자열을 보게 되고, **복원이 불가능**하기 때문에 실제 비밀번호를 알 수 없다.
- 악의적인 내부 직원이 비밀번호를 보더라도 암호화된 값만 보이므로 **실제 비밀번호를 유추할 수 없다.**

> 💡 **왜 고정된 길이의 문자열로 변환할까?**
>
> | 이유 | 설명 |
> |------|------|
> | **보안성** | 길이가 다르면 길이만 보고도 원래 비밀번호 길이를 유추할 수 있음 |
> | **일관성** | 길이가 동일해서 저장 공간 예측 및 설계가 쉽고, 검색/비교 처리 속도를 일정하게 유지 가능 |

---

## 7. 해시 (Hash)

> **임의의 크기를 가진 데이터를 고정된 크기의 고유한 값으로 변환하는 것**

### 믹서기 비유로 이해하기

> "해시는 데이터를 믹서기에 넣어 만든 주스와 같다."

| 특성 | 설명 |
|------|------|
| **단방향성** | 아무리 복잡한 데이터라도 한번 갈아버리면, **절대 다시 원래의 데이터로 되돌릴 수 없다** |
| **결정성** | 같은 재료를 넣으면 언제나 **똑같은 맛의 주스**가 나온다 (같은 입력 → 항상 같은 해시값) |

```
"mypassword"  →  [해시 함수]  →  a665a45920422f...  (복원 불가!)
"mypassword"  →  [해시 함수]  →  a665a45920422f...  (항상 동일!)
"mypasswor"   →  [해시 함수]  →  전혀 다른 값...     (한 글자만 달라도 완전히 다름)
```

### 해시의 두 가지 용도

1. **보안** → 원본을 알 수 없게 변환하여 비밀번호 보호
2. **무결성 검증** → 데이터가 변경되지 않았는지 확인 (파일 다운로드 후 해시값 비교 등)

---

## 8. 해시 함수 (Hash Function)

> **임의 길이 데이터를 입력받아 고정 길이(정수)로 변환해 주는 함수**
> 데이터를 넣으면 암호화된 문자열(해시값)을 뱉어내는 **'마법의 상자'**

### 해시 함수의 3가지 특성

| # | 특성 | 설명 |
|---|------|------|
| 1 | **고정된 길이** | 아무리 긴 소설책을 넣어도, 짧은 단어를 넣어도 결과물의 길이는 똑같다 |
| 2 | **단방향성 (One-way)** | 결과물만 보고 원본을 추측하는 것은 불가능하다 |
| 3 | **눈사태 효과 (Avalanche Effect)** | 입력값이 점 하나만 달라져도 결과물은 완전히 다른 값이 나온다 |

```
# 고정된 길이
"hi"                    →  2cf24db...  (64자리)
"안녕하세요 저는 주우입니다"  →  8f14e45...  (64자리)  ← 입력이 길어도 동일한 길이!

# 눈사태 효과
"password"   →  5e884898...
"Password"   →  e7cf3ef0...  ← P 하나 대문자로 바꿨을 뿐인데 완전히 다른 값!
```

---

## 9. Django의 비밀번호 저장 방식

> **Django는 기본적으로 SHA-256 해시 함수를 사용해서 암호화**

- 입력한 비밀번호의 길이와는 상관없이 **동일한 길이의 해시값** 생성
- 1글자만 다르더라도 **전혀 다른 해시값** 생성 (눈사태 효과!)

### 실제 DB 저장값 확인

| 입력 비밀번호 | DB에 저장된 값 |
|---|---|
| `1234` | `pbkdf2_sha256$600000$U7e5KT0ENtZKIA0xoWCWxs$cBWsUin9jHRdQppftp5pH06ltcWLZWeQajSZ54JjDIM=` |
| `1235` | `pbkdf2_sha256$600000$hLTECde6TpeEkv6qepn6d8$JyTHIuTIOuWh8S+KjRFwjM0l8s3qLoQjA6UpuunGH44=` |
| `12341234` | `pbkdf2_sha256$600000$32L7Li84kXRB6qtssWFV7e$TLni0ZiQAEigdxDJb/813mOWJGb3LGPg4L0yzO90ZgU=` |

> 💡 저장값 구조를 보면:
> ```
> pbkdf2_sha256 $ 600000 $ U7e5KT0ENtZKIA... $ cBWsUin9jHRd...
> ↑ 알고리즘      ↑ 반복횟수  ↑ Salt값               ↑ 해시값
> ```
> 단순 SHA-256이 아니라 **PBKDF2 + SHA-256 + Salt** 조합으로 저장되고 있다!

---

## 10. SHA-256

> **Secure Hash Algorithm - 256**
> 현재 가장 널리 사용되는 표준 암호화 해시 알고리즘

- 어떤 데이터를 넣더라도 항상 **256비트(64글자)** 의 결과물을 만들어낸다
- 현재 가장 **신뢰받는 알고리즘**으로, 비트코인의 핵심 기술이자 **전 세계 보안 표준**으로 사용된다

> 💡 얼마나 안전하냐면?
> 해킹하려면 **우주에 있는 모래알 수보다 많은 경우의 수**를 뚫어야 할 정도로 안전하다!

```
어떤 입력이든
    ↓
SHA-256
    ↓
항상 64글자 (256bit) 고정 출력
ex) 5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d
```

---

## 11. 레인보우 테이블 (Rainbow Table) 공격

### 해시만으로는 충분하지 않다?

비밀번호를 해시값으로 저장하면 안전해 보이지만, **공격자들은 해시값을 미리 계산해두는 방식으로 공격을 시도한다!**

> **이 방식이 바로 '레인보우 테이블' 공격**

### 레인보우 테이블이란?

> 공격자가 자주 사용되는 비밀번호들을 **미리 수백만, 수십억 개를 해시로 변환해 저장해 둔 거대한 정답지**

### 공격 방식

```
1. 공격자가 DB를 탈취해 사용자의 비밀번호 해시값을 얻음
        ↓
2. 해시값을 공격자 자신의 레인보우 테이블에서 검색
        ↓
3. 테이블에서 일치되는 값을 찾아내 비밀번호를 알아내는 데 성공
```

| 미리 계산된 비밀번호 | 미리 계산된 해시값 (SHA256) |
|---|---|
| 123456 | 8d969eef... |
| qwer1234 | 5e884898... |
| **iloveyou** | **1c8eda66...** |
| password1234 | 03ac6742... |

```
DB에서 훔친 해시값: 1c8eda66...
        ↓
레인보우 테이블에서 검색
        ↓
비밀번호: iloveyou  ← 역방향으로 되돌리지 않고도 알아냄!
```

> 💡 해시는 단방향이라 복호화가 불가능하지만
> 레인보우 테이블은 **복호화가 아니라 대조**하는 방식이라 단방향성을 우회할 수 있다!

---

## 12. 솔트 (Salt)

> **각 사용자마다 고유하게 생성된 임의의 문자열(솔트)을 비밀번호에 덧붙여서 해시값을 생성**
> 이 솔트(Salt)는 해시값과 함께 데이터베이스에 저장된다.

### Salt 미사용 vs 사용 비교

**Salt ❌ (취약)**

| 사용자 | 비밀번호 | 해시값 |
|--------|----------|--------|
| User A | qwer1234 | 5e884898... |
| User B | qwer1234 | 5e884898... |

→ 같은 비밀번호면 **해시값도 동일** → 레인보우 테이블에 한 번만 있으면 둘 다 뚫림!

**Salt ✅ (안전)**

| 사용자 | 비밀번호 | 솔트 | 해시값 |
|--------|----------|------|--------|
| User A | qwer1234 | !@#$ | a1b2c3d4... |
| User B | qwer1234 | %^&* | x7y8z9w0... |

→ 같은 비밀번호라도 **Salt가 달라서 해시값이 완전히 다름** ✅

### 💡 공격자가 Salt값을 알아도 되나요?

> Salt값을 알아도 상관없다!

```
Salt 없을 때  →  테이블 하나로 수백만 명 동시 공격 가능 😱
Salt 있을 때  →  사람 한 명 공격하려면 그 사람만을 위한 테이블을 새로 만들어야 함 😤
```

→ **'하나의 답안지로 수백만 명을 공격하는 방식'** 에서
**'한 명을 공격하기 위해 매번 새로운 답안지를 만드는 방식'** 으로 바뀌면서
**공격의 효율성을 극도로 떨어뜨린다!**

---

## 13. 무차별 대입 공격 (Brute-force Attack)

> **가장 원시적이지만 강력한 방법으로써 가능한 모든 비밀번호를 하나씩 대입하는 방식**
> 이 공격은 **시간과의 싸움**이며, 현대 컴퓨터의 **빠른 연산 속도**가 공격자의 무기가 된다.

Salt로 레인보우 테이블 공격을 막았지만, 공격자는 이제 Salt를 알고 있으니 **비밀번호만 하나씩 직접 대입**하는 방식으로 전환한다.

```
qwer1231 + !@#$  →  해시  →  z9x8c7v6...  ← 실패
qwer1232 + !@#$  →  해시  →  h5h3k2b5...  ← 실패
qwer1233 + !@#$  →  해시  →  skpsxkpw...  ← 실패
qwer1234 + !@#$  →  해시  →  a1b2c3d4...  ← 일치! 성공 😱
```

> 💡 최신 GPU는 **초당 약 1,500억 번 이상** 추측할 수 있어서 생각보다 훨씬 위협적이다!

---

## 14. 키 스트레칭 (Key Stretching)

> **솔트(Salt)를 적용한 해시 함수를 수만 ~ 수십만 번 반복하여 연산 시간을 의도적으로 늘리는 기법**

**결국 해결책은 "연산 속도를 늦추는 것"이 핵심!**
느리게 만들기 위해 **의도적으로 해시 연산을 수십만 번 반복**시켜, 공격 속도를 늦춘다.

```
일반 해시        →  1번 연산  →  즉시 완료  →  GPU 초당 1,500억 번 가능 😱
키 스트레칭 해시  →  60만 번 반복 연산  →  0.2초 소요  →  GPU 초당 5번만 가능 ✅
```

### 키 스트레칭 효과 비교

| 구분 | 키 스트레칭 없음 (1회) | 키 스트레칭 적용 (39만 회) |
|------|----------------------|--------------------------|
| 1개 비밀번호 검증 시간 | 0.000001초 | 0.1초 |
| **10억 개 비밀번호 검증** | **11.5일** | **약 3,170년** 😱 |

> 💡 **사용자 입장에서는 불편하지 않을까?**
>
> | 대상 | 체감 |
> |------|------|
> | **사용자** | 로그인 시 0.2초 → 실제로 **체감하지 못함** |
> | **공격자** | 초당 1,500억 번 → **초당 5번**으로 급감 → 사실상 공격 불가 |

### Django의 키 스트레칭 알고리즘

Django는 키 스트레칭을 구현하기 위해 **PBKDF2** 라는 검증된 알고리즘을 기본으로 사용한다.
최근에는 더 강력한 보안을 제공하는 **Argon2**, **bcrypt** 같은 알고리즘도 지원한다.

> 💡 PBKDF2, Argon2, bcrypt 알고리즘은 **키 스트레칭을 구현하는 다양한 방법** 정도로만 이해하기!

---

## 15. Django에서의 비밀번호 암호화

### How Django stores passwords

> Django는 유연한 비밀번호 저장 시스템을 제공하며 기본적으로 **PBKDF2**를 사용한다.
> User 객체의 `password` 속성은 아래 형식의 문자열로 저장된다.

```
<algorithm>$<iterations>$<salt>$<hash>
```

| 구성요소 | 설명 | 예시 |
|---------|------|------|
| `algorithm` | 어떤 알고리즘을 쓰는지 | `pbkdf2_sha256` |
| `iterations` | 키 스트레칭 횟수 | `600000` |
| `salt` | 생성된 솔트 | `U7e5KT0ENtZKIA...` |
| `hash` | 생성된 최종 해시 | `cBWsUin9jHRd...` |

### 실제 DB 저장값과 대조

```
pbkdf2_sha256 $ 600000 $ U7e5KT0ENtZKIA0xoWCWxs $ cBWsUin9jHRdQppftp5pH06l...
↑ algorithm    ↑ iterations  ↑ salt                   ↑ hash
```

### 지금까지 배운 보안 기법이 모두 담겨있다!

```
pbkdf2_sha256  →  SHA-256 해시 함수 사용
600000         →  키 스트레칭 (60만 번 반복)
U7e5KT0E...    →  솔트 (레인보우 테이블 방어)
cBWsUin9...    →  최종 해시값 (단방향, 복원 불가)
```

---

## 16. 비밀번호 암호화 정리

> **암호화 과정을 이해한 후, 검증된 프레임워크의 보안 기능을 신뢰하고 사용하기**

- 보안은 매우 어렵고 복잡한 분야이며, **Django 공식에도 가급적 재발명하지 않을 것을 권장**하고 있다.
- 단순히 코드를 복사해서 붙여넣는 것을 넘어, 이 기능이 **'왜'** 이렇게 만들어졌는지 이해하면 더 견고하고 안전한 애플리케이션을 만들 수 있다.

> 💡 **사용자는 우리의 서비스를 믿고 소중한 개인정보를 맡기고,**
> **그들의 데이터를 안전하게 지키는 것은 개발자의 '가장 기본적인 책임이자 직업윤리'**

### 전체 흐름 한눈에 보기

```
평문 저장                           ❌  →  DB 유출 시 즉시 노출
인코딩 (Base64)                     ❌  →  누구나 디코딩 가능, 사실상 평문과 동일
단순 해시                           ⚠️  →  레인보우 테이블 공격에 취약
해시 + Salt                         ⚠️  →  레인보우 테이블 방어, But Brute-force 취약
해시 + Salt + Key Stretching        ✅  →  현재 가장 안전한 방식
```

### Django가 채택한 방식

```
pbkdf2_sha256 $ 600000 $ Salt $ Hash
↑ SHA-256 해시  ↑ 60만번 반복   ↑ 사용자마다 고유  ↑ 복원 불가
                (Key Stretching)  (Salt)             (단방향 해시)
```

---

## 17. 비밀번호 초기화 (Password Reset)

> **비밀번호를 잊어버린 사용자가 이메일을 활용하여 비밀번호를 다시 설정하는 과정**

### 비밀번호 초기화 과정

```
1. 비밀번호를 찾으려고 하는 이메일 입력
        ↓
2. 이메일로 비밀번호 재설정 링크를 전송
        ↓
3. 비밀번호 재설정 페이지에서 새로운 비밀번호 설정
        ↓
4. 초기화 후 다시 로그인
```

---

### 이메일 입력 페이지를 직접 만들어야 할까? → No!

Django는 비밀번호에 관련된 **다양한 기능을 이미 내장**하고 있다.
`django.contrib.auth.urls` 를 include하면 아래 URL들이 자동으로 생성된다.

```python
# config/urls.py
urlpatterns = [
    path('admin/', admin.site.urls),
    path('articles/', include('articles.urls')),
    path('accounts/', include('accounts.urls')),
    path('accounts/', include('django.contrib.auth.urls')),  # 추가!
]
```

> 💡 **같은 `accounts/` prefix로 2개의 include를 써도 괜찮나요?**
> 같은 prefix로 여러 번 include를 해도, 각 include의 내부 URL 패턴을 **순차적으로 모두 시도**한다.
> **내부 URL이 겹치지 않으면 모두 동작**하기 때문에 괜찮다!
> `login`, `logout` 처럼 겹치는 경우엔 **우리가 만든 것이 먼저 매칭**돼서 우선 적용된다.

포함되는 URL 목록:

```
accounts/login/                           [name='login']
accounts/logout/                          [name='logout']
accounts/password_change/                 [name='password_change']
accounts/password_change/done/            [name='password_change_done']
accounts/password_reset/                  [name='password_reset']
accounts/password_reset/done/             [name='password_reset_done']
accounts/reset/<uidb64>/<token>/          [name='password_reset_confirm']
accounts/reset/done/                      [name='password_reset_complete']
```

> ⚠️ Django는 URL은 자동 제공하지만 **템플릿은 직접 만들어줘야 한다!**

---

### 이메일 전송 설정

실제 이메일을 보내려면 SMTP 서버 설정이 필요하다. 개발 환경에서는 **콘솔 이메일 백엔드**를 활용하면 된다.

```python
# config/settings.py
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
```

> 💡 이렇게 설정하면 실제 이메일 전송 없이
> **터미널에서 이메일 내용(재설정 링크 포함)을 바로 확인** 가능!
> 실제 서비스 배포 시엔 Gmail/SendGrid 등 실제 SMTP 서버로 교체해야 한다.

---

### 이메일 존재 여부를 알려주지 않는 이유

> ⚠️ 입력한 이메일에 매칭되는 사용자가 없어도 **동일한 "Password reset sent" 페이지**를 보여준다.

에러 메시지를 보여주면 공격자가 이메일 목록을 대입해서 **어떤 이메일이 우리 서비스에 가입되어 있는지 파악**할 수 있기 때문이다.

---

### 비밀번호 초기화 전체 흐름 정리

```
1. /accounts/password_reset/       → 이메일 입력
        ↓
2. /accounts/password_reset/done/  → 이메일 전송 완료 안내
        ↓
3. 콘솔에서 재설정 링크 확인
   /accounts/reset/<uidb64>/<token>/ → 새 비밀번호 입력
        ↓
4. /accounts/reset/done/           → 재설정 완료
        ↓
5. /accounts/login/                → 새 비밀번호로 로그인 ✅
```

> 💡 이 모든 과정이 **`django.contrib.auth.urls` include 한 줄**로 동작한다!

---

## 18. PasswordChangeForm의 인자 순서

### 왜 user 객체를 첫 번째 인자로 받을까?

> **부모 클래스인 `SetPasswordForm`의 생성자 함수 구성을 따르기 때문!**

### Form 상속 구조

```
Form
  └── ModelForm
        └── UserChangeForm      ← (data, instance) 순서
  └── Form
        └── SetPasswordForm     ← (user, data) 순서
              └── PasswordChangeForm  ← SetPasswordForm 구성 그대로 따름
```

| Form | 첫 번째 인자 | 이유 |
|------|------------|------|
| `ModelForm` 계열 (`UserChangeForm` 등) | `data` | ModelForm 기본 구성 |
| `SetPasswordForm` / `PasswordChangeForm` | `user` | 비밀번호 변경은 User 객체가 반드시 필요 |

> 💡 `PasswordChangeForm`은 `ModelForm`이 아닌 **일반 `Form`을 상속**받기 때문에
> 인자 순서가 다른 거다!

---

## 최종 정리

```
회원정보 수정
  → UserChangeForm을 그대로 쓰면 민감한 필드까지 전부 노출 → 반드시 상속 후 fields 지정
  → instance=request.user 로 기존 정보 채워서 폼 열기
  → @login_required 필수 (없으면 AnonymousUser 에러 발생)

비밀번호 변경
  → PasswordChangeForm(user, data) 순서 주의! (ModelForm과 반대)
  → form.save() 후 반드시 update_session_auth_hash(request, form.user) 호출
  → 안 하면 비밀번호 바꿀 때마다 강제 로그아웃 됨

비밀번호 암호화
  → 평문/인코딩 저장은 절대 안 됨
  → 해시(SHA-256) + Salt + Key Stretching 조합이 현재 가장 안전한 방식
  → Django는 pbkdf2_sha256$iterations$salt$hash 형식으로 자동 처리해줌

비밀번호 초기화
  → django.contrib.auth.urls include 한 줄로 URL 자동 생성
  → 템플릿은 직접 만들어야 함
  → 개발 환경에서는 EMAIL_BACKEND = console.EmailBackend 로 콘솔 확인 가능
```