## 音频质量检测服务

### 环境配置、工具安装

- 使用```git clone``` 下载项目

  ```shell
  git clone git@git.speakin.mobi:capacity/waveform-checker-service.git
  ```



- 切换到 ```abnormaltone``` 分支



- git 配置，下载安装私有仓库的依赖

  ```shell
  git config --global url."git@git.speakin.mobi:".insteadOf "http://git.speakin.mobi/"
  ```



- 安装swag，并且配置到环境变量中

  ```shell
  go get -u github.com/swaggo/swag/cmd/swag
  
  # 编辑.bashrc文件
  vim ~/.bashrc
  # 在最后一行添加 
  export PATH=/home/wan/go/bin:$PATH
  # 添加完成之后执行
  source ~/.bashrc
  ```



- 生成docs文件夹，将docs文件夹拖到pkg文件夹中

  ```shell
  swag init
  ```

  

- 下载项目依赖

  ```shell
  go mod tidy
  ```



### 下载头文件、动态链接库和模型

- 执行shell脚本下载

  ```shell
  ./hack/3rd.sh ready library_gpu
  ./hack/3rd.sh ready model_gpu
  ./hack/3rd.sh ready library_abnormal_tone
  ./hack/3rd.sh ready model_phone_align
  ```



- 部分动态库的缺失，可以通过软连接的方式创建

  ```shell
  cd 3rd/lib/waveform-checker
  
  ln -s libgfortran.so.3 libgfortran.so
  ln -s libwaveform_checker.so.1.2.20 libwaveform_checker.so
  ```



- 配置文件路径有误，需要创建一个软连接

  ```shell
  ln -sfn 3rd/models/phone_align/ conf
  ```

  



### 编译打包

- 使用Makefile

  ```shell
  make clean && make
  ```



### 服务模块

#### 初始化

- 输出版本信息

  > 版本信息在编译过程中注入到可执行性文件中，
  >
  > ```-X 'main.Version=v1.0.7-12-g649ab52-dirty' ```
  >
  > ```-X 'main.CommitHash=649ab5226862ba28ce1fcc645c7741dc1d3bde32'```
  >
  > ``` -X 'main.BuildTime=20210922171838'```

  ```shell
  CGO_CFLAGS="-g -O2 -I /home/wanmaoyuan/gocode/src/waveform-checker-service/3rd/include" CGO_LDFLAGS="-g -O2 -L /home/wanmaoyuan/gocode/src/waveform-checker-service/3rd/lib/waveform-checker -Wl,-rpath=.:./lib:./3rd/lib/waveform-checker" go build -ldflags "-s -w -X 'main.Version=v1.0.7-12-g649ab52-dirty' -X 'main.CommitHash=649ab5226862ba28ce1fcc645c7741dc1d3bde32' -X 'main.BuildTime=20210922171838'" -o dist/waveform-checker-service/waveform-checker-service 
  ```

  

- 读取配置文件

  > 先生成好配置文件的结构体

  ```go
  func Initialize() {
  	once.Do(func() {
  		v := viper.New()
  		v.SetConfigName("service")
  		v.AddConfigPath(".")
  		v.SetConfigType("yaml")
  		if err := v.ReadInConfig(); err != nil {
  			log.Fatal("initialize config", zap.Error(err))
  		}
  		if err := v.Unmarshal(&C); err != nil {
  			log.Fatal("unmarshal config", zap.Error(err))
  		}
  		log.Info("configuration initialized")
  	})
  }
  ```



- 算法引擎初始化

  > 1、波形检测引擎初始化
  >
  > ​       初始化需要的参数：				
  >
  > ​			engine:
  >
  > ​				waveform-checker:
  >
  > ​					threads: 1
  >
  > ​					sample-frequency: 8000
  >
  > ​					threshold_conf: ./3rd/models/waveform-checker/waveform_checker_options.conf
  >
  > ​					fbank_feat_conf: ./3rd/models/waveform-checker/feature_fbank_60dim.conf
  >
  > ​					fbank_complates_freq_conf: ./3rd/models/waveform-checker/fbank_complates_freq.conf
  >
  > ​					snr_feature_conf: ./3rd/models/waveform-checker/snr_feature.conf
  
  ```go
  opts := checker.Option{
  		Threads:                      config.C.Engine.WaveformChecker.Threads,
  		SampleFrequency:              config.C.Engine.WaveformChecker.SampleFrequency,
  		WaveformCheckerThresholdConf: config.C.Engine.WaveformChecker.ThresholdConf,
  		FbankFeatConf:                config.C.Engine.WaveformChecker.FbankFeatConf,
  		FbankComplatesFreqConf:       config.C.Engine.WaveformChecker.FbankComplatesFreqConf,
  		SNRFeatureConf:               config.C.Engine.WaveformChecker.SNRFeatureConf,
  }
  WaveformCheckHandler, err = checker.NewHandler(opts)
  ```
  
  
  
  > 2、异常音频检测引擎初始化
  >
  > ​	初始化需要的参数
  >
  > ​		engine: 
  >
  >  			abnormal_tone:
  >
  >   				thread_number: 1
  >
  >   				asr_conf: "phone_align/conf/asr.conf"
  >
  >   				phone_align_conf: "phone_align/conf/phone_align.conf"
  >
  >  				 abnormal_tone_conf: "conf/abnormal_tone.conf"
  >
  >   				fswitch: 
  >
  >    					repeat_number: true
  >
  >    					repeat_sentence: true
  >
  >    					noid: true
  >
  >    					accelerate: true
  >
  >    					undulate: true
  
  ```go
  // initialize abnormal tone engine
  abnormaltoneOption := abnormaltone.Option{
      ThreadNumber:     config.C.Engine.AbnormalTone.ThreadNumber,
      ASRConf:          config.C.Engine.AbnormalTone.ASRConf,
      PhoneAlignConf:   config.C.Engine.AbnormalTone.PhoneAlignConf,
      AbnormalToneConf: config.C.Engine.AbnormalTone.AbnormalToneConf,
      FSwitch: &abnormaltone.AbnormalToneFuncSwitch{
          RepeatNumber:   config.C.Engine.AbnormalTone.FSwitch.RepeatNumber,
          RepeatSentence: config.C.Engine.AbnormalTone.FSwitch.RepeatSentence,
          NoID:           config.C.Engine.AbnormalTone.FSwitch.NoID,
          Accelerate:     config.C.Engine.AbnormalTone.FSwitch.Accelerate,
          Undulate:       config.C.Engine.AbnormalTone.FSwitch.Undulate,
      },
  }
  AbnormalToneChecker, err = abnormaltone.CreateAbnormalToneChecker(abnormaltoneOption)
  ```



- http服务初始化

  > 服务接口及其功能

  ```go
  v1 := G.Group("/v1")
  
  v1.Handle("GET", "/healz", healz.Healz)
  v1.Handle("POST", "/waveform-check", waveform.Check)
  v1.Handle("POST", "/waveform-check/avg_energy", waveform.AvgEnergy)
  v1.Handle("POST", "/waveform-check/deviation", waveform.Deviation)
  v1.Handle("POST", "/waveform-check/snr", waveform.SNR)
  v1.Handle("POST", "/waveform-check/trunc_amp", waveform.TruncAMP)
  v1.Handle("POST", "/waveform-check/valid_speech", waveform.ValidSpeech)
  v1.Handle("POST", "/waveform-check/waveform", waveform.Waveform)
  v1.Handle("POST", "/waveform-check/complates_freq_mark", waveform.ComplatesFreqMark)
  v1.Handle("POST", "/waveform-check/loudness", waveform.Loudness)
  ```
  
  - ```/v1/healz``` 引擎信息API
  
  - ```/v1/waveform-check``` 音频质量检测API
  - ```/v1/waveform-check/avg_energy``` 平均能量API
  - ```/v1/waveform-check/complates_freq_mark``` 音频完整性评分API
  - ```/v1/waveform-check/deviation``` 波形偏移检测API
  - ```/v1/waveform-check/loudness``` 响度检测API
  - ```/v1/waveform-check/snr``` 信噪比检测API
  - ```/v1/waveform-check/trunc_amp``` 截幅检测API
  - ```/v1/waveform-check/valid_speech``` 有效音时长检测API
  - ```/v1/waveform-check/waveform``` 波形大小检测API
  
  

#### HTTP请求处理

- ```/v1/healz``` 引擎信息API

  > GET请求
  >
  > 返回 波形检测引擎的信息（名字、版本、描述）

- ```/v1/waveform-check``` 音频质量检测API

  > abnormaltone 和 waveform 都是可选
  >
  > 选择性的进行波形检测和异常音频检测
  >
  > ​	1、检测内容判断
  >
  > ​	2、表单文件读取
  >
  > ​	3、判断表单文件个数是否为0
  >
  > ​	4、根据请求头判断文件类型
  >
  > ​	5、检测文件是否能够打开
  >
  > ​	6、读取文件字节
  >
  > ​	7、将字节传入引擎，根据选择进行波形检测和异常音频检测

- ```/v1/waveform-check/avg_energy``` 平均能量API

  > 1、表单文件读取
  >
  > 2、判断表单文件个数是否为0
  >
  > 3、根据请求头判断文件类型
  >
  > 4、检测文件是否能够打开
  >
  > 5、读取文件字节
  >
  > 6、将字节传入引擎，进行平均能量检测

- ```/v1/waveform-check/complates_freq_mark``` 音频完整性评分API

  > 1、表单文件读取
  >
  > 2、判断表单文件个数是否为0
  >
  > 3、根据请求头判断文件类型
  >
  > 4、检测文件是否能够打开
  >
  > 5、读取文件字节
  >
  > 6、将字节传入引擎，进行音频完整性检测

- ```/v1/waveform-check/deviation``` 波形偏移检测API

  > 1、表单文件读取
  >
  > 2、判断表单文件个数是否为0
  >
  > 3、根据请求头判断文件类型
  >
  > 4、检测文件是否能够打开
  >
  > 5、读取文件字节
  >
  > 6、将字节传入引擎，进行波形偏移检测

- ```/v1/waveform-check/loudness``` 响度检测API

  > 1、表单文件读取
  >
  > 2、判断表单文件个数是否为0
  >
  > 3、根据请求头判断文件类型
  >
  > 4、检测文件是否能够打开
  >
  > 5、读取文件字节
  >
  > 6、将字节传入引擎，进行音频响度检测

- ```/v1/waveform-check/snr``` 信噪比检测API

  > 1、表单文件读取
  >
  > 2、判断表单文件个数是否为0
  >
  > 3、根据请求头判断文件类型
  >
  > 4、检测文件是否能够打开
  >
  > 5、读取文件字节
  >
  > 6、将字节传入引擎，进行信噪比检测

- ```/v1/waveform-check/trunc_amp``` 截幅检测API

  > 1、表单文件读取
  >
  > 2、判断表单文件个数是否为0
  >
  > 3、根据请求头判断文件类型
  >
  > 4、检测文件是否能够打开
  >
  > 5、读取文件字节
  >
  > 6、将字节传入引擎，进行截幅检测

- ```/v1/waveform-check/valid_speech``` 有效音时长检测API

  > 1、表单文件读取
  >
  > 2、判断表单文件个数是否为0
  >
  > 3、根据请求头判断文件类型
  >
  > 4、检测文件是否能够打开
  >
  > 5、读取文件字节
  >
  > 6、将字节传入引擎，进行有效音时长检测

- ```/v1/waveform-check/waveform``` 波形大小检测API

  > 1、表单文件读取
  >
  > 2、判断表单文件个数是否为0
  >
  > 3、根据请求头判断文件类型
  >
  > 4、检测文件是否能够打开
  >
  > 5、读取文件字节
  >
  > 6、将字节传入引擎，进行波形大小检测
