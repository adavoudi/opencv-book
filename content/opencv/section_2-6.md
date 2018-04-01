---
utid: 1000-02-06
chapter: 02
chaptername: فصل دوم - حوزهٔ مکان
part: 06
title: تغییر کنتراست و روشنایی تصویر
_index: contrast-brightness
---

به طور کلی کنتراست یکی از مشخصات تصویر است که به صورت نسبت روشنایی درخشان‌ترین رنگ (سفید) به روشنایی تاریک‌ترین رنگ (سیاه) موجود در آن تصویر تعریف می‌شود. در تصاویر هر چه نسبت کنتراست بالاتر باشد مطلوب‌تر است. این مقدار نسبت بیشترین میزان روشنایی رنگ سفید به کمترین میزان روشنایی رنگ سیاه کامل را نشان می‌دهد.

از آنجا که کنتراست و شدت روشنایی هر دو از مشخصه‌های پیکسلی هستند، می‌خواهیم در این بخش به بهانه تغییر کنتراست و روشنایی تصاویر، شما را با تبدیل‌های پیکسلی آشنا کنیم.

به طور کلی اگر $f\left( i,j \right)$ مقدار تصویر ورودی در پیکسل $\left( i,j \right)$ باشد، آنگاه می‌توان با استفاده از معادله زیر و تعیین مقادیر مناسب برای $\alpha$ و $\beta$، کنتراست و روشنایی تصویر $f$ را تغییر داد:

$$g\left( i,j \right) = \alpha\ .\ f\left( i,j \right) + \ \beta$$

در اینجا $g\left( i,j \right)$ مقدار جدید تصویر در پیکسل $\left( i,j \right)$ است. با استفاده از $\alpha$ می‌توان میزان کنتراست را تغییر داد و با استفاده از $\beta$ هم می‌توان میزان روشنایی را تغییر داد. توجه کنید که اگر می‌خواهید فقط میزان روشنایی را تغییر دهید، باید $\alpha$ را یک بگذارید و عدد $\beta$ را فقط تغییر دهید. اگر هم می‌خواهید فقط میزان کنتراست را تغییر دهید کافی است عدد $\beta$ را صفر بگذارید و عدد $\alpha$ را هر آنچه می‌خواهید قرار دهید.



## کد

```c++
#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>
#include <iostream>

int main( int argc, char** argv )
{
    double alpha; /**< Simple contrast control */
    int beta; /**< Simple brightness control */

    /// Read image given by user
    cv::Mat image = cv::imread( argv[1] );
    cv::Mat new_image = cv::Mat::zeros( image.size(), image.type() );

    /// Initialize values
    std::cout << " Basic Linear Transforms " << std::endl;
    std::cout << "-------------------------" << std::endl;
    std::cout << "* Enter the alpha value [1.0-3.0]: ";
    std::cin >> alpha;
    std::cout << "* Enter the beta value [0-100]: ";
    std::cin >> beta;

    /// Do the operation new_image(i,j) = alpha*image(i,j) + beta
    for( int y = 0; y < image.rows; y++ )
    { for( int x = 0; x < image.cols; x++ )
        { for( int c = 0; c < 3; c++ )
            {
                new_image.at<cv::Vec3b>(y,x)[c] =
                        cv::saturate_cast<uchar>( alpha*( image.at<cv::Vec3b>(y,x)[c] ) + beta );
            }
        }
    }

    /// Show stuff
    cv::imshow("Original Image", image);
    cv::imshow("New Image", new_image);
    cv::imwrite("D:/TRANSLATION/OpenCV Book for PGU/PICTURES/USED IN CODES/out2.jpg", new_image);

    /// Wait until user press some key
    cv::waitKey();

    return 0;
}
```



## توضیح

در خط 12، به دلیل اینکه می‌خواهیم تغییراتی در تصویر ورودی ایجاد کنیم، یک شیء جدید Mat درست می‌کنیم. همچنین می‌خواهیم این Mat جدید ویژگی‌های زیر را هم داشته باشد:

-   مقدار اولیهٔ صفر در همهٔ پیکسل‌ها
-   اندازه و نوع مطابق با عکس اصلی

تابع Mat::zeros یک شیء Mat با اندازه و نوع داده شده بر می‌گرداند که همهٔ پیکسل‌های آن صفر است.

حالا برای تغییر کنتراست و روشنایی، باید به همهٔ پیکسل‌های تصویر ورودی دسترسی پیدا کنیم. به دلیل اینکه با RGB کار می‌کنیم، به ازای هر پیکسل 3 مقدار داریم (R، G و B)؛ پس باید به هر کدام از آنها جداگانه دسترسی پیدا کنیم. خطوط 23 تا 31 همین کار را انجام می‌دهند.

به نکات زیر توجه کنید:

-   برای دسترسی به هر کدام از پیکسل‌ها از قاعدهٔ روبرو استفاده می‌کنیم: `image.at<Vec3b>(y,x)[c]` که y شماره سطر، x شماره ستون و c هم R، G یا B یعنی (0، 1 یا 2) است.
-   به خاطر اینکه عمل $\alpha.f(i,j)+\beta$ ممکن است مقادیری بزرگتر از حد مجاز یا غیر صحیح (اگر آلفا اعشاری باشد) تولید کند، از تابع saturate\_cast استفاده می‌کنیم تا مطمئن شویم که مقدار معتبری در پیکسل خروجی قرار می‌گیرد.

در آخر خروجی را در یک پنجره نشان می‌دهیم.

**نکته: به جای استفاده از حلقهٔ for برای اعمال فرمول بالا، می‌توانستیم از تابع زیر استفاده کنیم:**

```c++
image.convertTo(new_image, -1, alpha, beta);
```

**این تابع مقدار new\_image را از معادلهٔ $new_{image} = \alpha * image + \beta$ به دست می‌آورد. به هر حال هدف ما در این بخش این بود که شما را با نحوهٔ دسترسی مستقیم به پیکسل‌ها و انجام تبدیلات پیکسلی آشنا کنیم. هر دو روش خروجی یکسانی دارند.**



## خروجی

برنامه را به صورت زیر اجرا می‌کنیم:

```powershell
BasicLinearTransform.exe "PICTURES /2.jpg"
> Basic Linear Transforms
> -------------------------
> * Enter the alpha value [1.0-3.0]: 1.5
> * Enter the beta value [0-100]: 12
```

خروجی به صورت زیر است:

| ![تصویر ورودی](/opencv-book/media/image30.jpg) | ![تصویر خروجی](/opencv-book/media/image35.jpeg) |
| :--------------------------------------------: | :---------------------------------------------: |
|                  تصویر ورودی                   |                   تصویر خروجی                   |



