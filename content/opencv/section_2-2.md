---
utid: 1000-02-02
chapter: 02
chaptername: فصل دوم - حوزهٔ مکان
part: 02
title: ترکیب خطی دو تصویر
---

دو تصویر را می‌توان با استفاده از یک ترکیب خطی از مقادیر پیکسل‌های آنان، جمع کرد. فرم کلی یک ترکیب خطی به صورت زیر است:

$$g\left( x \right) = \alpha f_{0}\left( x \right) + \beta f_{1}\left( x \right) + \gamma\ : \alpha, \ \beta, \ \gamma \in R$$

قبلاً با عملگرهای ریاضی مُجازی که می‌توانستیم روی کلاس Mat استفاده کنیم آشنا شدیم. می دانیم که می‌توان به راحتی دو عکس (با ابعاد یکسان) را با هم جمع کرد و عکس دیگری با همان ابعاد به وجود آورد. اما در این بخش می‌خواهیم شما را با تابع addWeighted آشنا کنیم. این تابع دقیقاً مشابه معادله بالا عمل می‌کند؛ یعنی با دریافت دو ماتریس مبدأ و سه عدد $\alpha$، $\beta$ و $\gamma$ یک ترکیب خطی از دو ماتریس ورودی درست می‌کند و به عنوان خروجی بر می‌گرداند.



## کد

```c++
#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>
#include <iostream>

int main( int argc, char** argv )
{
    double alpha = 0.5; double beta; double input;
    cv::Mat src1, src2, dst;

    /// Ask the user enter alpha
    std::cout << " Simple Linear Blender "<<std::endl;
    std::cout << "-----------------------"<<std::endl;
    std::cout << "* Enter alpha [0-1]: ";
    std::cin >> input;

    // We use the alpha provided by the user iff it is between 0 and 1
    if( alpha >= 0 && alpha <= 1 )
        alpha = input;

    // Read image ( same size, same type )
    src1 = cv::imread("PICTURES/1.jpg");
    src2 = cv::imread("PICTURES/2.jpg");
    if( !src1.data )
    {
        std::cout << "Error loading src1" << std::endl;
        return -1;
    }
    if( !src2.data )
    {
        std::cout << "Error loading src2" << std::endl;
        return -1;
    }

    // Create Windows
    beta = ( 1.0 - alpha );
    cv::addWeighted( src1, alpha, src2, beta, 0.0, dst);

    cv::imshow( "Linear Blend", dst );
    cv::waitKey(0);

    return 0;
}
```



## توضیح

در این برنامه ابتدا از کاربر می‌خواهیم که یک مقدار بین 0 و 1 برای $\alpha$ انتخاب کند. اگر کاربر عدد درستی را وارد کرده باشد، متغیر alpha را برابر با آن عدد قرار می‌دهیم (خط 18)؛ در غیر این صورت از همان مقدار پیشفرض که 0.5 است، استفاده می‌شود.

در ادامه دو تصویر را از حافظه می‌خوانیم و در src1 و src2 قرار می‌دهیم. همچنین مطمئن می‌شویم که این دو تصویر به درستی بارگذاری شده‌اند.

در خط 35 مقدار beta را برابر `1 – alpha` قرار می‌دهیم. این یعنی می‌خواهیم تصویر نهایی به نسبت alpha از تصویر src1 و به نسبت `1 – alpha` از تصویر src2 تشکیل شده باشد.

برای اعمال ترکیب خطی در خط 36 تابع addWeighted را صدا می‌زنیم. توجه کنید که در اینجا مقدار $\gamma$ را صفر قرار داده‌ایم.



## خروجی

برنامه را به صورت زیر اجرا می‌کنیم:

```powershell
LinearBlend.exe
> 0.3
```

خروجی به صورت زیر است:

| ![تصویر ورودی اول](/opencv-book/media/image15.jpeg) | ![تصویر ورودی دوم](/opencv-book/media/image16.jpeg) |
| :-------------------------------------------------: | :-------------------------------------------------: |
|                   تصویر ورودی اول                   |                   تصویر ورودی دوم                   |


| ![تصویر خروجی](/opencv-book/media/image17.jpeg) |
| :---------------------------------------------: |
|                   تصویر خروجی                   |

