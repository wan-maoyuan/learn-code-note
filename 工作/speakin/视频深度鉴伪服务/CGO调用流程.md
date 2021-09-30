## 视频深度鉴伪头函数调用流程

### 头文件函数结构体使用步骤

#### 一、设置线程数

> 入参：传入一个int 类型（线程个数）
>
> 返回值：一个int类型（操作是否成功）

```c
EXPORT int dfdc_set_num_threads(int num_threads);
```



#### 二、新建一个```dfdc_handler```

> 入参：无
>
> 返回值：返回一个```handler``` 指针，需要判断返回值是否为空

```c
EXPORT void* dfdc_handler_new();
```



#### 三、```dfdc_handler``` 初始化

> 入参：```dfdc_handler``` 指针，配置文件的路径
>
> 返回值：返回一个int 类型，为0表示初始化成功，不为0表示初始化失败

```c
EXPORT int dfdc_handler_init(void* handler, const char* conf_dir);
```



#### 四、获取 ```dfdc_handler``` 引擎的信息

> 入参：```dfdc_handler``` 指针和```type``` 类型，```type``` 传0代表获取name，传1代表获取version，传2代表获取description。
>
> 返回值：返回对应的字符串信息

```c
EXPORT const char* dfdc_handler_info(void* handler, int type);
```



#### 五、创建一个 ```DfdcResult``` 结构体

> 入参：无
>
> 返回值：一个```DfdcResult``` 结构体

```c
typedef struct {
    float* score;
    unsigned int len;
} DfdcResult;

EXPORT DfdcResult dfdc_result_new();
```



#### 六、分析处理视频

> 入参：1、```handler``` 指针	2、视频的路径	3、结果结构体的指针
>
> 返回值：int类型，0表示成功，其他表示失败

```c
EXPORT int dfdc_process(void* handler, const char* video_path, DfdcResult* pred);
```



### 七、释放结果结构体的内存

> 入参：结果结构体的指针
>
> 返回值：无

```c
EXPORT void dfdc_result_free(DfdcResult* result);
```



#### 八、释放 ```dfdc_handler``` 视频深度鉴伪引擎的内存

> 入参：```dfdc_handler``` 结构体的指针
>
> 返回值：int类型，0表示成功，其他表示失败

```c
EXPORT int dfdc_handler_free(void** handler);
```

