# The second week of learning Django

## 云笔记项目

### 功能拆解

#### 用户模块

1. 注册-成为平台用户

2. 登陆-校验用户身份

3. 退出登陆-退出登陆状态

#### 笔记模块

1. 查看笔记列表-查

2. 创建新笔记-增

3. 修改笔记-改

4. 笔记删除-删

### 步骤

#### 配置环境

1. 配置数据库时出现的问题
   1. `mysql -u root -p `....这都能忘
   2. `create database NoteOnline default charset utf8;`

2. runserver时出现问题:

   1. `Error: [WinError 10013] 以一种访问权限不允许的方式做了一个访问套接字的尝试。`

   2. ```powershell
      C:\>netstat -ano|findstr 8000  
      TCP    0.0.0.0:8000     LISTENING       8124 
      C:\>tasklist |findstr 8124  
      KGService.exe                 8124 
      C:\Users\liyunzhi>taskkill /pid 8124 /F  
      成功: 已终止 PID 为 8124 的进程。  
      ```

   3. 存在占用,若不重要,终止即可

#### 注册问题

1. 密码的处理

#### 检查登录状态

1. 使用装饰器,检查

### 完成,详见代码示例

## 进一步

### 7.1缓存

1. 配置settings.py

   ```python
   CACHES = {
       'default': {
           'BACKEND': 'django.core.cache.backends.db.DatabaseCache',
           'LOCATION': 'my_cache_table',
           'TIMEOUT': 300,  # 缓存保存时间单位秒默认值为300
           'OPTIONS': {
               'MAX.ENTRIES': 300,  # 缓存最大数据条数
               'CULL_FREQUENCY1': 2,  # 缓存条数达到最大值时删除1/x的缓存数据
           }
       }
   }
   ```

2. 缓存策略

   1. 使用装饰器

   ```
   from django.views.decorators.cache import chache_page
   
   @cache_page(30) 	#单位s	
   def ......
   ```

3. 可以在路由中使用

   ```
   path('...',cache_page(30)views.view)
   ```

4. cache的配置是一个字典,所以我们可创建与default同级的配置,然后调用

   1. `from django.views.decorators.cache import chache_page`相当于直接引入default

5. 使用`cache.set(key,value,timeout)`

   1. key,缓存的键,str
   2. value:
   3. timeout:存储时间

6. 获取:cache.get(key)

7. cache.add(key,value) 存储缓存:key不存在时生效(返回:true,false)

8. cache.get/set_many,cache.delete

9. 浏览器缓存策略

   1. 先请求缓存,后请求服务器(强缓存)
   2. 响应头:Expires:需要缓存(绝对时间);cache-control控制(相对时间)
   3. 协商缓存:前缓存是否可用->决定是否要请求服务器

### 7.2中间件

1. 继承django.utils.deprecation.MiddlewareMixin
2. 五个方法
   1. process_request(self,request)
   2. process_view(self,request,callback,callback_args,callback_kwargs)到达view前
   3. process_response(self,request,response)
   4. process_exception(self,request,exception)抛出异常,返回HttpResponse对象(发送报错邮件)
   5. process_template_response(self,request,response)
3. 在settings中注册自定义中间件
   1. MIDDLEWARE=[]
4. request.META['REMOTE_ADDR']得到远程客户端的IP
5. csrf攻击:跨站伪造请求攻击

### 7.3分页 

1. Paginator对象
   1. paginator=Paginator(objuct_list,per_page)	# 对象列表,每页数据个数;返回paginator对象
   2. 属性:对象总数,页面数,从1开始range对象
   3. .page(page_number) 返回对应页

### 7.4生成csv文件

在网页中用的不多,这里就略过

### 8.1内建用户系统

1. 可以用django自带的用户表

2. from django.contrib.auth.models import User

3. ```
   from ...
   user=User.objects.create_user(.....)
   ```

4. 删除用户(伪)

   ```
   user=User.objects.get(username='用户名')
   user.is_active=False
   user.save()
   ```

5. 校验密码

   ```
   user=authticate(username=username,password=password)# 成功,返回user对象,否则返回None
   ```

6. 修改密码

   ```
   # user= ..get(..)
   user.set_password(...)
   user.save()
   ```

7. 登录状态保持

   ```
   from django.contrib.auth import login
   def login_view(request):
   	user=authenticate(...)
   	login(request,user)
   ```

8. 登录状态校验

   ```
   from ... import login_required
   @login_required
   def ....
   	#用户状态才可以访问
   	#用户获取方式:
   	login_user=request.user
   ```

9. 登录状态取消

   ```
   from ... logout
   def logout_view(request);
   	logout(request)
   ```

10. 添加字段(困难,略过)[自己做的时候,自建]

### 8.2文件上传

1. 表单用`<input type="file" name="xxx">`

2. 视图中,用request.FILES取文件框内容

   1. `file=request.FILES['xxx']`
   2. FILES的key对应html中name
   3. file.name文件名
   4. file.file文件的字节流数据

3. 配置文件的访问路径和存储路径

   1. MEDIA_URL='/media/'		

   2. MEDIA_ROOT = os.path.join(BASE_DIR,'media')

   3. MEDIA_URL和MEDIA_ROOT需要手动绑定

      ```
      # 主路由中
      ... import settings
      ... import static
      urlpatterns += static(settings.MEDIA_URL,document_root=settings.MEDIA_ROOT)
      ```

4. 文件写入

   1. open;借助ORM(略)

### 8.3发送邮件

```
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.qq.com' # 腾讯QQ邮箱 SMTP 服务器地址
EMAIL_PORT = 25  # SMTP服务的端口号
EMAIL_HOST_USER = '2650951624@qq.com'  # 发送由B件的QQ由B箱
EMAIL_HOST_PASSWORD = 'xodnydwcyraiecabb'  # 在QQ邮箱-＞设置-＞帐户-＞ "P0P3/IMAP……服务”里得到的在第三方登录QQ邮箱授权码
EMAIL_USE_TLS = False  # 与5乂丁「服务器通信时，是否启动TLS链接 （安全链接）默认False
```



```
>>> from django.core import mail
mail.send_mail(
	subject,#title
	message,#信息
	from_email,#发送者
	recipient_list=['xxx@qq.com'],#接受者
	)
```

1. 使用traceback:`print(traceback.format_exc(exception))`
2. 中间件发送邮件

### 8.4项目部署

这里看看就好了,不多说.

