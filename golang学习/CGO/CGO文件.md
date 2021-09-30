### CGO文件头

```go
/*
#cgo LDFLAGS: -lgender -lpthread
#cgo LDFLAGS: -Wl,-rpath=$ORIGIN:$ORIGIN/lib:$ORIGIN/3rd/lib
#
#include <stdlib.h>
#
#include "gender/gender_recognition_interface.h"
*/
```



### CGO编译参数

```shell
CGO_CFLAGS="-g -O2 -I /home/wan/go/src/gender-service/3rd/include" 														
CGO_LDFLAGS="-g -O2 -L /home/wan/go/src/gender-service/3rd/lib/gender -Wl,-rpath=.:./lib/:./3rd/lib/:./3rd/lib/gender/" 	
go build -ldflags "-s -w -X 'main.Version=v1.0.3-dirty' -X 'main.CommitHash=f0a97a28d0e7531c5cef7e65bc16470a1e1f75aa' -X 'main.BuildTime=20210916103416'" 
-o dist/gender-service/gender-service 
```

> ```CGO_CFLAGS="-g -O2 -I /home/wanmaoyuan/gocode/src/gender-service/3rd/include"```指定头文件位置

> ```CGO_LDFLAGS="-g -O2 -L /home/wanmaoyuan/gocode/src/gender-service/3rd/lib/gender -Wl,-rpath=.:./lib/:./3rd/lib/:./3rd/lib/gender/"```指定动态链接库的位置



### CGO运行测试时，编译运行时找不到动态链接库

> /home/wan/go/src/gender-go/3rd/lib/gender下的动态链接库，无法加载共享链接库

```shell
CGO_CFLAGS="-g -O2 -I /home/wan/go/src/gender-go/3rd/include/" 
CGO_LDFLAGS="-g -O2 -L /home/wan/go/src/gender-go/3rd/lib/gender/ -Wl,-rpath=.:./lib:./3rd/lib/gender" 
go test -v ./...
/tmp/go-build2518108419/b001/gender.test: error while loading shared libraries: libgender.so: cannot open shared object file: No such file or directory
FAIL    git.speakin.mobi/capacity/gender-go/gender      0.005s
FAIL
make: *** [Makefile:24: test] Error 1
```

> 需要设置一下程序共享库位置，再执行编译运行

```shell
export LD_LIBRARY_PATH=$PWD/3rd/lib/gender
```

查看当前环境的共享库

```shell
echo $LD_LIBRARY_PATH
```































