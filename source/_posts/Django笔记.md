---
title: Django笔记
meta: 'ios,python'
date: 2018-05-10 17:41:48
tags:
categories:
keywords:
---
### 静态文件
STATIC_ROOT = ''
>这个目录定义的是 当部署到生产环境，将使用python manage.py collectstatics 命令 
将静态文件拷贝的路径


STATIC_URL = '/static/' 

>url 在请求静态资源文件时url链接的前缀，例如http:localhost/static/

STATICFILES_DIRS = [
    os.path.join(BASE_DIR, "static"),
]

>静态资源文件存储的路径 这里指的是 项目根目录下的 static 目录

**在最近的几个版本中当 Debug 为false**的情况下，django不处理静态文件资源，会发现静态文件请求404

解决方案：
* 如果临时启动使用runserver 添加 --insecure 这个参数

* 如果使用uwsgi 可以在uwsgi.ini的配置文件中添加静态文件映射：

    [uwsgi 静态文件参数](http://uwsgi-docs-zh.readthedocs.io/zh_CN/latest/StaticFiles.html)

```angular2html
   static-map = /static=./staticfiles
```
    /static 对应上面的STATIC_URL 
 
    ./staticfiles 对应上面的STATICFILES_DIRS 目录。

* 如果使用nginx 可以配置location 接管静态文件资源


### Django 使用登录功能

#### 登录配置

```angularjs
LOGIN_URL = '/accounts/login/'
```
```angularjs
配置路由 
url(r'^accounts/', include('django.contrib.auth.urls')),
```

#### 在请求中验证用户登录
    * 如果使用　def xxx(request) 则可以使用login_required 装饰器
    
```angularjs
@login_required
@permission_required('can_access_assetView', login_url='/asset/error_403/')
def view_assetOperHistory(request):
    """
    # 查看修改历史
    """
    if request.method == 'GET':
        ...
```

   * 如果使用django的视图类，即通用视图，来实现VIEW功能　验证用户是否登录则可以使用如下方法：
   
   方法一： 自定义一个　LogingRequireMixin
   ```angularjs 
class LoginRequiredMixin(object):
    """
    登陆限定，并指定登陆url
    """
    @classmethod
    def as_view(cls, **initkwargs):
        view = super(LoginRequiredMixin, cls).as_view(**initkwargs)
        return login_required(view, login_url='/')
        
    class ViewAsset(LoginRequiredMixin, ListView):
    """
    具体的业务实现类
    """
    model = Group
    paginate_by = 10
    template_name = "asset_list.html"
```

### 在通用视图下使用装饰器
django的bug，不能直接对类进行装饰，必须使用 method_decorator,把装饰器当作参数传进去。

from django.utils.decorators import method_decorator
@method_decorator(wrapper, name="post")

```angularjs
from functools import wraps

def count_time_cost(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = datetime.datetime.now()
        r = func(*args, **kwargs)
        end = datetime.datetime.now()
        LOG.info("Django view function run finished! Task_message : {} Cost_time : {}".format(r.data.get("msg"),
                                                                                              (end - start)))
        return r
    return wrapper
```

```angularjs
class VolumeQueryNew(APIView):
    @method_decorator(count_time_cost)
    def get(self, request, *args, **kwargs):
        zone_id = request.query_params.get("zoneid")
        volume_uuid = request.query_params.get("volume_uuid") or request.query_params.get("volume_id")
        rc = get_rc_by_udc(request)
        add_mark = "1"
        volume_filtered_info = []
        try:
            msg = 'Get all volume info successfully!'
            LOG.info("volumes info is{}".format(str(volume_filtered_info)))
            pagination_class = APIViewPagePagination
            return common_page_response(volume_filtered_info, request, msg, pagination_class)
        except Exception as e:
            LOG.exception("failed to get volumes!error exception msg{}".format(e))
            return common_error_response("failed to get volumes!error msg",
                                         FAILED_TO_GET_ALL_VOLUMES)
```
