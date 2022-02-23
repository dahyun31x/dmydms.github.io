# URL dispatcher

> 더 이상 추가나 수정 될 부분이 없는 섹션은 ☑ 표시.

품질 높은 웹 애플리케이션에서는 깔끔하고 우아한 URL scheme이 중요.

## ☑ 개요 

- app URL들을 디자인하기 위해서 URLconf(URL configuration)라 불리는 Python 모듈을 생성해야 함. 이 모듈은 순수 Python 코드이고, URL path expression을 Python function (view)에 연결.
- >This mapping can be as short or as long as needed. It can reference other mappings. And, because it’s pure Python code, it can be constructed dynamically.
- Django는 활성 언어에 따라 URL 번역을 제공. [internationalization documentation](https://docs.djangoproject.com/en/4.0/topics/i18n/translation/#url-internationalization) 참고.

## Django로 Request 처리하는 방법

사용자가 Django 기반 사이트에서 페이지를 요청할 때, 실행할 Python 코드를 결정하기 위해 시스템이 따르는 알고리즘.

1. Django는 이용할 root URLconf를 결정. 보통, [ROOT_URLCONF](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-ROOT_URLCONF) 셋팅 값이지만, 들어오는 **HttpRequest** 객체가 [urlconf](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpRequest.urlconf) 속성을 갖고 있다면(미들웨어에 의해서 설정된), [ROOT_URLCONF](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-ROOT_URLCONF) 셋팅이 있는 곳에서 사용 될 것임.

2. Django는 Python 모듈을 로드하고, urlpatterns를 찾음. 이것은 django.urls.path() 와/또는 django.urls.re_path() 인스턴스의 sequence(다현: ex. 배열) 여야함.

3. Django는 각각의 URL 패턴을 순서대로 실행하고(run through) 요청된 URL과 일치하는 첫번째 패턴에서 멈추고, path_info와 일치해봄.

4. 일단 URL patterns 중의 하나의 패턴과 일치하면, Django는 주어진 Python 함수인 (또는 [class 기반의](https://docs.djangoproject.com/en/4.0/topics/class-based-views/)) view를 import하고 호출. view는 아래 매개인자(argument)를 전달함.

    - [HttpRequest](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpRequest) 인스턴스
    - 일치하는 패턴이 이름이 없는 그룹이라면, 정규식의 일치 항목이 positional argument로 제공됨.
    - >The keyword arguments are made up of any named parts matched by the path expression that are provided, overridden by any arguments specified in the optional kwargs argument to django.urls.path() or django.urls.re_path().

5. 일치하는 URL 패턴이 없다면, 또는 이 과정 중의 어떤 지점에서 예외가 발생한다면, Django는 error-handlindg view를 부름(invoke). [Error handling](https://docs.djangoproject.com/en/4.0/topics/http/urls/#error-handling) 참고.

## ☑ 예제

```python
from django.urls import path

from . import views

urlpatterns = [
    path('articles/2003/', views.special_case_2003),
    path('articles/<int:year>/', views.year_archive),
    path('articles/<int:year>/<int:month>/', views.month_archive),
    path('articles/<int:year>/<int:month>/<slug:slug>', views.article_detail),
]
```

Notes:

- URL에서 값을 캡쳐하려면 angle bracket을 사용.
- 캡쳐된 값은 선택적으로 converter type을 포함.  
  예: 정수 파라미터를 캡쳐하기 위해서 \<int:name> 사용하기.  
  만약 converter가 포함되지 않았다면, / 문자를 제외한 문자열이 일치하게 됨.
- 모든 URL이 갖고 있기 때문에, leading slash를 넣을 필요가 없음.  
ex. `/articles` -> `articles`

예시 requests:

- `/articles/2005/03/` 으로의 요청은 리스트에서 세번째 엔트리와 일치. Django는 `views.month_archive(request, year=2005, month=3)` 함수를 호출.
- `/articles/2003/` 는 리스트에서 첫번째 패턴과 일치. 두번째가 아님. 왜냐하면 패턴들은 순서대로 테스트되고, 첫번째 패턴이 통과하는 첫번째 테스트이기 때문. 특별한 경우를 위해서 순서를 사용하기.  `views.special_case_2003(request)`
- 각각의 패턴들은 end slash 필요로 하기 때문에 `/articles/2003` 는 어떠한 패턴들과도 일치하지 않음.
- `/articles/2003/03/building-a-django-site/` 는 마지막 패턴과 일치. Django는 `views.article_detail(request, year=2003, month=3, slug="building-a-django-site")`를 호출.

## ☑ Path converters

default로 사용 가능한 path converters:

- **str**: path 분리자 '/'를 제외한 비지 않은 문자열과 매치. converter가 표현식에 포함되지 않는다면 default.
- **int**: 0을 포함한 양수와 매치. `int` 리턴. 
- **slug**: ASCII 또는 숫자로 이루어진 slug string, hypen(-), underscore 문자(_)과 매치.  
ex. `building-your-1st-django-site`
- **uuid**: formatted UUID와 매치. 같은 페이지로 맵핑하는 여러개의 URL들을 막기 위해, 대쉬는 필수로 포함되고 문자들은 소문자여야함. [UUID 인스턴스](https://docs.python.org/3/library/uuid.html#uuid.UUID)
- **path**: 패스 분리자 '/'를 포함하는 비지 않는 문자열과 매치. 이를 통해 str에서와 같이 URL 경로의 세그먼트가 아닌 완전한 URL 경로와 일치시킬 수 있음.

## [custom path converters 등록하기](https://docs.djangoproject.com/en/4.0/topics/http/urls/#registering-custom-path-converters)

복잡한  매칭 요구사항을 위해 자신만의 path converter를 정의 할 수 있음.  

컨버터는 다음을 포함하는 클래스:

- **regex**
- **`to_python(self, value)`**
- **`to_url(self, value)`**

예:

```python
class FourDigitYearConverter:
    regex = '[0-9]{4}'

    def to_python(self, value):
        return int(value)
    
    def to_url(self, value):
        return '%04d' % value
```

URLconf에서 `register_converter()` 를 이용해 custom converter class 등록:

```python
from django.urls import path, register_converter

from . import converters, views 

register_converter(converters.FourDigitYearConverter, 'yyyy')

urlpatterns = [
    path('articles/2003/', views.special_case_2003),
    path('articles/<yyyy:year>/', views.year_archive),
    ...
]
```

## [정규 표현식 이용하기](https://docs.djangoproject.com/en/4.0/topics/http/urls/#using-regular-expressions)

path와 converter 문법이 URL pattern을 정의하기에 충분하지 않다면, 정규 표현식 이용. `path()` 대신 `re_path()` 이용.

Python 정규 표현식에서는 named regular expression groups에 대한 문법이 **(?p\<name>pattern)**

## [URLconf가 검색하는 대상](https://docs.djangoproject.com/en/4.0/topics/http/urls/#what-the-urlconf-searches-against)

## [view 매개인자(arguments)를 위한 구체적인 defaults](https://docs.djangoproject.com/en/4.0/topics/http/urls/#specifying-defaults-for-view-arguments)

## [성능](https://docs.djangoproject.com/en/4.0/topics/http/urls/#performance)

## [**urlpatterns** 변수의 문법](https://docs.djangoproject.com/en/4.0/topics/http/urls/#syntax-of-the-urlpatterns-variable)

## [Error handling](https://docs.djangoproject.com/en/4.0/topics/http/urls/#error-handling)

## [다른 URLconfs 포함하기](https://docs.djangoproject.com/en/4.0/topics/http/urls/#including-other-urlconfs)

## [view 함수에 추가 옵션 전송하기](https://docs.djangoproject.com/en/4.0/topics/http/urls/#passing-extra-options-to-view-functions)

## [Reverse resolutoin of URLs](https://docs.djangoproject.com/en/4.0/topics/http/urls/#reverse-resolution-of-urls)

## [URL 패턴 이름짓기](https://docs.djangoproject.com/en/4.0/topics/http/urls/#naming-url-patterns)

## [URL namespaces](https://docs.djangoproject.com/en/4.0/topics/http/urls/#url-namespaces)

## [Reversing namespaced URLs](https://docs.djangoproject.com/en/4.0/topics/http/urls/#reversing-namespaced-urls)
