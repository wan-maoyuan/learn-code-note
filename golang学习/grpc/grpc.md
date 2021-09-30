## GRPC简介

> 远程过程调用（Remote Procedure Call）：计算机调用远程内存中的函数就行调用本地函数一样。
>
> rpc原理图

![68278731d847eb1d6362043b4595890](..\..\imgs\68278731d847eb1d6362043b4595890.png)

> grpc是一种rpc，基于HTTP2.0开发，使用的接口描述语言IDL（Interface  Definition Language）是proto buffer。

## GRPC有四种服务类型(生命周期)

- unary

  > 客户端发送请求，服务端直接回应

- Client-side streaming

  > 客户端不断地上传请求到服务端，结束之后服务端返回一个响应

- Server-side streaming

  > 客户端上传一个请求，但是服务器源源不断的将响应返回给客户端

- Bidirectional streaming

  > 客户端不断地上传请求到服务器，服务器不断地返回响应给客户端