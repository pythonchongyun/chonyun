## Flask项目完整流程

## 一、安装环境

#### 进入桌面

> cd    Desktop/

#### 创建目录

> mkdir    tigereye

#### 进入目录

> cd    tigereye/

#### 查询python3的安装位置

> which    python3

#### 安装virtualenv

> pip    install    virtualenv

#### 创建env环境

> virtualenv    -p    /usr/bin/python3    env
>
> 如果默认python为python3版本，可用直接使用virtualenv    env创建环境；输入python查询当前使用python的版本

### 使用env环境

> source     env/bin/activate



## 二、创建项目

#### 在工程目录下创建app.py文件

```python
from flask import Flask

app = Flask(__name__)

app.run()
```

#### 在工程目录下创建requirements.txt文件，用处后台安装文件使用

```
pymysql
Flask
```

##### 注意：

​	如果使用requirements.txt文件没有提示install安装的话，需要在终端中手动安装

```
pip install -r requirements.txt
```

#### 添加路由进行测试

```python
@app.route('/index/')
def index():
    return "hello world"
```

#### 终端输入测试

```
python app.py
```



## 三、项目启动优化

#### 安装flask_script

#### 修改配置，settings——>project：tigereye——>Project intarpreter，修改为创建的env环境

#### 在工程目录下创建manage.py文件

```python
from flask_script import Manager
```

#### 在工程目录下创建一个python包tigereye，作为项目目录

#### 将app.py文件放入项目目录

####  修改app.py文件

```python
from flask import Flask

def create_app():
    app = Flask(__name__)
    app.debug = True
    return app
```

#### 在manage.py文件中引用app

```python
from flask_script import Manager,Server
from tigereye.app import create_app

app = create_app()
manager = Manager(app)

manager.add_command("runserver",Server("127.0.0.1",port=5000))

if __name__ == '__main__':
    manager.run()
```

#### 终端测试

```
python manage.py runserver
```



## 四、项目路由优化

#### 安装flask_classy，使用该插件可以自动生成路由

#### 在项目目录下创建python包api，用来存放视图路由

####  在api下创建misc.py文件

```python
from flask_classy import FlaskView,route

class MiscView(FlaskView):
    #定义访问ip后，可以直接访问到index
    route_base = '/'
    
    #定义index方法后，可以在浏览器输入文件名后，直接访问到某路由
    def index(self):
        return self.check()
    
    def check(self):
    	return "I'm OK"
```

#### 在app.py文件中注册视图

```python
from tigereye.api.misc import MiscView

def create_app():
    MiscView.register(app)
```

#### 测试：

##### 黑屏启动

```
python manage.py runserver
```

##### 浏览器输入( ip : 端口 / 方法文件名 / 方法名 )

```
127.0.0.1:5000/misc/check 
或
127.0.0.1:5000/misc 
或
127.0.0.1:5000
```



## 五、定义模型

#### 在项目目录下创建python包models，用于存放模型文件

#### 安装flask_SQLAlchemy

#### 在models目录下的__init\_\_.py修改初始化代码

```python
from flask_sqlalchemy import SQLAlchemy

#创建数据库对象
db = SQLAlchemy()
```

#### 在models目录下创建cinema.py文件，存放cinema类

```python
from tigereye.models import db

class Cinema(db.Model):
    cid = db.Column(db.Integer,primary_key=True,nullable=False)
    name = db.Column(db.String(64),unique=True,nullable=False)
    address = db.Column(db.String(128),nullable=False)
    halls = db.Column(db.Integer, default=0, nullable=False)
    hendle_fee = db.Column(db.Integer, default=0, nullable=False)
    buy_limit = db.Column(db.Integer, default=0, nullable=False)
    status = db.Column(db.Integer, default=0, nullable=False, index=True)
```

#### 创建hall.py文件，存放hall类

```python
from tigereye.models import db

class Hall(db.Model):
    hid = db.Column(db.Integer,primary_key=True,nullable=False)
    cid = db.Column(db.Integer)
    name = db.Column(db.String(64),unique=True,nullable=False)
    screen_type = db.Column(db.String(32))
    audio_type = db.Column(db.String(32))
    seats_num = db.Column(db.Integer,default=0,nullable=False)
    status = db.Column(db.Integer,default=0,nullable=False,index=True)

```

#### 创建seat.py文件，存放seat类

```python
from tigereye.models import db

class Seat(db.Model):
    sid = db.Column(db.Integer,primary_key=True)
    hid = db.Column(db.Integer)
    cid = db.Column(db.Integer)

    x = db.Column(db.Integer,default=0,nullable=False)
    y = db.Column(db.Integer, default=0, nullable=False)
    row = db.Column(db.String(8))
    column = db.Column(db.String(8))

    area = db.Column(db.String(16))
    seat_type = db.Column(db.String(16))
    love_seats = db.Column(db.String(32))
    status = db.Column(db.Integer, default=0, nullable=False, index=True)
```

#### 创建movie.py文件，存放movie类

```python
from tigereye.models import db

class Movie(db.Model):
    mid = db.Column(db.Integer,primary_key=True)
    name = db.Column(db.String(64),nullable=False)
    language = db.Column(db.String(32))
    subtitle = db.Column(db.String(32))
    show_date = db.Column(db.Date)
    mode = db.Column(db.String(16))
    vision = db.Column(db.String(16))
    screen_size = db.Column(db.String(16))
    introduction = db.Column(db.Text)
    status = db.Column(db.Integer,default=0,nullable=False,index=True)
```



## 六、定义配置文件

#### 在项目目录下创建python包configs，用于存放配置文件

#### 在configs下创建默认配置文件default.py文件

```python
class DefaultConfig(object):
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = "mysql+pymysql://root:123123@localhost/tigereye"
    SQLALCHEMY_TRACK_MODIFICATIONS = False
```

#### 在app.py文件中导入配置，初始化数据库对象

```python
from tigereye.models import db 
def create_app():
	#app.debug = False	
    																app.config.from_object("tigereye.configs.default.DefaultConfig")
     db.init_app(app)
	db.init_app(app)
```

#### 在manager.py文件中定义方法，根据类创建数据库表(记得导入要导入数据的库)

```python
from tigereye.models import db 

#创建表
@manager.command
def createdb():
    db.create_all()
   
#删除表
@manager.command
def dropdb():
    db.drop_all()
   
#更新表
@manager.command
def init():
    dropdb()
    createdb()
```

####  在数据库中创建tigereye库

#### 黑屏中输入

```
python manage.py createdb
```

**结果：**

​	在数据库中生成数据表



## 七、数据库操作

### 插入数据

### 安装ipython，在shell中可以使用tab键提示命令

```
pip install ipython
```



#### 方法一：

终端输入

```
python manage.py shell
from tigereye.models.cinema import Cinema
c = Cinema()
c.name = 'aaaa'
c.adress = 'bbbb'
c.halls = 10
c.status = 1
from tigereye.models import db
db.session.add(c)
db.session.commit()
```

##### 注意：

​	如果出错，需要执行db.session.rollback()

​	回滚后，修改数据，进行add和commit



#### 方法二：

在models目录下的\_\_init__.py文件中定义类

```python
class Model(object):
    def put(self):
        db.session.add(self)

    def commit(self):
        db.session.commit()

    def rollback(self):
        db.session.rollback()

    def save(self):
        try:
            self.put()
            self.commit()
        except Exception:
            raise

    def delete(self):
        db.session.delete(self)
```

让models目录下的类，继承\_\_init__.py中的Model类

```python
from tigereye.models import db,Model

class Cinema(db.Model,Model):
    
class Hall(db.Model,Model):
    
class Seat(db.Model,Model):
    
class Movie(db.Model,Model):
```

黑屏中输入

```
python manage.py shell
from tigereye.models.cinema import Cinema
c = Cinema()
c.name = 'aaaa'
c.adress = 'bbbb'
c.halls = 10
c.status = 1
c.save()
```

#### 扩展

在manage.py文件中配置shell

```python
from flask_script import Manager, Server,Shell
from tigereye.models.cinema import Cinema
from tigereye.models.hall import Hall
from tigereye.models.seat import Seat

def _make_context():
    locals().update(globals())
    return dict(**locals())
manager.add_command("shell",Shell(make_context=_make_context))



#将全局变量的库导入本地库中，再以字典的形式传入shell中使用，即在shell中不需要再手动导入包
```



## 八、视图路由

#### 方法一：

##### 在api目录下创建ciname.py文件

```python
from flask import  jsonify
from flask_classy import FlaskView
from tigereye.models.cinema import Cinema


class CinemaView(FlaskView):
    def all(self):
        cinemas = Cinema.query.all()
        cinema_list = []
        for cinema in cinemas:
            cinema_list.append({
                "cid":cinema.cid,
                "name":cinema.name,
                "address":cinema.address,
                "halls":cinema.halls,
                "handle_fee":cinema.hendle_fee,
                "buy_limit":cinema.buy_limit,
                "status":cinema.status,
            })
        return jsonify(cinema_list)
```



#### 方法二：

##### 在models目录下的\_\_init__.py文件中的Model类中添加方法

```python
#定义一个__json__方法，将数据直接转化为json对象
class Model():
    def __json__(self):
        _d = {}
        for k,v in vars(self).items():
            if k.startswith("_"):
                # “_”开头的键对应的是一个对象,而不是值,无法序列化
                continue
            _d[k] = v
        return _d
```

##### 对api目录下cinema.py中代码进行优化

```python
from flask import jsonify
from flask_classy import FlaskView
from tigereye.models.cinema import Cinema

class CinemaView(FlaskView):
    def all(self):
        cinemas = Cinema.query.all()
        cinema_list = []
        for cinema in cinemas:
            cinema_list.append(cinema.__json__())
            return jsonify(cinemas_list)
       
"""
class CinemaView(FlaskView):
    def all(self):
        cinemas = Cinema.query.all()
        cinemas = [cinema.__json__() for cinema in cinemas]
        return jsonify(cinemas)
"""
```



#### 方法三：

#### 重写json中的JsonEncoder方法

##### 在models目录下\_\_init__.py中添加JsonEncoder类

```python
#重写flask中的json方法
from flask import json as _json

class JsonEncoder(_json.JSONEncoder):
    #重载default方法，以支持Model类对象的json序列化
    def default(self,o):
        #判断0是否是Model中的属性或方法
        if isinstance(o,Model):
            #返回重写的json方法
            return o.__json__()
        #返回系统中的json方法
        return _json.JSONEncoder.default(self,o)
```

##### 在app.py文件中加载重写的json方法

```python
from tigereye.models import db,JsonEncoder

def create_app():
	app.json_encoder = JsonEncoder
```

##### 对api目录下cinema.py中代码进行优化

```python
from flask import jsonify
from flask_classy import FlaskView
from tigereye.models.cinema import Cinema

class CinemaView(FlaskView):
    def all(self):
        cinemas = Cinema.query.all()
        return jsonify(cinemas)
```



#### 方法四：

#### 重写FlaskView类

##### 在借口文件api目录下的\_\_init__.py文件中添加代码

```python
from flask_classy import FlaskView
from flask import request, Response, jsonify, make_response
import functools

from tigereye.helper.code import Code


class ApiView(FlaskView):
    @classmethod
    def make_proxy_method(cls, name):
        """Creates a proxy function that can be used by Flasks routing. The
        proxy instantiates the FlaskView subclass and calls the appropriate
        method.

        :param name: the name of the method to create a proxy for
        """

        i = cls()
        view = getattr(i, name)

        if cls.decorators:
            for decorator in cls.decorators:
                view = decorator(view)

        @functools.wraps(view)
        def proxy(**forgettable_view_args):
            # Always use the global request object's view_args, because they
            # can be modified by intervening function before an endpoint or
            # wrapper gets called. This matches Flask's behavior.
            del forgettable_view_args

            if hasattr(i, "before_request"):
                response = i.before_request(name, **request.view_args)
                if response is not None:
                    return response

            before_view_name = "before_" + name
            if hasattr(i, before_view_name):
                before_view = getattr(i, before_view_name)
                response = before_view(**request.view_args)
                if response is not None:
                    return response

            #是否是response对象，如果不是，则进入自己的处理流程
            response = view(**request.view_args)
            if not isinstance(response, Response):
                response = jsonify(response)

            after_view_name = "after_" + name
            if hasattr(i, after_view_name):
                after_view = getattr(i, after_view_name)
                response = after_view(response)

            if hasattr(i, "after_request"):
                response = i.after_request(name, response)

            return response

        return proxy
```

##### 对api目录下cinema.py中代码进行优化

```python
from tigereye.models.cinema import Cinema
from tigereye.api import ApiView

class CinemaView(ApiView):
    def all(self):
        return Cinema.query.all()
```



### 在app.py文件中注册视图

```python
from tigereye.api.cinema import CinemaView

def create_app():
	CinemaView.register(app)
```

#### 扩展：

当有多个视图需要追个进行注册时

```python
from tigereye.api import ApiView

def create_app():
	#实例化注册视图的类
    configure_views(app)
    

#重新定义一个注册视图的类
def configure_views(app):
    from tigereye.api.misc import MiscView
    from tigereye.api.cinema import CinemaView
    from tigereye.api.movie import MovieView

    #依次注册本地中视图类
    for view in locals().values():
        #loacls()中包含app,app不是一个类,无法进行注册
        if type(view) == type and issubclass(view,ApiView):
            view.register(app)
```

#####  黑屏测试：

```
python manage.py runserver
```

##### 浏览器测试：

```
127.0.0.1:5000/cinema/all
```



### 封装小方法

封装get方法，在models目录下的\_\_init__.py文件中，为Model类添加方法

```python
class Model(object):
    @classmethod
    def get(cls,primary_key):
        return cls.query.get(primary_key)
```

在项目目录下创建helper目录，在该目录下创建code.py文件存放错误码

```python
from enum import Enum


#定义所有状态码
class Code(Enum):
    #成功
    succ = 0
    #缺少必要参数
    requir_paraneter_missing = 100
    #影院不存在
    cinnema_dose_exist = 200
```

修改借口文件api目录下的\_\_init__.py文件中的代码

```python
from flask_classy import FlaskView
from flask import request, Response, jsonify, make_response
import functools

from tigereye.helper.code import Code


class ApiView(FlaskView):
    @classmethod
    def make_proxy_method(cls, name):
        """Creates a proxy function that can be used by Flasks routing. The
        proxy instantiates the FlaskView subclass and calls the appropriate
        method.

        :param name: the name of the method to create a proxy for
        """

        i = cls()
        view = getattr(i, name)

        if cls.decorators:
            for decorator in cls.decorators:
                view = decorator(view)

        @functools.wraps(view)
        def proxy(**forgettable_view_args):
            # Always use the global request object's view_args, because they
            # can be modified by intervening function before an endpoint or
            # wrapper gets called. This matches Flask's behavior.
            del forgettable_view_args

            if hasattr(i, "before_request"):
                response = i.before_request(name, **request.view_args)
                if response is not None:
                    return response

            before_view_name = "before_" + name
            if hasattr(i, before_view_name):
                before_view = getattr(i, before_view_name)
                response = before_view(**request.view_args)
                if response is not None:
                    return response

            #是否是response对象，如果不是，则进入自己的处理流程
            response = view(**request.view_args)
            if not isinstance(response, Response):
                # response = jsonify(response)

                #读取response的类型
                response_type = type(response)
                #如果是元组，并且长度大于1，则视为是接口的错误提示
                if response_type is tuple and len(response) > 1:
                    #分别提取错误码，和数据
                    rc,_data = response
                    #返回错误码和数据提示的json响应
                    response = jsonify(
                        rc = rc.value,
                        msg = rc.name,
                        data = _data
                    )
                else:
                    #正常返回数据，此时response就是一个正常的列表或字典
                    response = jsonify(
                        rc = Code.succ.value,
                        msg = Code.succ.name,
                        data  = response
                    )
                response = make_response(response)



            after_view_name = "after_" + name
            if hasattr(i, after_view_name):
                after_view = getattr(i, after_view_name)
                response = after_view(response)

            if hasattr(i, "after_request"):
                response = i.after_request(name, response)

            return response

        return proxy
```





## 小案例

##### 实现：查询某个电影院中的各个观影听的信息

在借口文件cinema.py文件中添加借口

```python
from flask import jsonify,request
from flask_classy import FlaskView
from tigereye.models.cinema import Cinema
from tigereye.models.hall import Hall
from tigereye.api import ApiView
from tigereye.helper.code import Code

#重写FlaskView改为ApiView
class CinemaView(ApiView):

    #查询某个电影院中的各个观影厅的信息
    def halls(self):
        #获取url中的cid值
        cid = request.args["cid"]
        #查询cid对应电影院
        cinema = Cinema.get(cid)
        #获取不到cid影院，返回错误信息
        if not cinema:
            return Code.cinnema_dose_exist,request.args
        #讲观影厅的数据插入对应电影院的halls中
        cinema.halls = Hall.query.filter_by(cid=cid).all()
        return cinema

```

在app.py中导入视图，并注册

在manage.py文件中导入类

python    manage.py    shell      插入数据

python    manage.py     runserver	运行代码

127.0.0.1:5000/ciname/halls/?cid=1		浏览器测试



