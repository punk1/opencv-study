# opencv入门
  
  
## 一 引用
  
  
~~~命名空间与头文件
opencv 头文件：#include <opencv2/opencv.hpp>
  
opencv 中的函数 都是定义在 cv 空间下的 using namespace cv;
  
~~~
  
## 二 图像的载入和显示
  
~~~
    Mat src = imread("D:/lena.jpg", 1);
    cvtColor(src, src, COLOR_BGR2GRAY);
    namedWindow("src", WINDOW_AUTOSIZE);
    imshow("src", src);
~~~
  
  
  
1:Mat 类  用于存放保存图像以及其他矩阵数据的数据结构
~~~
  
Mat src = imread("D:/lena.jpg", 1);
~~~
  
  
2:imread 函数 （ imread函数从文件中加载图像并返回该图像。）
~~~
 函数原型
Mat imread(const string& filename, intflags=1 );
  
~~~
  
  
-  第一个参数，const string&类型的filename，填我们需要载入的图片路径名
  
- 第二个参数，int类型的flags，为载入标识，它指定一个加载图像的颜色类型。(默认为 1)
  + flags >0返回一个3通道的彩色图像。
  + flags =0返回灰度图像。
  + flags <0返回包含Alpha通道的加载的图像。
  
3:namedWindow 函数 （namedWindow函数，用于创建一个窗口。）
函数原型
~~~
void namedWindow(const string& winname,int flags=WINDOW_AUTOSIZE ); 
~~~
-  第一个参数，const string&型的name，即填被用作窗口的标识符的窗口名称。
- 第二个参数，int 类型的flags ，窗口的标识，可以填如下的值：
  + WINDOW_NORMAL设置了这个值，用户便可以改变窗口的大小（没有限制）
  + WINDOW_AUTOSIZE如果设置了这个值，窗口大小会自动调整以适应所显示的图像，并且不能手动改变窗口大小。
  + WINDOW_OPENGL 如果设置了这个值的话，窗口创建的时候便会支持OpenGL。0
  
  
4:imshow (imshow 函数用于在指定的窗口中显示图像。）
~~~
函数原型
void imshow(const string& winname, InputArray mat);
~~~
  
- 第一个参数，const string&类型的winname，填需要显示的窗口标识名称。
  
- 第二个参数，InputArray 类型的mat，填需要显示的图像。
  
5:cvtcolor()   (函数是一个颜色空间转换函数，可以实现RGB颜色向HSV，HSI等颜色空间转换。也可以转换为灰度图。)
~~~
函数原型
void cvtColor(InputArray src, OutputArray dst, int code, int dstCn=0 );
  
vtColor(src, src, COLOR_BGR2GRAY);转为灰度图
~~~
  
### 三 图像处理
  
  
1: 图像的频率域是图像像元的灰度值随位置变化的空间频率，以频谱表示信息分布特征，傅立叶变换能把遥感图像从空间域变换到只包含不同频率信息的频率域，原图像上的灰度突变部位、图像结构复杂的区域、图像细节及干扰噪声等信息集中在高频区，，而原图像上灰度变化平缓部位的信息集中在低频区。
2:高斯滤波
  
  
- 求卷积模板（这里求一个  5X5）
  
   -  图像处理用的高斯函数为二维高斯函数 ，公式 ![二维高斯函数公式](https://img-blog.csdn.net/20160824221516652 )
  
      高斯滤波中 x0 ，y0  为0
   - 将卷积模板归一化 
     将卷积中各个值加起来，记为sum ，卷积中 各个值乘以 1/sum 即位卷积模板
  
  
```
        Mat model = Mat(5, 5, CV_64FC1);
  
	double sigma = 80;
	for (int i = -2; i <= 2; i++)
	{
		for (int j = -2; j <= 2; j++)
		{
			model.at<double>(i + 2, j + 2) =
				exp(-(i * i + j * j) / (2 * sigma * sigma)) /
				(2 * PI * sigma * sigma);
		}
	}
  
```
```
        double gaussSum = 0;
	gaussSum = sum(model).val[0];
	for (int i = 0; i < model.rows; i++)
	{
		for (int j = 0; j < 5; j++)
		{
			model.at<double>(i, j) = model.at<double>(i, j) /
				gaussSum;
		}
	}
```
3:进行卷积
  
```
Mat dst = Mat(src.rows - 4, src.cols - 4, CV_8UC1);// dst 为输出图像，卷积操作后原图将丢失上下两行 与 左右 两列
  
	for (int i = 2; i < src.rows - 2; i++)//卷积窗口平滑次数
	{
		for (int j = 2; j < src.cols - 2; j++)//卷积窗口纵向平滑次数
		{
			double sum = 0;
			for (int m = 0; m < model.rows; m++)//进行卷积操作
			{
				for (int n = 0; n < model.cols; n++)
				{
					sum += (double)src.at<uchar>(i- 2+m, j - 2 + n) *
						model.at<double>(m, n);
				}
			}
  
			dst.at<uchar>(i - 2, j - 2) = (uchar)sum;//模糊后的该点的值
  
		}
	}
  
```
4:应用
  
 GaussianBlur函数的作用是用高斯滤波器来模糊一张图片，对输入的图像src进行高斯滤波后用dst输出。它将源图像和指定的高斯核函数做卷积运算。
```
 C++: void GaussianBlur(InputArray src,OutputArray dst, Size ksize, double sigmaX, double sigmaY=0, intborderType=BORDER_DEFAULT )
```
- 第三个参数，Size类型的ksize高斯内核的大小。其中ksize.width和ksize.height可以不同，但他们都必须为正数和奇数。或者，它们可以是零的，它们都是由sigma计算而来
- 第四个参数，double类型的sigmaX，表示高斯核函数在X方向的的标准偏差。
- 第五个参数，double类型的sigmaY，表示高斯核函数在Y方向的的标准偏差。若sigmaY为零，就将它设为sigmaX，如果sigmaX和sigmaY都是0，那么就由ksize.width和ksize.height计算出来。
  
```
       //载入原图
       Mat image=imread("D:/lena.jpg");
       //进行滤波操作
       Mat out;
       GaussianBlur( image, out, Size( 5, 5 ), 0, 0 ); 
```
  
  
  
  
  
  
  
  
  
  