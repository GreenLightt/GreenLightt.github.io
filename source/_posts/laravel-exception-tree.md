---
title: Laravel 异常树
date: 2017-11-18 11:57:41
description: 整理 Laravel 可能出现的 Exception, 及捕捉异常中间件、跨域中间件
tags:
- Laravel-5.4
categories:
- Laravel
copyright: false
---

# 异常树
- `Exception `
    - `ErrorException`
    - `RuntimeException` 运行时异常
        - `UnexpectedValueException` 未预期值异常
        - `Symfony\Component\HttpKernel\Exception\HttpException`
            - `Symfony\Component\HttpKernel\Exception\NotFoundHttpException`
            - `Symfony\Component\HttpKernel\Exception\MethodNotAllowedHttpException`
        - `Illuminate\Http\Exception\HttpResponseException`  http response 异常
        - `Illuminate\Database\Eloquent\ModelNotFoundException` 找不到 Model 异常
        - `Illuminate\Database\Eloquent\MassAssignmentException`
        - `Illuminate\Contracts\Encryption\DecryptException` 解密异常
        - `Illuminate\Contracts\Encryption\EncryptException` 加密异常
        - `Illuminate\Contracts\Validation\ValidationException`
        - `Illuminate\Contracts\Validation\UnauthorizedException`
    - `LogicException `
        - `InvalidArgumentException` 参数错误异常
            - `Illuminate\Contracts\Queue\EntityNotFoundException`
        - `BadFunctionCallException`
            - `BadMethodCallException` 调用方法异常
    - `PDOException`
        - `Illuminate\Database\QueryException` 数据库查询异常
    - `Illuminate\Auth\Access\UnauthorizedException` 未认证用户
    - `Illuminate\Contracts\Filesystem\FileNotFoundException` 文件不存在
    - `Illuminate\Session\TokenMismatchException` Token不匹配异常
    - `Illuminate\Http\Exception\PostTooLargeException` post 数据量大异常
    - `Illuminate\Container\BindingResolutionException`
        - `Illuminate\Contracts\Container\BindingResolutionException`
         

# 异常概要
## `Exception`
`Exception` 是所有异常的基类。 

```
Exception {
    /* 属性 */
    protected string $message ;   // 异常消息内容
    protected int $code ;         // 异常代码
    protected string $file ;      // 抛出异常的文件名
    protected int $line ;         // 抛出异常在该文件中的行号
    /* 方法 */
    public __construct ([ string $message = "" [, int $code = 0 [, Throwable $previous = NULL ]]] )
    final public string getMessage ( void )        // 获取异常消息内容
    final public Throwable getPrevious ( void )    // 返回异常链中的前一个异常
    final public int getCode ( void )              //  获取异常代码
    final public string getFile ( void )           // 创建异常时的程序文件名称
    final public int getLine ( void )              // 获取创建的异常所在文件中的行号
    final public array getTrace ( void )           // 获取异常追踪信息
    final public string getTraceAsString ( void )  // 获取字符串类型的异常追踪信息
    public string __toString ( void )              // 将异常对象转换为字符串
    final private void __clone ( void )            // 异常克隆
}
```

# 异常捕捉中间件
首先，定义中间件 `CatchExceptionMiddleware`

```
use Closure;
use Exception;
use stdClass;
use Illuminate\Http\Response;

class CatchExceptionMiddleware {
    public function handle($request, Closure $next) {
        try {
            $response = $next($request);
            return $response;
        } catch (Exception $e) {
            $response = new Response([
                'status' => 500,
                'msg' => $e->getCode() . " : " . $e->getMessage(),
                "data" => new stdClass()
            ], 200);
            return $response;
        }
    }
}
```

然后在 某个`ServiceProvider` 中的 `boot` 方法执行注册：

```
use Illuminate\Routing\Router;

...

    public function boot(Router $router){
        $router->middleware('catch_exception', CatchExceptionMiddleware::class);
    }
```

# 支持跨域中间件

```
use Closure;

class EnableCrossRequestMiddleware {

    public function handle($request, Closure $next) {
        $response = $next($request);
        
        $response->header('Access-Control-Allow-Origin', config('app.cross_allow_origin', '*'));
        $response->header('Access-Control-Allow-Headers', 'Origin, Content-Type, Cookie, Accept');
        $response->header('Access-Control-Allow-Methods', 'GET, POST, PATCH, PUT, OPTIONS');
        // $response->header('Access-Control-Allow-Credentials', 'true');
        
        return $response;
  }

}
```
其中有以下需要注意的地方：

- 对于跨域访问并需要伴随认证信息的请求，需要在 `XMLHttpRequest` 实例中指定 `withCredentials` 为 `true`；
- 这个中间件你可以根据自己的需求进行构建，如果需要在请求中伴随认证信息（包含 `cookie`，`session`）那么你就需要指定 `Access-Control-Allow-Credentials` 为 `true`, 因为对于预请求来说如果你未指定该响应头，那么浏览器会直接忽略该响应；
- 在响应中指定 `Access-Control-Allow-Credentials` 为 true 时，`Access-Control-Allow-Origin` 不能指定为 `*`；
- 后置中间件只有在正常响应时才会被追加响应头，而如果出现异常，这时响应是不会经过中间件的；


如果想要设置 `Access-Control-Allow-Origin` 的值为 请求的 `referer` 值：
```
$request_referer_info = parse_url($request->header('Referer'));
$request_referer = count($request_referer_info) > 1
    ? $request_referer_info['scheme'] . '://' . $request_referer_info['host'] 
        . (isset($request_referer_info['port']) ? ':' . $request_referer_info['port'] : '')
    : '*'
    
$response->header('Access-Control-Allow-Origin', $$request_referer);
```

参考阅读 [Laravel 开启跨域功能](https://segmentfault.com/a/1190000006207353)；





