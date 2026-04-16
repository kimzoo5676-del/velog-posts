## 목차

1. [HTML Form의 한계](#1-html-form의-한계)
2. [유효성 검사 (Validation)](#2-유효성-검사-validation)
3. [Django Form Class](#3-django-form-class)
4. [Form Class 정의 및 활용](#4-form-class-정의-및-활용)
5. [Widget](#5-widget)
6. [Form vs ModelForm](#6-form-vs-modelform)
7. [ModelForm](#7-modelform)
8. [ModelForm Class 정의](#8-modelform-class-정의)
9. [Meta Class](#9-meta-class)
10. [ModelForm을 적용한 create / update 로직](#10-modelform을-적용한-create--update-로직)
11. [save() 메서드](#11-save-메서드)
12. [두 함수를 하나로 통합 (리팩토링)](#12-두-함수를-하나로-통합-리팩토링)
13. [Widget attrs 설정](#13-widget-attrs-설정)
14. [필드를 수동으로 렌더링하기](#14-필드를-수동으로-렌더링하기)

---

## 1. HTML Form의 한계

지금까지 사용자로부터 데이터를 받을 때 HTML `<form>` 태그를 직접 작성했다.

```html
<form method="POST">
  <input type="text" name="title">
  <button type="submit">제출</button>
</form>
```

이 방식의 문제는 **서버 입장에서 들어오는 데이터를 그냥 믿어야 한다**는 것이다.

- 이메일 칸에 엉뚱한 값이 들어와도 알 수 없음
- 숫자만 받아야 하는 칸에 문자가 들어와도 알 수 없음
- 악의적인 사용자가 이상한 값을 넣어도 **필터링 불가**

즉, 비정상적이거나 악의적인 요청을 막을 수 없고, 유효한 데이터인지 확인하는 로직을 별도로 직접 구현해야 한다.

---

## 2. 유효성 검사 (Validation)

**유효성 검사**란 수집한 데이터가 정확하고 유효한지 확인하는 과정이다.

유효성 검사를 직접 구현하려면 고려해야 할 항목이 한두 가지가 아니다.

| 항목 | 예시 |
|------|------|
| 입력 값 | 빈 값인지, 타입이 맞는지 |
| 형식 | 이메일, 전화번호, 날짜 형식 등 |
| 중복 | 이미 존재하는 아이디/이메일인지 |
| 범위 | 나이가 0~150 사이인지, 가격이 음수가 아닌지 |
| 보안 | SQL Injection, XSS 공격 방어 등 |

이걸 전부 직접 개발하면 코드가 복잡해지고 실수할 가능성도 높아진다.  
그래서 **Django가 제공하는 Form 클래스**를 사용하면 이런 복잡한 과정을 대부분 자동으로 처리해준다.

> ☁️ **DevOps 연결** : 유효성 검사는 단순히 UX 문제가 아니라 **운영 DB 보호**의 문제다.  
> 잘못된 데이터가 DB에 쌓이면 서비스 장애로 이어질 수 있어서, 입력 검증은 배포 환경에서도 핵심 안전장치다.

---

## 3. Django Form Class

**Form Class**란 사용자 입력 데이터를 수집하고, 처리 및 유효성 검사를 수행하기 위한 도구다.

- 유효성 검사를 **단순화하고 자동화**하는 기능 제공
- 잘못 입력된 데이터는 **자동으로 오류 처리** → 안전성 향상
- 빠르고 **일관된 입력 검증 기능** 구현 가능

> 💡 HTML `<form>`이 입력창을 **보여주는** 역할이라면,  
> Django Form은 데이터를 **받고 + 검증하고 + 오류를 돌려주는** 역할까지 담당한다.

---

## 4. Form Class 정의 및 활용

### forms.py 파일 생성

Django는 `forms.py` 파일을 자동으로 생성해주지 않는다.  
`models.py`, `views.py`는 앱 생성 시 자동으로 만들어지지만, `forms.py`는 **직접 생성**해야 한다.

```
articles/
├── models.py
├── views.py
├── forms.py   ← 직접 생성!
├── urls.py
└── templates/
```

### Form Class 정의

```python
# articles/forms.py
from django import forms

class ArticleForm(forms.Form):
    title = forms.CharField(max_length=10)
    content = forms.CharField()
```

`models.py`와 구조가 매우 유사하지만 **목적이 다르다.**

| | models.py | forms.py |
|---|---|---|
| 목적 | DB 테이블 정의 | 입력 데이터 검증 |
| 상속 | `models.Model` | `forms.Form` |
| 결과 | DB에 저장 | 유효성 검사 후 데이터 반환 |

### views.py에서 활용

**Form 클래스 도입 전:**

```python
# articles/views.py

def new(request):
    return render(request, 'articles/new.html')
    # 그냥 빈 템플릿만 렌더링
    # 입력 데이터 검증? → 없음
    # 오류 메시지 전달? → 없음
```

**Form 클래스 도입 후:**

```python
# articles/views.py
from .forms import ArticleForm

def new(request):
    form = ArticleForm()   # 빈 폼 인스턴스 생성
    context = {
        'form': form,
    }
    return render(request, 'articles/new.html', context)
```

| | 도입 전 | 도입 후 |
|---|---|---|
| 템플릿 전달 | 없음 | `ArticleForm()` 인스턴스 전달 |
| 유효성 검사 | ❌ 직접 구현 필요 | ✅ 자동 |
| 오류 메시지 | ❌ 없음 | ✅ form 안에 자동으로 담김 |
| HTML 입력창 생성 | 템플릿에 직접 작성 | `{{ form }}` 한 줄로 자동 생성 |

> 💡 **핵심 변화**  
> 단순히 "화면만 보여주는" 함수에서  
> "화면도 보여주고 + 데이터 검증도 하고 + 오류도 돌려주는" 함수로 진화한 것이다!

- `ArticleForm()` — 빈 폼 인스턴스 (사용자에게 보여줄 빈 입력창)
- `ArticleForm(request.POST)` — 사용자가 제출한 데이터가 담긴 폼 (create에서 사용)

### new.html 템플릿 (수정 필요 action 비어있을 때 동작원리)

```html
<h1>NEW</h1>
<form action="{% url 'articles:create' %}" method="POST">
  {% csrf_token %}
  {{ form }}
  <input type="submit">
</form>
```

`{{ form }}` 한 줄이 아래의 복잡한 HTML을 **자동으로 대체**한다.

```html
<!-- {{ form }}이 자동으로 생성해주는 HTML -->
<div>
    <label for="id_title">Title:</label>
    <input type="text" name="title" maxlength="10" required id="id_title">
</div>
<div>
    <label for="id_content">Content:</label>
    <input type="text" name="content" required id="id_content">
</div>
```

Django가 자동으로 해주는 것들:

| 항목 | 설명 |
|------|------|
| `<label>` | 필드 이름 자동 생성 |
| `name="title"` | 필드명 자동 설정 |
| `maxlength="10"` | `forms.py`의 `max_length=10` 자동 반영 |
| `required` | 필수 입력 속성 자동 추가 |
| `id="id_title"` | id 자동 생성 |

---

## 5. Widget

### Widget이란?

**Widget**이란 폼 필드를 화면에 표시하는 HTML 입력 요소를 정의하는 구성 요소다.  
쉽게 말하면 **"이 필드를 HTML에서 어떤 태그로 보여줄지"** 결정하는 것이다.

> Widget은 단순히 input 요소의 속성 및 출력되는 부분을 변경하는 것  
> (데이터 검증과는 무관하다!)

### 대표적인 Widget 종류

| Widget | HTML 태그 | 화면에 보이는 모습 | 주요 용도 |
|--------|------------|-------------------|-----------|
| `TextInput` | `<input type="text">` | `[ 한 줄 입력창 ]` | 제목, 이름 등 짧은 텍스트 (CharField 기본값) |
| `Textarea` | `<textarea>` | 여러 줄 입력 가능한 큰 박스 | 게시글 본문, 댓글 등 긴 텍스트 |
| `Select` | `<select>` | 드롭다운 메뉴 ▼ | 카테고리, 성별 등 선택지가 정해진 경우 |
| `CheckboxInput` | `<input type="checkbox">` | ☑ 체크박스 | 동의 여부, 다중 선택 |
| `PasswordInput` | `<input type="password">` | `[ ●●●●●● ]` 입력값 숨김 | 비밀번호 입력 |

```python
# 각 Widget 적용 예시
class ExampleForm(forms.Form):
    title    = forms.CharField()                          # [ 한 줄 입력창 ]
    content  = forms.CharField(widget=forms.Textarea)     # 여러 줄 큰 박스
    category = forms.ChoiceField(widget=forms.Select)     # 드롭다운 ▼
    agree    = forms.BooleanField(widget=forms.CheckboxInput)  # ☑ 체크박스
    password = forms.CharField(widget=forms.PasswordInput)     # ●●●●●●
```

### Widget 사용 예시

`forms.CharField`의 기본 위젯은 `TextInput`이라 `<input type="text">`로 렌더링된다.  
게시글 본문처럼 여러 줄 입력이 필요한 경우 `Textarea` 위젯을 명시적으로 지정해야 한다.

```python
# articles/forms.py
from django import forms

class ArticleForm(forms.Form):
    title = forms.CharField(max_length=10)
    content = forms.CharField(widget=forms.Textarea)  # ← 위젯 지정!
```

---

## 6. Form vs ModelForm

| | Form | ModelForm |
|---|---|---|
| 용도 | 입력 데이터를 **DB에 저장 안 할 때** | 입력 데이터를 **DB에 저장할 때** |
| 예시 | 검색, 로그인 | 게시글 작성, 회원가입 |
| 모델 연결 | ❌ | ✅ |
| 필드 자동 생성 | ❌ (직접 정의) | ✅ (모델에서 자동) |

> 💡 **로그인**은 DB에 저장하는 게 아니라 DB에서 꺼내서 **비교**하는 것이라 Form을 쓴다.  
> 헷갈리기 쉬운 부분이므로 주의!

---

## 7. ModelForm

**ModelForm**이란 Model과 연결된 Form을 자동으로 생성해주는 기능이다.

일반 Form으로 게시글 작성 폼을 만들면 아래처럼 중복이 발생한다.

```python
# models.py
class Article(models.Model):
    title = models.CharField(max_length=10)
    content = models.TextField()

# forms.py (일반 Form)
class ArticleForm(forms.Form):
    title = forms.CharField(max_length=10)   # 중복!
    content = forms.CharField(widget=forms.Textarea)  # 중복!
```

models.py와 forms.py에 **같은 필드 정보가 중복**으로 존재한다.  
ModelForm을 쓰면 이 문제가 해결된다.

| | 일반 Form | ModelForm |
|---|---|---|
| 필드 정의 | 직접 하나하나 작성 | 모델에서 **자동 생성** |
| DB 저장 | 직접 구현 | `.save()` 한 줄로 끝 |
| 중복 코드 | models.py랑 중복 발생 | 중복 없음 |

---

## 8. ModelForm Class 정의

```python
# articles/forms.py
from django import forms
from .models import Article

class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'content']
```

### 일반 Form과 비교

```python
# 전 (일반 Form)
class ArticleForm(forms.Form):
    title = forms.CharField(max_length=10)
    content = forms.CharField(widget=forms.Textarea)

# 후 (ModelForm)
class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'content']
```

필드 정의가 **통째로 사라졌다!** 모델에서 자동으로 가져오기 때문이다.  
`Meta` 클래스가 어떤 역할을 하는지는 다음 섹션에서 자세히 다룬다.

---

## 9. Meta Class

**Meta Class**란 ModelForm의 정보를 작성하는 곳이다.

ModelForm 내부에서 **어떤 모델과 연결할지, 어떤 필드를 사용할지** 등을 정의하는 설정 공간이며, 폼의 동작 방식을 제어하는 핵심 역할을 한다.

```python
class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article                # 어떤 모델과 연결할지
        fields = ['title', 'content']  # 어떤 필드를 사용할지
```

> 💡 **왜 클래스 안에 클래스가 있을까?**  
> Meta 클래스는 `ArticleForm` 자체의 동작 로직이 아니라, `ArticleForm`에 대한 **설정 정보**를 담는 공간이다.  
> Django가 내부적으로 Meta 클래스를 읽어서 폼을 자동 생성하는 구조이므로, 파이썬의 inner class 같은 **문법적 관점으로 접근하지 말 것!**  
> Django ModelForm에서 설정 정보를 담는 **약속된 공간**으로 이해하면 된다.

### fields 및 exclude 속성

```python
class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article

        # 방법 1. 포함할 필드 직접 지정 (권장)
        fields = ['title', 'content']   # 리스트
        fields = ('title', 'content')   # 튜플
        fields = '__all__'              # 전체 필드

        # 방법 2. 제외할 필드 지정
        exclude = ('title',)   # title만 제외 → content만 폼에 표시됨
```

| | 설명 | 예시 |
|---|---|---|
| `fields` | **포함할** 필드를 명시 | `['title', 'content']` |
| `exclude` | **제외할** 필드를 명시 | `('title',)` → title 빼고 나머지 전부 |

> 💡 `fields`와 `exclude`는 동시에 사용하지 않는다.  
> 포함할 필드가 적으면 `fields`, 제외할 필드가 적으면 `exclude`를 선택!

---

## 10. ModelForm을 적용한 create / update 로직

### CREATE vs UPDATE 흐름 비교

| | Create | Update |
|---|---|---|
| 차이 | 데이터가 기존에 **없음** → 빈 폼 | 데이터를 DB에서 **꺼내오는 작업 필요** → 채워진 폼 |
| 폼 생성 | `ArticleForm()` | `ArticleForm(instance=article)` |

### create 뷰

```python
def create(request):
    form = ArticleForm(request.POST)  # 사용자 입력 데이터를 폼에 바인딩
    if form.is_valid():               # 유효성 검사 통과 시
        article = form.save()         # DB 저장 후 인스턴스 반환
        return redirect('articles:detail', article.pk)
    # 유효성 검사 실패 시 → 에러 메시지가 담긴 form을 다시 렌더링
    context = {
        'form': form,
    }
    return render(request, 'articles/create.html', context)
```

```
사용자 입력 → is_valid()
    통과 ✅  →  form.save()  →  DB 저장  →  detail로 redirect
    실패 ❌  →  에러 메시지 포함된 form으로 create.html 다시 렌더링
```

> 💡 `else` 없이도 동작하는 이유!  
> `is_valid()`가 True면 redirect로 함수가 끝나버리고,  
> False면 if 블록을 건너뛰고 아래 render로 자연스럽게 내려오는 구조다.

### edit 뷰

```python
def edit(request, pk):
    article = Article.objects.get(pk=pk)    # 기존 데이터 DB에서 꺼내옴
    form = ArticleForm(instance=article)    # 기존 data를 폼에 채워줌
    context = {
        'article': article,
        'form': form,
    }
    return render(request, 'articles/update.html', context)
```

`instance=article` 하나로 기존에 템플릿에서 직접 하던 작업을 대체한다.

```html
<!-- 전 (ModelForm 없이) - 템플릿에서 직접 기존값 채워줘야 했음 -->
<input type="text" name="title" value="{{ article.title }}">
<textarea name="content">{{ article.content }}</textarea>

<!-- 후 (ModelForm) -->
{{ form }}   <!-- 기존값까지 자동으로 채워진 채로 렌더링! -->
```

### update 뷰

```python
def update(request, pk):
    article = Article.objects.get(pk=pk)
    form = ArticleForm(request.POST, instance=article)  # create와 딱 한 줄 차이!
    if form.is_valid():
        article = form.save()
        return redirect('articles:detail', article.pk)
    context = {
        'article': article,
        'form': form,
    }
    return render(request, 'articles/update.html', context)
```

create vs update는 `instance=article` 유무가 전부다.

```python
# create
form = ArticleForm(request.POST)

# update
form = ArticleForm(request.POST, instance=article)
#                                ↑ 이것만 추가!
```

---

## 11. save() 메서드

**save()** 란 데이터베이스 객체를 만들고 저장하는 ModelForm의 인스턴스 메서드다.

- 폼 데이터가 유효한 경우 `save()`를 호출하면 모델 인스턴스를 생성하고 DB에 저장
- `instance` 인자를 통해 새 객체 생성과 기존 객체 수정을 구분
- 추가 코드 없이 손쉽게 DB 연동 가능

```python
# instance 없음 → 새 객체 생성 (INSERT)
form = ArticleForm(request.POST)
article = form.save()

# instance 있음 → 기존 객체 수정 (UPDATE)
form = ArticleForm(request.POST, instance=article)
article = form.save()
```

| 상황 | SQL |
|---|---|
| `instance` 없음 | `INSERT INTO articles_article ...` |
| `instance` 있음 | `UPDATE articles_article SET ... WHERE id=pk` |

> 💡 `form.save()`는 저장된 인스턴스를 **반환**하기 때문에 `article = form.save()`로 받아서 `article.pk`를 redirect에 활용할 수 있다!

---

## 12. 두 함수를 하나로 통합 (리팩토링)

HTTP request method 차이점을 활용해 동일한 목적을 가지는 2개의 view 함수를 하나로 구조화할 수 있다.

```
전                          후
─────────────────────────────────────
def new      (GET)   ┐
def create   (POST)  ┘  →  def create (GET/POST 통합)

def edit     (GET)   ┐
def update   (POST)  ┘  →  def update (GET/POST 통합)

new.html              →  create.html
edit.html             ┐
update.html           ┘  →  update.html 하나로
```

### create 통합

```python
def create(request):
    if request.method == 'POST':
        # POST → 데이터 처리 + 저장
        form = ArticleForm(request.POST)
        if form.is_valid():
            article = form.save()
            return redirect('articles:detail', article.pk)
    else:
        # GET → 빈 폼 보여주기
        form = ArticleForm()
    context = {
        'form': form,
    }
    return render(request, 'articles/create.html', context)
```

### update 통합

```python
def update(request, pk):
    article = Article.objects.get(pk=pk)
    if request.method == 'POST':
        # POST → 데이터 처리 + 저장
        form = ArticleForm(request.POST, instance=article)
        if form.is_valid():
            article = form.save()
            return redirect('articles:detail', article.pk)
    else:
        # GET → 기존 데이터 채워진 폼 보여주기
        form = ArticleForm(instance=article)
    context = {
        'article': article,
        'form': form,
    }
    return render(request, 'articles/update.html', context)
```

> 💡 `request.method`는 현재 요청이 GET인지 POST인지 **문자열**로 담겨있다.  
> 비교할 때 반드시 **대문자** `'POST'`, `'GET'`으로 써야 한다!

> ☁️ **DevOps 연결** : POST 후 redirect하는 PRG(Post/Redirect/Get) 패턴은 웹 표준이다.  
> 중복 요청을 방지해서 서버 부하를 줄이고, 운영 환경에서 DB 중복 데이터 생성을 막는 안전장치가 된다.

> 💡 **render vs redirect 구분법**  
> `render` → 화면(파일)을 그려주는 함수 → 파일 경로 → `/`  
> `redirect` → URL로 보내는 함수 → URL 이름 → `:`  
> 함수의 목적을 기억하면 자연스럽게 연상된다!

---

## 13. Widget attrs 설정

Widget에 `attrs`를 사용하면 HTML 속성을 직접 지정할 수 있다.

```python
# forms.py
class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'content']
        widgets = {
            'title': forms.TextInput(attrs={
                'class': 'my-title',
                'placeholder': '제목을 입력하세요',
            }),
            'content': forms.Textarea(attrs={
                'class': 'my-content',
                'rows': 5,
            }),
        }
```

| attrs 키 | HTML 결과 |
|---|---|
| `'class': 'my-title'` | `class="my-title"` |
| `'placeholder': '...'` | `placeholder="..."` |
| `'rows': 5` | `rows="5"` |

> 💡 Bootstrap 같은 CSS 프레임워크 사용 시 `class` 속성을 넣어야 하는 경우가 많아서 실무에서 자주 활용된다!

> ☁️ **DevOps 연결** : CSS 프레임워크를 Widget attrs로 연결하면 프론트엔드 배포 시 디자인 일관성을 코드 레벨에서 관리할 수 있다.  
> 나중에 Nginx + 정적 파일 서빙 환경에서 Bootstrap CDN과 함께 쓰는 패턴이 자주 등장한다.

---

## 14. 필드를 수동으로 렌더링하기

`{{ form }}` 한 줄로 전체를 자동 렌더링하는 대신, 필드를 **하나씩 직접 제어**할 수도 있다.

```html
{{ form.non_field_errors }}   <!-- 폼 전체에 관련된 에러 메시지 -->

<form action="..." method="POST">
  {% csrf_token %}
  <div>
    {{ form.title.errors }}                              <!-- title 필드 에러 메시지 -->
    <label for="{{ form.title.id_for_label }}">Title:</label>
    {{ form.title }}                                     <!-- title 입력창 -->
  </div>
  <div>
    {{ form.content.errors }}                            <!-- content 필드 에러 메시지 -->
    <label for="{{ form.content.id_for_label }}">Content:</label>
    {{ form.content }}                                   <!-- content 입력창 -->
  </div>
  <input type="submit">
</form>
```

| | 자동 `{{ form }}` | 수동 렌더링 |
|---|---|---|
| 코드량 | 한 줄 | 필드마다 직접 작성 |
| 레이아웃 제어 | ❌ Django가 알아서 | ✅ 내가 원하는 대로 |
| 에러 위치 | 자동 | 직접 지정 가능 |

> 💡 Bootstrap 같은 CSS 프레임워크로 커스텀 디자인할 때 수동 렌더링을 주로 사용한다!

---

## 최종 정리

```
HTML Form → 유효성 검사 없음, 직접 구현 필요
    ↓
Django Form → 유효성 검사 자동화, BUT 모델과 중복 발생
    ↓
Django ModelForm → 모델 기반 폼 자동 생성, 중복 제거, .save()로 DB 저장까지!
```

| 개념 | 역할 |
|---|---|
| `forms.py` | Form/ModelForm 클래스 정의 공간 |
| `is_valid()` | 유효성 검사 (True/False) |
| `form.save()` | DB 저장 (INSERT or UPDATE) |
| `instance=article` | 기존 데이터 연결 (edit/update) |
| `Widget` | 필드의 HTML 렌더링 방식 결정 |
| `Meta class` | ModelForm 설정 정보 (model, fields) |

오늘 수업의 핵심은 **ModelForm**이다.  
모델이 이미 정의되어 있다면 굳이 폼 필드를 다시 정의할 필요 없이, ModelForm이 모델을 보고 폼을 자동으로 만들어준다.  
덕분에 코드 중복이 줄고, `.save()` 한 줄로 DB 저장까지 처리할 수 있다.

---

## ☁️ DevOps 연결 포인트

| Django 개념 | DevOps 연결 |
|-------------|-------------|
| 유효성 검사 (is_valid) | 잘못된 데이터가 DB에 저장되지 않도록 보호 → **데이터 무결성** 유지, 운영 DB 안정성과 직결 |
| CSRF 토큰 | 보안 미들웨어 개념 → **API Gateway 인증**, 토큰 기반 인증(JWT)의 기초 |
| PRG 패턴 (redirect) | POST → Redirect → GET 웹 표준 패턴 → **중복 요청 방지**, 서버 부하 관리 |
| ModelForm 리팩토링 | 코드 중복 제거 → **유지보수성 향상** → CI/CD 파이프라인에서 코드 품질 관리와 연결 |
| request.method 분기 | GET/POST 역할 분리 → **REST API 설계 원칙**의 기초, 나중에 DRF(Django REST Framework)로 이어짐 |
| Widget attrs (class 속성) | Bootstrap 등 CSS 프레임워크 연동 → **프론트엔드 배포 환경**에서 UI 일관성 유지 |

> 💡 **보안 관점 핵심**  
> `is_valid()` → 브라우저 레벨(required) + 서버 레벨(Django) 두 겹으로 검증  
> `{% csrf_token %}` → POST 요청에 반드시 토큰 포함  
> 이 두 가지가 실제 서비스 배포 시 기본 보안 레이어가 된다!

> 💡 **REST API 연결 포인트**  
> 지금은 `new/`, `create/`, `edit/`, `update/` URL이 전부 따로 존재하지만,  
> REST API 방식으로 가면 URL은 `/articles/`, `/articles/1/` 로 통일하고  
> HTTP Method(GET/POST/PUT/DELETE)로 동작을 구분하는 구조가 된다.  
> 오늘 배운 `request.method` 분기가 그 개념의 출발점이다!