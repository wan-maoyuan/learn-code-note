## CGO中的编译选项

- CGO文件配置

  ```go
  package hcnet
  
  /*
  #cgo CFLAGS: -I${SRCDIR}
  #cgo linux LDFLAGS: -L${SRCDIR}/../../3rdParty/Linux/hc_net_sdk/lib -lhcnetsdk -lhpr -lHCCore -Wl,-rpath=/usr/local/lib
  #cgo linux LDFLAGS: -L${SRCDIR}/../../3rdParty/Linux/hc_net_sdk/lib/HCNetSDKCom -lHCGeneralCfgMgr -lHCIndustry -Wl,-rpath=/usr/local/lib/HCNetSDKCom
  #cgo windows LDFLAGS: -L${SRCDIR}/../../3rdParty/Windows/hc_net_sdk/lib -lHCCore -lHCNetSDK -lhpr -Wl,-rpath=./
  #cgo windows LDFLAGS: -L${SRCDIR}/../../3rdParty/Windows/hc_net_sdk/lib/HCNetSDKCom -lHCGeneralCfgMgr -lHCIndustry -Wl,-rpath=./HCNetSDKCom
  #include "HCNetSDK.h"
  //#include <stdio.h>
  #include <stdlib.h>
  //#include <unistd.h>
  extern void fRemoteConfigCallbackCgo(DWORD dwType, void* lpBuffer, DWORD dwBufLen, void* pUserData);
  extern BOOL MSGCallBackV31Cgo(LONG lCommand, NET_DVR_ALARMER *pAlarmer, char *pAlarmInfo, DWORD dwBufLen, void* pUser);
  */
  import "C"
  import (
      "encoding/hex"
      "fmt"
      "unsafe"
  )
  ```

- 参数解析

  > ```CFLAGS```是引入头文件
  > ```${SRCDIR}```是指当前 go 文件所在的文件夹
  > ```LDFLAGS```引入链接库
  > -L 链接库路径
  > -l 链接库名
  > linux 和 windows 是跨平台选项
  > ```rpath``` 指编译出的可执行文件依赖的链接库路径

- 在CGO文件中配置好了头文件，链接库等配置，可以执行下面命令进行编译

  ```shell
  CGO_ENABLED=1 GOOS=linux GOARCH=amd64 CC=gcc go build -o . .
  ```

- 也可以直接在编译命令中配置

  ```shell
  CGO_CFLAGS="-g -O2 -I /home/wan/go/src/gender-go/3rd/include/"  CGO_LDFLAGS="-g -O2 -L /home/wan/go/src/gender-go/3rd/lib/gender/ -Wl,-rpath=.:./lib:./3rd/lib/gender"  go build -o dist/gender/gender-cgo
  ```

  

## CGO中的内置函数

- CGO中内置一下函数

  ```go
  // Go string to C string
  // The C string is allocated in the C heap using malloc.
  // It is the caller's responsibility to arrange for it to be
  // freed, such as by calling C.free (be sure to include stdlib.h
  // if C.free is needed).
  func C.CString(string) *C.char
  
  // Go []byte slice to C array
  // The C array is allocated in the C heap using malloc.
  // It is the caller's responsibility to arrange for it to be
  // freed, such as by calling C.free (be sure to include stdlib.h
  // if C.free is needed).
  func C.CBytes([]byte) unsafe.Pointer
  
  // 将go语言中的bool类型转换成 C语言中的bool类型
  func C.bool(bool) C.bool
  
  // C string to Go string
  func C.GoString(*C.char) string
  
  // C data with explicit length to Go string
  func C.GoStringN(*C.char, C.int) string
  
  // C data with explicit length to Go []byte
  func C.GoBytes(unsafe.Pointer, C.int) []byte
  ```

- CString

  > ==必须释放内存==
  >
  > 由于使用了`free`函数，所以需要`#include <stdlib.h>`头文件。

  ```go
  sDVRIP := C.CString(ip)
  defer C.free(unsafe.Pointer(sDVRIP))
  ```

  

- CBytes

  > ==必须释放内存==
  >
  > 由于使用了`free`函数，所以需要`#include <stdlib.h>`头文件。

  ```go
  sAddress := C.CBytes(address)
  defer C.free(sAddress)
  ```

  

- GoString

  ```go
  // char* ip = "192.168.1.168"
  sAddress := C.GoString(ip)
  ```

  

- 数组

  ```go
  // char ip[20] = "192.168.1.168"
  sAddress := C.GoString(&ip[0])
  ```

  

## CGO中 ```void*``` 的参数实现 

- 入参是 ```void*``` 时

  > 有如下函数,其中pSendBuf会根据dwDataType类型的不同，传送不同类型的结构体指针;dwBufSize是结构体的大小。

  ```c
  BOOL NET_DVR_SendRemoteConfig(LONG lHandle, DWORD dwDataType, char *pSendBuf, DWORD dwBufSize);
  ```

  > 以下是cgo的实现
  > go中的入参是`interface{}`类型，在cgo中通过断言获取data的类型，并转换为C的类型。
  > **注意C结构体大小的计算方法**

  ```go
  // SendRemoteConfig 发送长连接数据
  func SendRemoteConfig(lHandle int32, dwDataType uint32, data interface{}) (err error) {
      var lpInBuffer C.LPVOID
      var inBufferSize uint32
      switch value := data.(type) {
      case *CardCfgV50:
          var temp C.NET_DVR_CARD_CFG_V50
          inBufferSize = uint32(unsafe.Sizeof(temp))
          lpInBuffer = C.LPVOID(unsafe.Pointer(GoCCardCfgV50(value)))
      case *CardCfgSendData:
          var temp C.NET_DVR_CARD_CFG_SEND_DATA
          inBufferSize = uint32(unsafe.Sizeof(temp))
          lpInBuffer = C.LPVOID(unsafe.Pointer(GoCCardCfgSendData(value)))
      default:
          err = fmt.Errorf("data type %T not support now", value)
          return
      }
      ret := C.NET_DVR_SendRemoteConfig(C.LONG(lHandle), C.DWORD(dwDataType), (*C.char)(lpInBuffer), C.DWORD(inBufferSize))
      if ret != TRUE {
          err = GetLastError()
          return
      }
      return
  }
  ```

  

- 当出参是 ```void*``` 时

  > 有如下函数,其中lpOutBuffer会根据dwCommand类型的不同，传送不同类型的结构体指针;dwOutBufferSize是结构体的大小。

  ```c
  BOOL NET_DVR_GetDVRConfig(LONG lUserID, DWORD dwCommand, LONG lChannel, LPVOID lpOutBuffer, DWORD dwOutBufferSize, LPDWORD lpBytesReturned);
  ```

  > 以下是cgo的实现
  > 由于没有入参，所以根据dwCommand的类型创建出参的C的结构体指针。
  > 返回时，将C的结构体转换为go的结构体，并作为go的返回参数。

  ```go
  // GetDVRConfig 获取设备的配置信息
  func GetDVRConfig(lUserID int, dwCommand uint32, lChannel uint32) (out interface{}, bytesReturned uint32, err error) {
      var lpOutBuffer C.LPVOID
      outBufferSize := uint32(0)
      switch dwCommand {
      case NetDvrGetAcsCfg:
          var temp C.NET_DVR_ACS_CFG
          outBufferSize = uint32(unsafe.Sizeof(temp)) // 单纯计算size， 无法使用类型计算
          lpOutBuffer = C.LPVOID(unsafe.Pointer(&C.NET_DVR_ACS_CFG{}))
      case NetDvrGetDoorCfg:
          var temp C.NET_DVR_DOOR_CFG
          outBufferSize = uint32(unsafe.Sizeof(temp)) // 单纯计算size， 无法使用类型计算
          lpOutBuffer = C.LPVOID(unsafe.Pointer(&C.NET_DVR_DOOR_CFG{}))
      case NetDvrGetAcsWorkStatusV50:
          var temp C.NET_DVR_ACS_WORK_STATUS_V50
          outBufferSize = uint32(unsafe.Sizeof(temp)) // 单纯计算size， 无法使用类型计算
          lpOutBuffer = C.LPVOID(unsafe.Pointer(&C.NET_DVR_ACS_WORK_STATUS_V50{}))
      default:
          err = fmt.Errorf("command %d not support now", dwCommand)
          return
      }
      cBytesReturned := C.DWORD(0)
      ret := C.NET_DVR_GetDVRConfig(C.LONG(lUserID), C.DWORD(dwCommand), C.LONG(lChannel), lpOutBuffer, C.DWORD(outBufferSize), &cBytesReturned)
      if ret != TRUE {
          err = GetLastError()
          return
      }
      bytesReturned = uint32(cBytesReturned)
      switch dwCommand {
      case NetDvrGetAcsCfg:
          out = CgoDvrAcsCfg(C.LPNET_DVR_ACS_CFG(lpOutBuffer))
      case NetDvrGetDoorCfg:
          out = CgoDoorCfg(C.LPNET_DVR_DOOR_CFG(lpOutBuffer))
      case NetDvrGetAcsWorkStatusV50:
          out = CgoAcsWorkStatusV50(C.LPNET_DVR_ACS_WORK_STATUS_V50(lpOutBuffer))
      default:
          err = fmt.Errorf("command %d not support now", dwCommand)
          return
      }
      return
  }
  ```



## CGO中 callback 函数的实现

- callback 的实现

  > 链接库提供了函数进行回调函数赋值
  > **HCNetSDK.h**

  ```c
  typedef BOOL (*MSGCallBack_V31)(LONG lCommand, NET_DVR_ALARMER *pAlarmer, char *pAlarmInfo, DWORD dwBufLen, void* pUser);
  BOOL NET_DVR_SetDVRMessageCallBack_V31(MSGCallBack_V31 fMessageCallBack, void* pUser);
  ```

  > 需要使用C语言实现回调函数，并导出回调函数的声明，让go去实现
  > **cfunctions.go**

  ```go
  // Wrappers for Go callback functions to be passed into C.
  package hcnet
  
  /*
  #cgo CFLAGS: -I${SRCDIR}
  #include "HCNetSDK.h"
  
  // export from go
  extern BOOL MSGCallBackV31Go(LONG lCommand, NET_DVR_ALARMER *pAlarmer, char *pAlarmInfo, DWORD dwBufLen, void* pUser);
  
  // WRAP 函数
  BOOL MSGCallBackV31Cgo(LONG lCommand, NET_DVR_ALARMER *pAlarmer, char *pAlarmInfo, DWORD dwBufLen, void* pUser)
  {
      return MSGCallBackV31Go(lCommand, pAlarmer, pAlarmInfo, dwBufLen, pUser);
  }
  */
  import "C"
  ```

  > go实现函数声明
  > MSGCallBackV31是函数的全局变量，在导出函数中调用全局变量。
  > 注意export是go导出函数的实现
  > **common.go**

  ```go
  var MSGCallBackV31 MSGCallBackV31Func
  
  //export MSGCallBackV31Go
  func MSGCallBackV31Go(lCommand C.LONG, pAlarmer C.LPNET_DVR_ALARMER, pAlarmInfo *C.char, dwBufLen C.DWORD, pUser unsafe.Pointer) C.BOOL {
      if MSGCallBackV31 != nil {
          buffer := C.GoBytes(unsafe.Pointer(pAlarmInfo), C.int(dwBufLen))
          return C.BOOL(MSGCallBackV31(uint32(lCommand), CgoAlarmer(pAlarmer), buffer, pUser))
      }
      return TRUE
  }
  ```

  > 设置回调函数的实现
  > 真正的回调函数赋值给了全局变量，C的注册函数注册是是cfunctions中的Cgo函数。
  > Cgo函数调用全局变量函数。
  > **common.go**

  ```go
  // SetDVRMessageCallBackV31 注册报警信息回调函数
  func SetDVRMessageCallBackV31(cb MSGCallBackV31Func, pUserData unsafe.Pointer) (err error) {
      MSGCallBackV31 = cb
      ret := C.NET_DVR_SetDVRMessageCallBack_V31(C.MSGCallBack_V31(C.MSGCallBackV31Cgo), pUserData)
      if ret != TRUE {
          err = GetLastError()
          return
      }
      return
  }
  ```

  > go中回调函数的声明
  > **defination.go**

  ```go
  type MSGCallBackV31Func func(lCommand uint32, pAlarmer *Alarmer, pAlarmInfo []byte, pUserData unsafe.Pointer) int32
  ```

  

## C,CGO,GO中类型转换关系

- 类型转换

  | C语言类型              | CGO类型        | Go语言类型     |
  | :--------------------- | :------------- | :------------- |
  | bool                   | C.bool         | bool           |
  | char                   | C.char         | byte           |
  | singed char            | C.schar        | int8           |
  | unsigned char          | C.uchar        | uint8          |
  | short                  | C.short        | int16          |
  | unsigned short         | C.ushort       | uint16         |
  | int                    | C.int          | int32          |
  | unsigned int           | C.uint         | uint32         |
  | long                   | C.long         | int32          |
  | unsigned long          | C.ulong        | uint32         |
  | long long int          | C.longlong     | int64          |
  | unsigned long long int | C.ulonglong    | uint64         |
  | float                  | C.float        | float32        |
  | double                 | C.double       | float64        |
  | size_t                 | C.size_t       | uint           |
  | void*                  | unsafe.Pointer | unsafe.Pointer |

  

## CGO中的结构体和 ```void*``` 转换

> C语言中入参大部分情况下是void*类型的指针，需要转换成go语言相应的类型。
> go语言相应的结构体，也需要转换成C语言的void*指针。

- 实现

  > 有如下C结构体

  ```c
  typedef  unsigned short     WORD;
  typedef  unsigned char      BYTE;
  
  #define ACS_CARD_NO_LEN                 32  //门禁卡号长度
  typedef struct tagNET_DVR_CARD_CFG_SEND_DATA
  {
      DWORD dwSize;
      BYTE byCardNo[ACS_CARD_NO_LEN]; //卡号
      DWORD dwCardUserId;    //持卡人ID
      BYTE byRes[12];
  }NET_DVR_CARD_CFG_SEND_DATA, *LPNET_DVR_CARD_CFG_SEND_DATA;
  ```

  > 定义成如下go结构体
  > byRes是占位符，所以在go中是不关心大小的，所以定义成了切片。
  > **defination.go**

  ```go
  // CardCfgSendData 获取卡参数的发送数据
  type CardCfgSendData struct {
      Size       uint32
      CardNo     string //卡号 ACS_CARD_NO_LEN
      CardUserId uint32 //持卡人ID
      byRes      []byte //size12
  }
  ```

  > 结构体转换函数
  > 注意1: C的宏定义转go的全局变量
  > 注意2: C的char数组转go的string，最后会有一个`0x00`的结束符是go不需要的。
  > 注意2：由于go无法使用类型计算结构体大小，所以计算结构体大小时需要先创建变量。
  > **convert.go**

  ```go
  AcsCardNoLen              = C.ACS_CARD_NO_LEN                // 32  //门禁卡号长度
  
  // GoCCardCfgSendData Go to C
  func GoCCardCfgSendData(in *CardCfgSendData) (out C.LPNET_DVR_CARD_CFG_SEND_DATA) {
      cardNo := [AcsCardNoLen]C.BYTE{}
      for i := 0; i < len(in.CardNo) && i < AcsCardNoLen-1; i++ {
          cardNo[i] = C.BYTE(in.CardNo[i])
      }
  
      var temp C.NET_DVR_CARD_CFG_SEND_DATA
      size := uint32(unsafe.Sizeof(temp)) // 单纯计算size， 无法使用类型计算
      out = &C.NET_DVR_CARD_CFG_SEND_DATA{
          dwSize:       C.DWORD(size),
          byCardNo:     cardNo,
          dwCardUserId: C.DWORD(in.CardUserId),
      }
      return
  }
  ```

  > 万事具备，最后是C结构体指针转C的void指针
  > 使用`unsafe.Pointer`转换为go中无符号指针，再通过`C.LPVOID`进行强转。

  ```go
  lpInBuffer = C.LPVOID(unsafe.Pointer(GoCCardCfgSendData(value)))
  ```

  