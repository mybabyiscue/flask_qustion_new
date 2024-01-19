# flask-SQLAlchemy笔记
[flask-SQLAlchemy笔记](https://zlkt.net/book/detail/10/297)

## 一、SQLAlchemy和flask-SQLAlchemy的区别
1、SQLAlchemy：是一个独立的ORM框架，让我们操作数据库的时候不要再用SQL语句了，跟直接操作模型一样。安装命令为：pip install SQLAlchemy
可以独立于flask存在，也可以在其他项目中使用，比如在Django中。

2、Flask-SQLAlchemy：对SQLAlchemy的一个封装，能够更加适合在flask中使用

## 二、安装和验证
```
1、连接数据库的库：pip install pymysql
1、pip install flask-sqlalchemy

```

## 三、连接数据库
```
HOSTNAME = '127.0.0.1'
PORT = '3306'
DATABASE = 'xt_flask'
USERNAME = 'admin'
PASSWORD = 'cstorfs'
DB_URI = 'mysql+pymysql://{}:{}@{}:{}/{}?charset=utf8'.format(USERNAME, PASSWORD, HOSTNAME, PORT, DATABASE)
app.config['SQLALCHEMY_DATABASE_URI']=DB_URI # 配置数据库连接
```

## 四、Flask-Migrate插件数据库迁移
在实际的开发环境中，经常会发生数据库修改的行为。一般我们修改数据库不会直接手动的去修改，而是去修改ORM对应的模型，然后再把模型映射到数据库中。这时候如果有一个工具能专门做这种事情，就显得非常有用了，而flask-migrate就是做这个事情的。flask-migrate是基于Alembic进行的一个封装，并集成到Flask中，而所有的迁移操作其实都是Alembic做的，他能跟踪模型的变化，并将变化映射到数据库中。
使用Flask-Migrate需要安装，命令如下：
```pip
pip install Flask-Migrate
```
要让Flask-Migrate能够管理app中的数据库，需要使用Migrate(app,db)来绑定app和数据库。假如现在有以下app文件：
```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from constants import DB_URI
from flask_migrate import Migrate

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = DB_URI
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = True
db = SQLAlchemy(app)
# 绑定app和数据库
migrate = Migrate(app,db)

class User(db.Model):
    id = db.Column(db.Integer,primary_key=True)
    username = db.Column(db.String(20))

    addresses = db.relationship('Address',backref='user')

class Address(db.Model):
    id = db.Column(db.Integer,primary_key=True)
    email_address = db.Column(db.String(50))
    user_id = db.Column(db.Integer,db.ForeignKey('user.id'))

db.create_all()

@app.route('/')
def hello_world():
    return 'Hello World!'

if __name__ == '__main__':
    app.run()
```

```
#之后，就可以在命令行中映射ORM了。首先需要初始化一个迁移文件夹：
flask db init
#然后再把当前的模型添加到迁移文件中：
flask db migrate -m "XXX"
#最后再把迁移文件中对应的数据库操作，真正的映射到数据库中：
flask db upgrade
```