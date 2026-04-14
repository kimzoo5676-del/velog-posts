# 1. 환경 설정
가상환경 생성 및 패키지 설치


```bash
cd django_hw_1_2
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```
`requirements.txt`가 있는 폴더 안에서 명령어를 실행해야 한다.

# 2. 서버 실행 확인

```bash
python manage.py runserver
```
`127.0.0.1:8000` 접속 시 로켓 화면이 뜨면 정상.

하지만 `127.0.0.1:8000/hello/` 접속하면 404 에러 발생.

# 3. 404 원인 파악
`my_pjt/urls.py`를 열면 이렇게 생겼다:

```python
urlpatterns = [
    path('admin/', admin.site.urls),
]
```
`/hello/` 경로가 등록되어 있지 않아서 404가 뜨는 것.

---

`views.py`에는 이미 `hello`함수가 작성되어 있는 상태.

```python
def hello(request):
    return render(request, 'hello.html')
```
# 4. urls.py 수정
urlpatterns에 한 줄 추가:

```python
urlpatterns = [
    path('admin/', admin.site.urls),
    path('hello/', views.hello),
]
```
다시 `127.0.0.1:8000/hello/`접속하면 정상 작동.

---

# 5. GitLab push 전 .gitignore 설정
프로젝트 루트(manage.py 있는 위치)에 .gitignore 생성:

```.gitignore
db.sqlite3
__pycache__/
*.pyc
.env
```
`venv/`는 같은 폴더 안에 없으면 안 써도 됨.

# 6. Git push
```bash
git add my_pjt/urls.py .gitignore
git commit -m "Add hello URL mapping and gitignore"
git push origin master
```
번외: venv 위치는 각 프로젝트 폴더 안에
venv는 내부에 절대 경로가 하드코딩되어 있어서 폴더를 옮기면 깨진다.

각 프로젝트 폴더 안에서 따로 생성하는 게 정석.


```
instroduction/
├── django_ws_1_1/
│   └── venv/
└── django_hw_1_2/
    └── venv/
```