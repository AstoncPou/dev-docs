# 极市算法SDK说明文档V2.0（C&C++)

| 版本 |              内容说明               | 编写人 |  更新时间  |
| :--: | :---------------------------------: | :----: | :--------: |
| V2.0 | 更新算法接口及参数,SDK授权，模型加密 | YuanCheng.Huang | 2019/03/27 |
| V1.2 | 更新接口参数说明 | YuanCheng.Huang | 2018/12/05 |
| V1.1 | 更新ji_util_h文件，加入异常处理说明 | Xin.du | 2018/09/29 |
| V1.0 | 提交至极市的算法Linux SDK规范的说明 | Xin.du | 2018/09/11 |


**注意：此规范支持Windows和Linux,但Windows需要修改授权，如需Windows规范工程，请联系工作人员获取**

# 目录
* [结构体说明](#结构体说明)
    * [JI_CV_FRAME](##JI_CV_FRAME)
    * [JI_EVENT](##JI_EVENT)
* [常量值说明](#常量值说明)
    * [函数返回值定义](##函数返回值定义)
    * [检测器类型值定义](##检测器类型值定义)
    * [分析输出code值定义](##分析输出code值定义)
* [函数说明](#函数说明)
    * [ji.h](##ji.h接口说明)
* [JSON说明](#JSON说明)
    * [图像处理类算法Json实例](##图像处理类算法Json实例)
* [模型加解密](#模型加解密)
	* [模型加密](##模型加密)
	* [模型解密](##模型解密)

# 结构体说明
## JI_CV_FRAME
- 单帧信息，参考CV::Mat

## JI_EVENT
- 分析输出信息
    - int code;           //详见"分析输出code值定义"  
    - const char *json;   //算法输出结果,[json格式](#图像处理类算法Json实例),json格式可到[bejson](https://www.bejson.com/)进行验证  

# 常量值说明
## 函数返回值定义

- JISDK_RET_SUCCEED               (0)             //成功
- JISDK_RET_FAILED                (-1)            //失败
- JISDK_RET_UNUSED                (-2)            //未实现
- JISDK_RET_INVALIDPARAMS         (-3)            //参数错误
- JISDK_RET_OVERMAXQPS            (-99)           //超过最大请求量
- JISDK_RET_UNAUTHORIZED          (-999)          //未授权


## 检测器类型值定义

	1.如果算法仅支持不带状态的检测器,传任何参数均创建不带状态的检测器;
	1.如果算法仅支持带状态的检测器,传任何参数均创建带状态的检测器;
	2.如果算法同时支持不带或带状态的检测器时：
	  JISDK_PREDICTOR_DEFAULT:       开发者自行决定创建;
	  JISDK_PREDICTOR_SEQUENTIAL:    创建带状态的检测器;
	  JISDK_PREDICTOR_NONSEQUENTIAL: 创建不带状态的检测器;

- JISDK_PREDICTOR_DEFAULT         (0)             //默认
- JISDK_PREDICTOR_SEQUENTIAL      (1)             //连续的,即带状态的
- JISDK_PREDICTOR_NONSEQUENTIAL   (2)             //非连续的,即不带状态的


## 分析输出code值定义

- JISDK_CODE_ALARM                (0)             //报警
- JISDK_CODE_NORMAL               (1)             //正常
- JISDK_CODE_FAILED               (-1)            //失败

# 函数说明
## ji.h接口说明
*[ji_init](##初始化)  
*[ji_create_predictor](##创建检测器（必选项）)  
*[ji_destroy_predictor](##释放检测器（必选项）)  
*[ji_calc_buffer](##算法检测针对图片缓冲（图片必选项）)  
*[ji_calc_file](##算法检测针对图片文件（图片必选项）)  
*[ji_calc_video_file](##算法检测针对视频文件（视频必选项）)  
*[ji_calc_frame](##算法检测针对一帧（视频必选项）)  

### 初始化（可选项）

	函 数 名	: ji_init
	功能描述	: sdk初始化,可在函数接入license,glog,log4cpp等
	函数参数	: 变参原型,参数有开发者自行定义
	返回参数	: 详见"函数返回值定义",成功返回JISDK_RET_SUCCEED,其它表示失败  
	注     意   : 无
	
	int ji_init(int argc, char** argv);  
    


### 创建检测器（必选项）

	函 数 名	: ji_create_predictor
	功能描述	: 创建检测器,即每个检测实例的上下文
	函数参数	: 
	    pdtype: [必选参数] 输入参数,检测器实例类型,详见"检测器类型值定义"
	返回参数	: 成功返回检测器实例指针,错误返回NULL  
	注     意: 无
	
	void *ji_create_predictor(int pdtype);



### 释放检测器（必选项）

	函 数 名    : ji_destroy_predictor
	功能描述    : 释放检测器实例
	函数参数    : 检测器实例
	    predictor: [必选参数] 输入参数,检测器实例
	返回参数    : 无  
	注     意: 空指针异常判断以免崩溃（请参考ji.cpp）
	
	void ji_destroy_predictor(void *predictor);



### 算法检测针对图片缓冲（图片必选项）

	函 数 名    : ji_calc_buffer
	功能描述    : 算法检测针对图片缓冲
	函数参数    : 
	    predictor: [必选参数] 输入参数,不允许为NULL,检测器实例;有ji_create_predictor函数生成.
	    buffer:    [必选参数] 输入参数,不允许为NULL,源图片缓冲地址.(请参考sdk 2.0 test.cpp)
	    length:    [必选参数] 输入参数,必须大于0,源图片缓冲大小.
	    args:      [必选参数] 输入参数,允许为NULL,检测参数;由开发者自行定义，例如:roi.(请参考sdk 2.0 3rd\boost_interface)
	    outfile:   [必选参数] 输入参数,允许为NULL,输出结果图片文件路径.
	    event:     [必选参数] 输出参数,允许为NULL,输出结果;event->json有开发者创建和释放.
	返回参数    : 详见"函数返回值定义",成功返回JISDK_RET_SUCCEED,其它表示失败 
	注     意   : 参数异常判断以免崩溃（请参考ji.cpp）
	
	int ji_calc_buffer(void *predictor, const void *buffer, int length,
	                   const char *args, const char *outfile, JI_EVENT *event);

### 算法检测针对图片文件（图片必选项）

	函 数 名    : ji_calc_file
	功能描述    : 算法检测针对图片文件
	函数参数    : 
	   predictor: [必选参数] 输入参数,不允许为NULL,检测器实例;有ji_create_predictor函数生成.
	   infile:    [必选参数] 输入参数,不允许为NULL,源图片文件路径.
	   args:      [必选参数] 输入参数,允许为NULL,检测参数;有开发者自行定义，例如:roi.(请参考sdk 2.0 3rd\boost_interface)
	   outfile:   [必选参数] 输入参数,允许为NULL,输出结果图片文件路径.
	   event:     [必选参数] 输出参数,允许为NULL,输出结果;event->json有开发者创建和释放.
	返回参数    : 详见"函数返回值定义",成功返回JISDK_RET_SUCCEED,其它表示失败 
	注     意   : 参数异常判断以免崩溃（请参考ji.cpp）
	
	int ji_calc_file(void *predictor, const char *infile, const char *args,
	                 const char *outfile, JI_EVENT *event);

### 算法检测针对视频文件（视频必选项）

	函 数 名    : ji_calc_video_file
	功能描述    : 算法检测针对视频文件
	函数参数    : 
	    predictor: [必选参数] 输入参数,不允许为NULL,检测器实例;有ji_create_predictor函数生成
	    infile:    [必选参数] 输入参数,不允许为NULL,源视频文件路径.
	    args:      [必选参数] 输入参数,允许为NULL,检测参数;有开发者自行定义，例如:roi.(请参考sdk 2.0 3rd\boost_interface)
	    outfile:   [必选参数] 输入参数,允许为NULL,输出结果视频文件路径.
	    jsonfile:  [必选参数] 输入参数,允许为NULL,输出结果json文件路径.
	返回参数    : 详见"函数返回值定义",成功返回JISDK_RET_SUCCEED,其它表示失败
	注     意   : 参数异常判断以免崩溃（请参考ji.cpp）

	int ji_calc_video_file(void *predictor, const char *infile, const char* args,
                       const char *outfile, const char *jsonfile);

### 算法检测针对一帧（视频必选项）
 
	函 数 名    : ji_calc_frame  
	功能描述    : 算法检测针对一帧  
	函数参数    :   
	    predictor: [必选参数] 输入参数,不允许为NULL,检测器实例;有ji_create_predictor函数生成.  
	    inframe:   [必选参数] 输入参数,不允许为NULL,输入源帧.  
	    args:      [必选参数] 输入参数,允许为NULL,检测参数;有开发者自行定义，例如:roi.(请参考sdk 2.0 3rd\boost_interface)  
	    outframe:  [必选参数] 输出参数,允许为NULL,输出结果帧;outframe->data有开发者创建和释放.  
	    event:     [必选参数] 输出参数,允许为NULL,输出结果;event->json有开发者创建和释放.  
	返回参数    : 详见"函数返回值定义",成功返回JISDK_RET_SUCCEED,其它表示失败  
	注     意   : 参数异常判断以免崩溃（请参考ji.cpp）

	int ji_calc_frame(void *predictor, const JI_CV_FRAME *inframe, const char *args,
                  JI_CV_FRAME *outframe, JI_EVENT *event);

# JSON说明
## 图像处理类算法Json实例


	{  
		"info": {  
		"numOfStyle": "",
		"style": [{
			"id": "",
			"top": "1",
			"prob": ""
		},
		{
			"id": "",
			"top": "2",
			"prob": ""
		},
		{
			"id": "",
			"top": "3",
			"prob": ""
		}
	]
	},
	"image_path": "#图片保存路径 ~/dest/UUID.jpg #以/dest/为图片文件夹存储相对路径;以C++ UUID库产生随机数,作为图片命名名称;以.jpg为图片后缀#",
	"status": "#分类图片,分类成功0 分类异常-1#",
	"message": "如果status = 0, 输出#Imagefile classify successed# 如果status = -1, 输出#Imagefile classify failed#"
	} 

# 模型加解密

## 模型加密

加密可执行文件

	encrypt_module

使用后会在***本目录***生成一个 model_str.hpp头文件，请添加到ji.cpp内（可以挪动位置）。  
头文件内容为两个字符串，一个为加密后的，一个为混淆密钥，解密用到的就是这两个字符串。

Demo

	./encrypt_module number_detection_inference.cfg 11111111111111111111111111111111  
	||
	./encrypt_module number_detection_inference.cfg
	可以不用输32位密钥（有默认密钥）。如需要请必须添加32位。

## 模型解密

	函 数 名    : CreateEncryptor  
	功能描述    : 进行混淆密钥解密  
	函数参数    :   
		str         :   [必选参数] 输入参数,不允许为NULL,加密后的字符串.  
    	str_len     :   [必选参数] 输入参数,不允许为NULL,加密后字符串长度.  
		key         :   [必选参数] 输入参数,不允许为NULL,密钥.  
	返回参数    : 返回一个解密实例   


void* CreateEncryptor(const char* str,int str_len,const char* key);

***

	函 数 名    : FetchBuffer  
	功能描述    : 解密字符串  
	函数参数    :   
		ptr            :   [必选参数] 输入参数,不允许为NULL,解密器返回后的实例（CreateEncryptor 函数返回）.  
    	buffer_len     :   [必选参数] 输入参数,不允许为NULL,返回字符串的长度.  
	返回参数    : 返回解密后的字符串   


void* FetchBuffer(void* ptr,int &buffer_len);

***

Demo样例

	void* FetchBuffer(void* ptr,int &buffer_len);
	void* file = CreateEncryptor(model_str.c_str(), model_str.size(), key.c_str());
        int fileLen = 0;
        char *fileContent = (char*)FetchBuffer(file, fileLen);
        std::cout << "FetchBuffer:" << fileContent << std::endl;



