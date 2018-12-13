### 使用Swagger编写API文档

[Swagger](http://swagger.io/)是一个Restful风格接口的文档在线自动生成和测试的框架。

#### 环境搭建

下载`Swagger UI`

```
git clone https://github.com/swagger-api/swagger-ui.git
```

##### node服务

创建一个空文件夹`node_app`

```
mkdir node_app
```

初始化`node`，创建`package.json`文件

```
cd node app
npm init
```

安装`express`

```
npm install express --save
```

创建`index.js`，把下面代码贴入`index.js`中

```javascript
var express = require('express');
var app = express();
app.get('/', function (req, res) {
  res.send('Hello World!');
});

app.listen(3000, function () {
  console.log('Example app listening on port 3000!');
});
```

在`node_app`中创建空目录`public`

```
mkdir public
cd public
```

修改路由

```javascript
//在index.js的第三行插入下面这句话
app.use('/static', express.static('public'));
```

把下载好的`Swagger UI`文件中`dist`目录下的文件全部复制到`public`文件夹下。启动`node`服务。

```
node index.js
```

打开浏览器，输入[http://localhost:3000/static/index.html](https://link.jianshu.com/?t=http://localhost:3000/static/index.html)

至此，我们已经将官方的`demo`在本地配置好了。

#### Swagger Editor

使用[Swagger Editor](http://editor.swagger.io/#/)编写API文档，`Swagger`可以选择使用`JSON`或者`YAML`的语言格式来编写`API`文档。可以参考这篇文章[Swagger从入门到精通](https://legacy.gitbook.com/book/huangwenchao/swagger/details)熟悉`yaml`的语法。

```yaml
#必要字段！Swagger规范版本，必须填2.0，否则该YAML将不能用于Swagger其他组件
swagger: '2.0'
#Swagger会提供测试用例，host指定测试时的主机名，如果没有指定就是当前主机,可以指定端口．
host: 127.0.0.1
#定义的api的前缀，必须已/开头,测试用例的主机则为:host＋bashPath
basePath: /api
#必要字段！描述API接口信息的元数据
info:
  #接口文档的描述
  description: API示例，关于yaml语法的说明
  #版本号
  version: 0.0.1
  #接口标题
  title: API文档示例
tags:
  - name: user
    description: 用户模块接口
  - name: order
    description: 订单模块接口
#指定调用接口的协议，必须是:"http", "https", "ws", "wss"．默认是http.-表示是个数组元素，即schemes接受一个数组参数
schemes:
  - https
  - http
produces:
  - application/json
#必要字段!定义API
paths:
  /searchUsers:
    #必要字段!定义HTTP操作方法，必须是http协议定义的方法
    post:
      #标签，方便快速过滤出User相关的接口
      tags:
        - user
      #接口概要
      summary: 查询用户接口
      #接口描述
      description: 根据年龄以及收藏查询用户集合
      #请求参数
      parameters:
        #参数key
        - name: age
          #传递方法，formData表示表单传输，还有query表示url拼接传输，path表示作为url的一部分
          #body表示http头承载参数(body只能有一个,有body不能在有其他的)
          in: query
          description: 年龄
          #参数类型，可选的包括array,integer,boolean,string.使用array必须使用items
          type: string
        - name: favorites
          in: query
          description: 收藏夹
          type: array
          items:
            type: string
      #返回值描述，必要字段
      responses:
        #返回的http状态码
        '200':
          description: 请求成功
          #描述返回值
          schema:
            # 必要参数
            required:
              - code
              - message
              - data
            properties:
              code:
                 #返回值格式，可选的有array,integer,string,boolean
                type: integer
                description: 响应状态码
              message:
                type: string
                description: 响应描述
              data:
                type: object
                $ref: '#/definitions/Page'
  /getOrder:
    get:
      tags:
        - order
      summary: 获取用户订单详情
      description: 获取用户订单详情
      parameters:
        - name: id
          in: query
          description: 订单ID
          type: string
      responses:
        '200':
          description: 请求成功
          schema:
            required:
              - code
              - message
              - data
            properties:
              code:
                type: integer
                description: 响应状态码
                example: 200
              message:
                type: string
                description: 响应描述
                example: 'success'
              data:
                type: object
                $ref: '#/definitions/Order'
definitions:
  Page:
    type: object
    properties:
      pageNum:
        type: integer
        description: 当前页
        example: 1
      pageSize:
        type: integer
        description: 每页的数量
        example: 10
      size:
        type: integer
        description: 当前页的数量
        example: 10
      startRow:
        type: integer
        description: 当前页面第一个元素在数据库中的行号
        example: 1
      endRow:
        type: integer
        description: 当前页面最后一个元素在数据库中的行号
        example: 10
      pages:
        type: integer
        description: 总页数
        example: 4
      total:
        type: number
        description: 总记录数
        example: 33
      firstPage:
        type: integer
        description: 第一页
        example: 1
      prePage:
        type: integer
        description: 前一页
        example: 1
      isFirstPage:
        type: boolean
        description: 是否为第一页
        example: true
      isLastPage:
        type: boolean
        description: 是否为最后一页
        example: false
      hasPreviousPage:
        type: boolean
        description: 是否有前一页
        example: false
      hasNextPage:
        type: boolean
        description: 是否有下一页
        example: true
      navigatePages:
        type: integer
        description: 导航页码数
        example: 8
      navigatepageNums:
        type: array
        items:
          type: integer
        description: 所有导航页号
        example: []
      list:
        type: array
        description: 结果集
        items:
          $ref: '#/definitions/User'
        example: [
      {
        "id": "100860001",
        "userName": "test",
        "sex": "female",
        "age": 18,
        "favorites": ['LOL','王者荣耀','DOTA'],
        "email": "test@123.com",
      },{
        "id": "100860002",
        "userName": "qwer1",
        "sex": "male",
        "age": 22,
        "favorites": ['LOL','NBA','CS'],
        "email": "qwe1@123.com",
      },{
        "id": "100860003",
        "userName": "flwcy",
        "sex": "male",
        "age": 28,
        "favorites": ['LOL','NBA','DOTA'],
        "email": "flwcy@123.com",
      }
    ]
  User:
    type: object
    properties:
      id:
        type: string
        description: 用户ID
      userName:
        type: string
        description: 用户名
      sex:
        type: string
        description: 性别
        # 定义枚举，sex的值只能从l两个枚举值中选择
        enum:
          - male
          - female
      age:
        type: integer
        description: 年龄
      favorites:
        type: array
        description: 收藏夹
        items:
          type: string
      email:
        type: string
        description: 邮箱
  Order:
    type: object
    properties:
      id:
        type: string
        description: 订单ID
      price:
        type: number
        description: 价格
      productCount:
        type: number
        description: 商品数量
      productId:
        type: string
        description: 商品ID
      productName:
        type: string
        description: 商品名称
      status:
        type: integer
        description: 订单状态
      userId:
        type: string
        description: 用户ID
      createTime:
        type: string
        description: 创建时间
    example: {
      id: '10010001',
      price: 888,
      productCount: 24,
      productId: '11012011900',
      productName: 'iphone',
      status: 1,
      userId: '100860003',
      createTime: '2018-12-12 23:59:59'
    }
```

导出`JSON`文件`test.json`，把 `test.json`放到`node_app/public`目录下。修改`node_app/public/index.html`，将`url = "http://petstore.swagger.io/v2/swagger.json";`改为`url = "/static/test.json";`。

重启`node`服务，浏览器中打开`http://localhost:3000/static/index.html`就是我们写的`API`文档了。

#### nginx下部署

我们也可以采用`nginx`来部署静态`API`文档，首先下载并解压`nginx`，将之前`swagger-ui/dist`目录下的所有文件复制到`nginx/html`目录下，将`test.json`也复制到该目录下，修改`index.html`文件，将`url = "http://petstore.swagger.io/v2/swagger.json";`改为`url = "/test.json";`。修改`nginx`监听的端口号：

```
    server {
        listen       8888;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }
```

