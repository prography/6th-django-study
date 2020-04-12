# 2. Requests and Responses
## 2.1 Request
DRF가 제공하는 ```Request``` 객체는 Django의 ```HttpRequest```를 확장한 것이다.
- ```HttpRequest.POST```: POST request의 정보를 포함하고 있는 객체 (파일 업로드 정보는 포함하지 않음)
- ```Request.data```: POST, PUT, PATCH request의 정보를 모두 포함한 객체 (파일 입력도 포함)

## 2.2 Response
```Response``` 객체 또한 DRF에 포함되어 있는데, 이는 Django의 ```SimpleTempleteResponse```의 서브클래스이다.
- ```Response(data)```: 매개변수에 serializer의 데이터를 전달하면 클라이언트가 요청한 content type으로 해당 데이터를 변환하여 return

## 2.3 Status
```status``` 모듈에는 HTTP status code가 들어있는데, 해당 코드에 대한 간단한 설명? 같은 것도 변수명에 포함되어 있어서 이를 사용하면 가독성을 높일 수 있다.

## 2.4 Wrapper
- ```@api_view```: 함수 기반 view에 적용하는 데코레이터
- ```APIView```: 클래스 기반 view에서 상속하는 클래스

## 2.5 Views
위의 요소들을 포함하여 ```snippets/views.py``` 파일을 작성하자.
```python
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.response import Response
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer


@api_view(['GET', 'POST'])
def snippet_list(request):
    # GET method면 모든 Snippet 리스트 출력
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)

    # POST method면 새로운 Snippet 생성
    elif request.method == 'POST':
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)


@api_view(['GET', 'PUT', 'DELETE'])
def snippet_detail(request, pk):
    # 해당 pk의 Snippet이 있는지 확인, 없으면 404 NOT FOUND
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    # GET method면 해당 Snippet 출력
    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return Response(serializer.data)

    # PUT method면 해당 Snippet 수정
    elif request.method == 'PUT':
        serializer = SnippetSerializer(snippet, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    # DELETE method면 해당 Snippet 삭제
    elif request.method == 'DELETE':
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

## 2.6 Format
view 함수의 매개변수에 format 속성을 지정하면 특정 포맷으로 출력 형식을 지정할 수 있다.
```python
def snippet_list(request, format=None):
    ...

def snippet_detail(request, pk, format=None):
    ...
```
이후 snippets/urls.py 파일을 수정하자.
```python
from django.urls import path
from rest_framework.urlpatterns import format_suffix_patterns
from snippets import views

urlpatterns = [
    path('snippets/', views.snippet_list),
    path('snippets/<int:pk>/', views.snippet_detail),
]

urlpatterns = format_suffix_patterns(urlpatterns)
```

## 2.7 How it works
상태를 확인하기 위해 다시 서버를 실행시키고 접속하자.
```
python manage.py runserver
http http://127.0.0.1:8000/snippets/
```
지금은 Snippet 데이터가 없으므로 리스트에 아무것도 없다.
```
HTTP/1.1 200 OK
Allow: GET, OPTIONS, POST
Content-Length: 2
Content-Type: application/json
Date: Sun, 12 Apr 2020 13:57:10 GMT
Server: WSGIServer/0.2 CPython/3.8.2        
Vary: Accept, Cookie
X-Content-Type-Options: nosniff
X-Frame-Options: DENY

[]
```
새로운 Snippet을 생성하면 다음과 같은 응답이 온다.
```
http POST 127.0.0.1:8000/snippets/ code="print('hello')"
```
```
HTTP/1.1 201 Created
Allow: GET, OPTIONS, POST
Content-Length: 98
Content-Type: application/json
Date: Sun, 12 Apr 2020 14:06:15 GMT
Server: WSGIServer/0.2 CPython/3.8.2        
Vary: Accept, Cookie
X-Content-Type-Options: nosniff
X-Frame-Options: DENY

{
    "code": "print('hello')",
    "id": 1,
    "language": "python",
    "linenos": false,
    "style": "friendly",
    "title": ""
}
```
이제 Snippet 리스트 view에 생성한 Snippet이 뜨는 것을 확인할 수 있다.
```
http http://127.0.0.1:8000/snippets/
```
```
HTTP/1.1 200 OK
Allow: GET, OPTIONS, POST
Content-Length: 100
Content-Type: application/json
Date: Sun, 12 Apr 2020 14:09:06 GMT
Server: WSGIServer/0.2 CPython/3.8.2        
Vary: Accept, Cookie
X-Content-Type-Options: nosniff
X-Frame-Options: DENY

[
    {
        "code": "print('hello')",
        "id": 1,
        "language": "python",
        "linenos": false,
        "style": "friendly",
        "title": ""
    }
]
```
디테일 view에서도 확인 가능하다.
```
http http://127.0.0.1:8000/snippets/1/
```
```
HTTP/1.1 200 OK
Allow: PUT, GET, OPTIONS, DELETE
Content-Length: 98
Content-Type: application/json
Date: Sun, 12 Apr 2020 14:10:56 GMT
Server: WSGIServer/0.2 CPython/3.8.2        
Vary: Accept, Cookie
X-Content-Type-Options: nosniff
X-Frame-Options: DENY

{
    "code": "print('hello')",
    "id": 1,
    "language": "python",
    "linenos": false,
    "style": "friendly",
    "title": ""
}
```