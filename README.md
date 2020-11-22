# 基于计算机视觉和PYQT5的批量抠图软件
## 方案设计
###    一 抠图软件整体架构  
**设计思路:**  
1、确定原图路径，可以实现批量的P图  
2、确定导出路径，P好的图会被保存在这个路径下  
3、点击抠图按钮，启动P图流程  
4、程序会读取原图路径下的所有图片文件，交给人工智能的API来处理，完成抠像工作(实现对人像和背景的切割)  
5、返回抠图图片（带有透明图层）  
6、将带有透明图层的图片与白色或者蓝色的底片进行融合  
7、得到白底或蓝底的扣出人像的图片  
8、在使用Python人脸识别库，确定精确的人脸范围，对图像进行裁剪         

--源代码有七个主要函数，koutu函数是放在PYQT5框架下运行的函数，koutu函数按要求调用其余函数，实现对应功能。PyQt5 是用来创建Python GUI应用程序的工具包。作为一个跨平台的工具包，PyQt可以在所有主流操作系统上运行，PyQt5是基于Digia公司强大的图形程式框架Qt5的python接口，由一组python模块构成。PyQt5本身拥有超过620个类和6000函数及方法。在可以运行于多个平台，包括：Unix, Windows, and Mac OS。此布局有五个选项，给选择源文件夹，选择文件保存目录和抠图设置信号后自定义点击事件函数，自定义点击事件函数slot_btn_koutu中调用koutu函数，选择白底和蓝底为实例化的两个按钮，设置按钮点点击状态且绑定点击事件。os.getcwd函数获取当前程序文件位置，是实现选择文件夹的主要函数。

*七个主要函数*  
```def GetWhiteScreen(width, height, wpath):  
def GetBlueScreen(width, height, wpath):   
def Getface(source, output):  
def BlendImg(fore_image, base_image, output_path):  
def get_access_token():  
def get_foreground(originalImagePath, outputpath):  
def koutu():  
```
--先用get_foreground函数调用百度api使用其训练的模型实现人像分割，识别人体的轮廓范围，与背景进行分离，适用于拍照背景替换、照片合成、身体特效等场景；输入正常人像图片，返回分割后的二值结果图、灰度图、透明背景的人像图,返回的图像是透明图层，根据所选背景颜色是白色还是蓝色，GetWhiteScreen函数得到白底图片，GetBlueScreen函数得到蓝底图片，BlendImg函数将用get_foreground函数人物分割得到的图像与底色图片加权成一个新图片。因人像分割后不能得到具体的面部照片，Getface函数调用face_recognition库是世界上最简洁的人脸识别库，可以使用Python和命令行工具提取、识别、操作人脸。可得到具体的面部图片，保存并输出到目标文件夹。  

*kousu函数调用流程*  
```
if wob:  
  print("white")  
  GetWhiteScreen(img.shape[0], img.shape[1], path + "/" + "白底" + item)  
  BlendImg(path + "/" + item, path + "/" + "白底" + item, path + "/" + "合成" + item)       
  Getface(path + "/" + "合成" + item, path + "/" + "最终" + item)                        
  os.remove(path + "/" + "白底" + item)  
else:  
  print("blue")  
  GetBlueScreen(img.shape[0], img.shape[1], path + "/" + "蓝底" + item)  
  BlendImg(path + "/" + item, path + "/" + "蓝底" + item, path + "/" + "合成" + item)  
  Getface(path + "/" + "合成" + item, path + "/" + "最终" + item)  
  os.remove(path + "/" + "蓝底" + item)  
```
 
### 二 关键技术难点及解决方案
**技术难点分析:**    
1、需要通过软件界面来做文件选择和按钮。  
2、使用何种简单的人工智能方法(最好是免费的)。  
**解决方案:**  
1、使用PYQT5的Python图形界面库，解决软件界面的搭建。  
2、调用百度Al的API(应用程序接口)，通过网络调用人工智能接口给出源数据，返回所需数据。来实现Al赋能。  

*核心代码*  
```
def koutu():
    global wob
    path = dir_choose    # 图片路径

    # 调用百度api实现人像分割，返回的图像是透明图层
    if os.path.exists(path):     # 判断路径是否存在
        files = os.listdir(path)
        for item in files:
            get_foreground(path + "/" + item, str(fileName_choose) + "/" + item)        # 批量抠图
        print("---- 程序结束 ----")
    else:
        print("输入的路径不存在！！！")
        print("---- 程序结束 ----")

    path = fileName_choose

    # 合成
    if os.path.exists(path):        # 判断路径是否存在
        files = os.listdir(path)
        for item in files:
            img = cv2.imdecode(np.fromfile(path + "/" + item, dtype=np.uint8), cv2.IMREAD_UNCHANGED)
            print(item)
            # 第三步：生成绿幕并合成
            if wob:
                print("white")
                GetWhiteScreen(img.shape[0], img.shape[1], path + "/" + "白底" + item)                 # 白底
                BlendImg(path + "/" + item, path + "/" + "白底" + item, path + "/" + "合成" + item)     # 原透明图层人像+白底
                Getface(path + "/" + "合成" + item, path + "/" + "最终" + item)                         # 白底人像+face_rerecognition得到白底大头照
                os.remove(path + "/" + "白底" + item)
            else:
                print("blue")
                GetBlueScreen(img.shape[0], img.shape[1], path + "/" + "蓝底" + item)
                BlendImg(path + "/" + item, path + "/" + "蓝底" + item, path + "/" + "合成" + item)
                Getface(path + "/" + "合成" + item, path + "/" + "最终" + item)
                os.remove(path + "/" + "蓝底" + item)

    print("完成!!!")
```

**总结：**   
1、最初使用Paddle(百度飞桨机器学习平台)，用谷歌训练好的模型在本地做人像切割(抠图)(可以无需联网），但是无法识别中文文件，所以后来改用了百度Al来实现，还需用CV2的  decode方法，不要直接imread读取(来解决读取中文文件)。  
2、识别脸部的face recogition库，(是离线的模型，不用联网），但是只识别脸。头发、耳朵、脖子都不管。只会给你个脸部范围:用4个参数x1、x2、y1、y2组成的矩形来确定。  解决方案是:以基本参数扩大一点(应该有更好的数学模型) 

*脸部范围代码 *  
`# 以基本参数扩大一点,获取头像区域范围  
 imagen = image[top - top // 2 - top // 3:bottom + bottom // 4, left - left // 2:right + right // 5]  
`
 
