## CGO编译和链接参数

### 编译参数：CFLAGS/CPPFLAGS/CXXFLAGS

> 编译参数主要是头文件的检索路径，预定义的宏等参数。

- CFLAGS对应C语言的编译参数
- CPPFLAGS对应c/c++的编译参数
- CXXFLAGS对应纯c++的编译参数

### 链接参数：LDFLAGS

> 链接参数主要包含要链接库的检索目录和要链接库的名字。因为历史遗留问题，链接库不支持相对路径，我们必须为链接库指定绝对路径。 cgo 中的 ${SRCDIR} 为当前目录的绝对路径。经过编译后的C和C++目标文件格式是一样的，因此LDFLAGS对应C/C++共同的链接参数.



## #cgo语句

> C头文件可以是相对目录，但是库文件检索目录则需要绝对路径。在库文件的检索目录中可以通过`${SRCDIR}`变量表示当前包目录的绝对路径：

```go
// #cgo LDFLAGS: -L${SRCDIR}/libs -lfoo
链接时会展开为
// #cgo LDFLAGS: -L/go/src/foo/libs -lfoo
```



### Linux编译命令

```shell
CGO_CFLAGS="-g -O2 -I /home/wanmaoyuan/gocode/src/waveform-checker-service/3rd/include" CGO_LDFLAGS="-g -O2 -L /home/wanmaoyuan/gocode/src/waveform-checker-service/3rd/lib/waveform-checker -Wl,-rpath=.:./lib:./3rd/lib/waveform-checker" go build -ldflags "-s -w -X 'main.Version=v1.0.7-17-g85453ed-dirty' -X 'main.CommitHash=85453ed35dd7b71b7b5983e784e924296257abe3' -X 'main.BuildTime=20210929172522'" -o dist/waveform-checker-service/waveform-checker-service 
```



### Windows编译命令

```shell
set CGO_CFLAGS="-g -O2 -I E:\workSpace\code\go\src\deep-fake-detection\3rd\include"  CGO_LDFLAGS="-g -O2 -L E:\workSpace\code\go\src\deep-fake-detection\3rd\bin -Wl,-rpath=.\3rd\bin" go build -ldflags "-s -w -X 'main.Version=v1.0.7-17-g85453ed-dirty' -X 'main.CommitHash=85453ed35dd7b71b7b5983e784e924296257abe3' -X 'main.BuildTime=20210929172522'" -o deep-fake-detection
```









































