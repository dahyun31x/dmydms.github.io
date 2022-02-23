# [Writing your first Django app, part 2](https://docs.djangoproject.com/en/4.0/intro/tutorial02/)

## [Database setup](https://docs.djangoproject.com/en/4.0/intro/tutorial02/#database-setup)

- Django 설정들을 보여주는 module-level 변수들을 담고있는 `mysite/settings.py` 열기
- default로 SQLite 설정을 사용하며, SQLite는 Python에 포함되어 있기 때문에, 설치가 필요 없으며 가장 쉬운 방법. 하지만 PostgreSQL 같은 확장 가능한 DB를 사용해 DB 전환 문제를 피할 수 있음.
- 다른 DB를 사용하고 싶다면 적절한 [database bindings](https://docs.djangoproject.com/en/4.0/topics/install/#database-installation)을 사용하고 [DATABASES](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-DATABASES) 'default' 아이템에 있는 다음 키들을 선택한 database 연결 settings에 맞추기.
  - `ENGINE`:
    - 'django.db.backends.postgresql'
    - 'django.db.backends.sqlite3'
    - 다른 DB 옵션은 공식 문서 참조.
  - `NAME`: database의 이름. SQLite를 쓰면 이 이름의 파일이 생성됨. (SQLite 내용이라 생략)
  - SQLite 이외의 DB를 사용할거라면, USER, PASSWORD 그리고 HOST가 추가되어야하며 자세한 내용은 [DATABASES](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-DATABASES) 참고.

---

cf. SQLite가 아닌 database를 사용하는 경우.

- SQLite가 아닌 database를 사용한다면, 이 시점에서 `CREATE DATABASE database_name`을 이용해서 DB를 생성.
- `mysite/settings.py`에 작성되는 DB 유저는 "create database" 권한이 있어야함. 권한이 있어야만, 이후 튜토리얼에서 필요한 [test database](https://docs.djangoproject.com/en/4.0/topics/testing/overview/#the-test-database) 자동 생성이 허용됨.

---  

- `mysite/settings.py`의 [TIME_ZONE](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-TIME_ZONE)을 변경하기. 
    > 이해 중 🤔  
    >powered sites, each with a separate time zone setting.
    >
    >When USE_TZ is False, this is the time zone in which Django will store all datetimes. When USE_TZ is True, this is the default time zone that Django will use to display datetimes in templates and to interpret datetimes entered in forms.

  - ```python
    TIME_ZONE = 'Asia/Seoul'
    USE_TZ = False
    ```

- [`INSTALLED_APPS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-INSTALLED_APPS) 는 settings.py의 상단에 있는 설정으로 Django instance에서 모든 Django application들의 이름을 활성화함. 앱들은 여러개의 프로젝트에서 사용될 수 있고 다른 프로젝트에서 사용하기 위해서 앱들을 package하거나 분리할 수 있음.
- default로 [`INSTALLED_APPS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-INSTALLED_APPS) 다음 앱을 포함 (TODO: 들여다보기):
  - [`django.contrib.admin`](https://docs.djangoproject.com/en/4.0/ref/contrib/admin/#module-django.contrib.admin) – The admin site. You’ll use it shortly.
  - [`django.contrib.auth`](https://docs.djangoproject.com/en/4.0/topics/auth/#module-django.contrib.auth) – An authentication system.
  - [`django.contrib.contenttypes`](https://docs.djangoproject.com/en/4.0/ref/contrib/contenttypes/#module-django.contrib.contenttypes) – A framework for content types.
  - [`django.contrib.sessions`](https://docs.djangoproject.com/en/4.0/topics/http/sessions/#module-django.contrib.sessions) – A session framework.
  - [`django.contrib.messages`](https://docs.djangoproject.com/en/4.0/ref/contrib/messages/#module-django.contrib.messages) – A messaging framework.
  - [`django.contrib.staticfiles`](https://docs.djangoproject.com/en/4.0/ref/contrib/staticfiles/#module-django.contrib.staticfiles) – A framework for managing static files.
- 위 application들은 보통의 경우 편리함을 위해 default로 포함됨.
- 이런 응용 프로그램 중 일부는 최소한 하나의 데이터베이스 테이블을 사용하므로 사용전 DB에 테이블을 생성해야함. 생성을 위해 다음 명령어를 실행.
  `python manage.py migrate`
- migrate 명령은 `INSATLL_APPS` 설정을 살펴보고 mysite/settings.py 파일의 DB 설정과 앱과 함께 제공되는 DB 마이그레이션에 따라 필요한 DB 테이블을 생성. (추후에 다룸.)
- `\dt` (PostgreSQL) 실행시 Django 가 생성한 테이블 표시함.

---
**최소주의자(minimalists)들을 위하여.**

기본으로 제공되는 어플리케이션은 일반적인 상황을 염두에 두었으나, 모두에게 필요한 것은 아님. 필요 없다고 생각 되면, `migrate` 실항 전, `INSTALLED_APPS`에서 제거할 애플리케이션들을 주석처리(comment-out) 하거나 삭제 하면 됨. `migrate` 명령은 `INSTALLED_APPS`에 등록된 어플리케이션에 한해서 실행됨.

---

## [Creating models](https://docs.djangoproject.com/en/4.0/intro/tutorial02/#creating-models)

모델이란, 메타데이터를 가진 DB의 구조를 말함.



**철학**

모델은 데이터에 대한 정보의 단일하고 선언적인 자원. 저장 중인 데이터의 필수 필드와 동작이 포함됨. Django는 DRY 원칙을 따름. 목표는 한 곳에서 데이터 모델을 정의하고, 자동으로 데이터 모델로부터 파생(derive, 문맥상 모델을 생성한다는 의미처럼 보임)시키는 것.

여기에는 migration이 포함됨. 예를 들어 Ruby On Rails 와 달리 migration은 모두 모델 파일에서 파생되며, 기본적으로 Django가 데이터베이스 스키마를 현재의 모델에 도달할 수 있게 해주는 기록.


### `polls/models.py` 에 아래 모델 추가

```python
from django.db import models

class Question(models.Model):
  question_text = models.CharField(max_length=200)
  pub_date = models.DateTimeField('date published')

class Choice(models.Model):
  question = models.ForeignKey(Question, on_delete=models.CASCADE)
  choice_text= models.CharField(max_length=200)
  vote = models.IntegerField(default=0)
```

- 각 모델은 `django.db.models.Model`의 하위 클래스로 표현됨. 모델마다 여러 클래스 변수가 있으며, 각 클래스 변수는 모델에서 데이터베이스 필드를 나타냄.

- DB의 각 필드는 `Field`클래스의 인스턴스로 표현됨. 각 필드가 어떤 자료형을 가질 수 있는지 Django에게 전달함.
  - `CharField`: 문자(character) 필드
  - `DateTimeField`: 날짜와 시간(datetime)

- 각각의 `Field` 인스턴스의 이름(`question_text` 또는 `pub_date`)은 기계가 읽기 좋은(machine-friendly format) 형식의 DB 필드 이름. 이 필드명을 Python 코드에서 사용할 수 있으며, DB에서는 컬럼명으로 사용.

- 몇몇 `Field` 클래스들은 필수 인수가 필요함. `CharField`의 경우 `max_length`가 필수. 스키마에서만 필요한 것이 아니라 값을 검증할 때에도 쓰이는데 곧 보게 될 것.

- `Field`는 다양한 선택적 인수들을 가질 수 있음.

- `ForeignKey`를 사용한 관계 설정. 이 예제에서는 각각의 Choice

- `ForeignKey`를 사용한 관계 설정. 이 예제에서는 각각의 **Choice** 가 하나의 **Question**에 관계된다는 것을 Django에게 알려줌. Django는 다대일(many-to-on), 다대다(many-to-many), 일대일(one-to-one)과 같은 모든 일반 DB관계들을 지원함.

## [Activating models](https://docs.djangoproject.com/en/4.0/intro/tutorial02/#activating-models)

Django가 모델 정보를 가지고 할 수 있는 일.

- 이 앱을 위한 DB schzema 생성 (CREATE TABLE 문)
- **Qustion**과 **Choice** 객체에 접근하기 위한 Python DB 접근 API를 생성

---
철학

Django 앱들은 꼈다뺐다 할 수 있음. 앱을 다수의 프로젝트에서 사용 가능하며 앱을 배포할 수도 있음. 특정 Django 사이트에 앱이 묶여있지 않아도 됨.

---


- 앱을 현재 프로젝트에 포함시키려면 앱의 구성 클래스에 대한 참조를 `INSTALLED_APPS` 설정에 추가해야 함. `PollsConfig` 클래스는 `polls/apps.py` 파일 내에 존재. 즉, `polls.apps.PollsConfig`.

```python
# mysite/settings.py

INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

```

Django는 polls 앱이 포함 된 것을 알게됨.
다음 명령 실행.

`python manage.py makemigrations polls`

실행 결과

```
Migrations for 'polls':
  polls/migrations/0001_initial.py
    - Create model Question
    - Create model Choice
```

- `makemigrations`을 실행시킴으로서, 모델을 변경시킨 사실과 이 변경사항을 migration으로 저장시키고 싶다는 것을 Django에게 알려줌.

- Migration은 Django가 모델(즉, DB 스키마)의 변경사항을 디스크에 저장하는 방법. Django가 mirgarion을 만들 때마다 읽진 않겠지만, Django의 변경점을 수동으로 수정할 수 있게 설계함.

- migration을 실행하고 DB 스키마를 관리해주는 migrate 명령어 실행. sqlmigrate 명령은 migration 이름을 인수로 받아, 실행하는 SQL 문장을 보여줌.

`python manage.py sqlmigrate polls 0001`

실행 결과

```SQL
BEGIN;
--
-- Create model Question
--
CREATE TABLE "polls_question" (
    "id" serial NOT NULL PRIMARY KEY,
    "question_text" varchar(200) NOT NULL,
    "pub_date" timestamp with time zone NOT NULL
);
--
-- Create model Choice
--
CREATE TABLE "polls_choice" (
    "id" serial NOT NULL PRIMARY KEY,
    "choice_text" varchar(200) NOT NULL,
    "votes" integer NOT NULL,
    "question_id" integer NOT NULL
);
ALTER TABLE "polls_choice"
  ADD CONSTRAINT "polls_choice_question_id_c5b4b260_fk_polls_question_id"
    FOREIGN KEY ("question_id")
    REFERENCES "polls_question" ("id")
    DEFERRABLE INITIALLY DEFERRED;
CREATE INDEX "polls_choice_question_id_c5b4b260" ON "polls_choice" ("question_id");

COMMIT;
```

- 사용 DB에 따라 출력결과 달라질 수 있으며, 위 출력결과는 PostgreSQL이 생성.
- 테이블 이름은 app(polls)과 모델의 소문자 이름이 결합해서 자동으로 생성됨. (아 동작을 override 가능.)
- Primary keys (IDs)는 자동으로 추가됨. (override 가능.)
- 컨벤션으로, Django는 "_id"를 외래키 필드명에 붙임. (override 가능.)
- 외래키 관계는 FOREIGN KEY 라는 제약에 의해 명시.

## [Playing with the API](https://docs.djangoproject.com/en/4.0/intro/tutorial02/#playing-with-the-api)
문서 참고


## [Introducing the Django Admin](https://docs.djangoproject.com/en/4.0/intro/tutorial02/#introducing-the-django-admin)

### admin 유저 생성

```python
python manage.py createsuperuser

Username: admin

Email address: admin@example.com

# 비밀번호 만들 때, 8자리 이상 + 너무 쉽지 않게 (예: 12345678 경고 문구 뜸.)
Password: ********
Password (again): ********
Superuser created successfully.
```

### [개발 서버 실행](https://docs.djangoproject.com/en/4.0/intro/tutorial02/#start-the-development-server)

이미지와 함께 확인하기 위해서 이하 내용은 공식 문서 참고.