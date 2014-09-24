# Chapter 14 session_user



##存、取cookie

取出cookie:
> request.COOKIES["favorite_color"]

```python
# mysite/cookie_view.py
def show_color(request):
    if "favorite_color" in request.COOKIES:
        return HttpResponse("Your favourite color is %s" % \
                            request.COOKIES["favorite_color"])
    else:
        return HttpResponse("You don't have a favorite color.")
```

存cookie:
>  response.set_cookie("favourtie_color",cookie_value)

```python
# mysite/cookie_view.py
def set_color(request):
    if "favourite_color" in request.GET:
        response = HttpResonse("Your favourite color is now %s" % \
                              request.GET["favourite_color"]
                              )
        response.set_cookie("favourtie_color",request.GET["favourite_color"])
        return response
    else:
        return HttpResponse("You don't have a favorite color.")
```

![cookie_view.png](https://raw.githubusercontent.com/urmyfaith/NotesOfDjangoBook/master/notes/images/cookie_view.png)

----


## 打开Sessions功能

> 1. 编辑 MIDDLEWARE_CLASSES 配置，确保 MIDDLEWARE_CLASSES 中包含 'django.contrib.sessions.middleware.SessionMiddleware'。

> 2. 确认 INSTALLED_APPS 中有 'django.contrib.sessions' 

> 3. (如果你是刚打开这个应用，别忘了运行 manage.py syncdb )

```python
# mysite/settings.py

INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'mysite',
    'books',
    'blog'
)

MIDDLEWARE_CLASSES = (
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.auth.middleware.SessionAuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
)
```

---
## 在View(视图)中使用Session
> SessionMiddleware 激活后，每个传给视图(view)函数的第一个参数``HttpRequest`` 对象都有一个 session 属性，这是一个字典型的对象。

```python

#设置session值
request.seesion["fav_color"]="blue"

#读取session值
fav_color=request.session["fav_color"]

#删除某个session值
del request.session["fav_color"]

#判断是否存在某个session值
if "fav_color" in request.session:
    ...
```
注意:

1> 不要使用下划线开头的变量作为key

2> 不要将新对象替换 request.session对象.

### 简单（但不很安全）的、防止多次评论方法
```python
#mysite/urls.py
urlpatterns += patterns('mysite.session_view',
    url(r'^chapter14/post_comment/$','post_comment'),
)
#mysite/session_view.py
def post_comment(request):
    errors=[]
    if request.method == 'POST':
        if not request.POST.get('subject',''):
            errors.append('Enter a subject.')
        if not request.POST.get('message',''):
            errors.append('Enter a meeage.')
        if request.POST.get('email') and '@' not in request.POST['email']:
            errors.append('Enter a valid e-mail address.')
        if request.session.get('has_commented', False):
           return HttpResponse("You've already commented.")
        if not errors:
            #do something here, eg: save it to database
            request.session['has_commented'] = True
            return HttpResponse('Thanks for your comment!')
    return render_to_response('post_comment.html',\
                              {'errors':errors,}, \
                              context_instance=RequestContext(request))

#mysite/templates/post_comment.html
<html>
<head>
    <title>post comment</title>
</head>
<body>
    <h1>post comment</h1>
    {% if errors %}
        <ul>
            {% for error in errors %}
            <li>{{ error }}</li>
            {% endfor %}
        </ul>
    {% endif %}
	<form action="" method="post">
		{% csrf_token %}
		<p> Subject:<input type="text" name= "subject" value="{{ subject }}"></p>
		<p>Your e-mail(optional):<input type="text" name="email" value="{{ email }}">
		<p>message:<textarea name ="message" rows="10" cols="50">{{ message }}</textarea><p>
		<input type="submit" value="Submit">
 	</form>
</body>
</html>

```
上面:

> 1.URLconf

> 2.视图中,判断是否为post方法,

a) post:判断提交参数合法性,判断session值,分别返回.

b) get:显示form

> 3.模版,显示一个form和错误处理.

![befor_post_comment.png](https://raw.githubusercontent.com/urmyfaith/NotesOfDjangoBook/master/notes/images/befor_post_comment.png)

![after_post.png](https://raw.githubusercontent.com/urmyfaith/NotesOfDjangoBook/master/notes/images/after_post.png)

![post_again.png](https://raw.githubusercontent.com/urmyfaith/NotesOfDjangoBook/master/notes/images/post_again.png)

![table_django_session.png](https://raw.githubusercontent.com/urmyfaith/NotesOfDjangoBook/master/notes/images/table_django_session.png)


---
### 设置测试是否允许Cookies

在访问网页时(get),尝试设置cookie,

在登录的时候(post),判断设置cookie是否正常.然后进行分支处理.

```python
def login(request):
    errors=[]
    if request.method == 'POST':
        if not request.POST.get('username',''):
            errors.append('Enter a username.')
        if not request.POST.get('password',''):
            errors.append('Enter a password.')
        if not errors:
            if request.session.test_cookie_worked():
                request.session.delete_test_cookie()
                return HttpResponse("Logged in.")
            else:
                return HttpResponse('Please enable coookie.')
    request.session.set_test_cookie()
    return render_to_response('login.html',\
                              {'errors':errors,}, \
                              context_instance=RequestContext(request))
```
1)判断是否允许设置cooke:
> request.session.test_cookie_worked()

2)删除测试cookie:
> request.session.delete_test_cookie()

3)请求的时候设置cookie(正常访问网页时判断)
> request.session.set_test_cookie()

![set_test_cookie.png](https://raw.githubusercontent.com/urmyfaith/NotesOfDjangoBook/master/notes/images/set_test_cookie.png)

---
## 在视图(View)外使用Session

1)需要导入包: from django.contrib.sessions.models import Session

2) 常见的函数/属性:exprire_data,session_data,get_decoded()

```python
D:\Documents\GitHub\NotesOfDjangoBook\mysite>python manage.py shell
>>> from django.contrib.sessions.models import Session
>>> s= Session.objects.get(pk='vyrd47zvgrj2ot7z7lguekw1d42n0isl')
>>> s.expire_date
datetime.datetime(2014, 10, 5, 5, 49, 6, 915000, tzinfo=<UTC>)
>>> s.session_data
u'ODNiOGJmNmVkN2NjMjA5ODhhN2M1NmViNjU1N2E2ZDlhNGQ4Yzc5Yjp7fQ=='
>>> s.session_key
u'vyrd47zvgrj2ot7z7lguekw1d42n0isl'
>>> s.get_decoded()
{u'has_commented': True, u'testcookie': u'worked'}
>>> cookies=s.get_decoded()
>>> cookies['has_commented']
True
>>>


```
---

## Users and Authentication

Django 认证/授权 系统会包含以下的部分：

>用户 : 在网站注册的人

>权限 : 用于标识用户是否可以执行某种操作的二进制(yes/no)标志

>组 :一种可以将标记和权限应用于多个用户的常用方法

>Messages : 向用户显示队列式的系统消息的常用方法

## 打开认证支持

1> 需要确认用户使用cookie，这样sesson 框架才能正常使用。1

2>将 'django.contrib.auth' 放在你的 INSTALLED_APPS 设置中，然后运行 manage.py syncdb以创建对应的数据库表。

3>确认 SessionMiddleware 后面的 MIDDLEWARE_CLASSES 设置中包含 'django.contrib.auth.middleware.AuthenticationMiddleware' SessionMiddleware

### 判断用户是否登录
```python
if request.user.is_authenticated():
    # Do something for authenticated users.
else:
    # Do something for anonymous users.
```