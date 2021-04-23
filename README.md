# 开发说明

## 目录结构

```
root
├─app // 应用目录
├─bootstrap // 启动目录
├─config // 配置目录
├─database // 数据库
├─deploy // 发布
├─public // 网站根目录
├─resources // 前端资源
├─routes // 路由
├─storage // 本地存储
├─tests // 测试
└─vendor // 第三方包
```

项目开发一般使用的目录有**app、config、database、routes、storage**

### app 目录结构

```
app
├─Commons // 核心扩展功能目录
├─Console // 命令行、任务调度
├─Exceptions // 异常处理
├─Http // 网络层
├─Jobs // 队列
├─Logics // 业务逻辑层
└─Traits // 业务特性层
```

正常工程保持如上目录结构，如果需要事件处理，则增加 EventBus 事件总线目录

### Http 网络层

```
Http
├─Controllers // 控制器
├─Middleware  // 中间件
└─Requests  // 请求对象
```

1. 网络层只能触发业务，不能包涵有业务逻辑，以便不同的接口能调用相同的业务逻辑
2. 参数验证和用户登录态信息只能在控制器层获取，这些不属于业务逻辑，请通过参数传递到业务层
3. 网络层可增加 Resources 目录，提供对响应数据结构做转换（不同的接口可能调用相同的业务逻辑，但返回结果不一致）

### Logics 业务逻辑层

业务逻辑层调用一个或多个领域服(Domain)务完成应用业务，对于数据展示等业务(主要为查询服务)，通过业务特性层(Traits)实现

**数据展示服务变动较大，不应该整合到领域服务层，后续可配合大数据统计服务实现各种报表**

### Controllers 与 Requests、Logics 与 Traits 关系

1. 一个控制器(Controllers)对应一个业务逻辑(Logics)；
2. 一个控制器对应多个请求对象(Requests)，即一个 Controller 文件有一个对应的 Requests 目录；
3. 一个业务逻辑对应多个业务特性，即一个 Logic 文件有一个对应的 Traits 目录；

```
Http
├─Controllers
│  └─Api
│    └─Login
│      └─LoginController.php
├─Requests
│  └─Api
│    └─Login // 目录
Logics
├─Api
│  └─Login
│    └─LoginLogic.php
Traits
└─Api
   └─Login // 目录
```

### Commons 核心扩展功能目录

```
Commons
├─Config // 全局配置目录
├─Console // 全局命令、任务调度
├─Domain // 领域层
├─Exceptions // 全局异常处理
├─Libs // 扩展
├─Http // 全局网络层（中间件）
├─Validators // 参数验证器
└─Providers // 全局服务提供者
```

1. Config 目录为默认配置，将合并工程目录下的 config，并覆盖相同键值的配置；
2. Domain 为业务领域，主要处理核心业务
3. Libs // 扩展提供各种非业务的基础功能封装
4. Http // 提供网络层路由中间件
5. Providers // 核心服务提供者
6. Validators // 扩展 laravel 请求参数验证功能

Commons 目录提供了一个可持续开发的项目模板，放在任何版本的 Laravel 下面都能提供一致的架构基础，从而快速配置新的 Laravel 项目

## 路由定义规则

1. 路由只能使用小写字母，多个单词使用“-”进行连接，禁止使用下划线；
2. 路由必须包括控制器和方法名，如 /api/login/logout login 为控制器，logout 为方法；
3. 路由地址可添加多层目录进行分类，如 /api/system/log/list 对应方法路径为 Api/System/LogController@list；
4. 接口请求方式使用 Get 和 Post，restful 接口除外；
5. 路由中的控制器名使用单数，restful 接口必须使用复数，如 /api/login/logout 为普通接口，/api/articles 为 restful 接口；

### restful 接口定义（管理后台使用）

restful 接口一般在管理后台使用，对应数据的增删改查，其他应用的接口不推荐使用
restful 接口定义规则如下：

-   get /api/articles 文章列表，对应方法 index
-   post /api/articles 创建文章, 对应方法 add
-   get /api/articles/{aritcle_id} 获取文章信息，对应方法 get
-   put /api/articles/{article_id} 更新文章, 对应方法 update
-   delete /api/articles/{aritcle_id} 删除文章，对应方法 delete

### 代码生成器使用

使用 Artisan 命令 pm:run 即可
选择从 postman 的集合目录中生成网络层代码，可以选择多级目录，选择「生成代码」或最后一级目录即开始生成代码

## 用户日志

用户日志采用控制器方法添加注解的方式实现，如

```php
    use App\Commons\Extensions\Annotations\UserLog; // 必须引用注解命名空间
    /**
     * @UserLog(type=1, tpl="{{mobile}},{{type}}审核提现,订单号{{order_no}},签名{{sign}}")
     * @param  TestRequest $request [description]
     * @return [type]               [description]
     */
    public function index(TestRequest $request)
    {
        // 设置日志模板变更
        user_log(['order_no' => 'test', 'sign' => 'sign']);
        // 或
        user_log('order_no', 'test');
        user_log('sign', 'sign');
    }
```

### 内置注解模板变量

在用户登录状态下，登录用户的如下信息将自动添加到模板变量，可直接使用：

-   uid
-   mobile
-   username
-   nickname

## 参数获取

参数获取推荐两种方式

### 获取 value 数组

```php
[$username, $page] = $request->params(['username', 'page'])
```

### 获取 key-value 数组

```php
$param = $request->params();    // $param=['username' => 'xxx', 'page' => xx]
```

## 响应资源映射

## 抛出异常

```php
// 直接抛出错误码
throw_e(0x000001);
// 抛出异常信息
throw_e('异常信息');
// 空条件抛出
throw_empty($user, 0x000001);   // $user变量为空则抛出异常
throw_empty($user, '异常信息');   // 同上
// 判断条件抛出
throw_on($user->status === -1, 0x000001);
// 或
throw_on($user->status === -1, '异常信息');
```

## 多语言实现

### 添加多语言中间件

1. 创建中间件 ApiLocale
2. 中间件判断请求头携带的语言信息，设置当前请求的语言
3. 将中间件添加到 api 全局中间件
4. 在目录 resources.lang 目录下编写对应语言的 json 文件
5. 分页、参数验证国际化参考 en 目录创建对应文件即可
6. error_code 国际化移到文件 Commons/Libs/ErrorCode/config.php 中
