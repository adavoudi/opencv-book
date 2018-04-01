---
utid: 1000-02-05
chapter: 02
chaptername: فصل دوم - حوزهٔ مکان
part: 05
title: نگاشت
_index: map
---

در این بخش به چگونگی استفاده از تابع remap برای پیاده سازی نگاشت‌های ساده می‌پردازیم.

به فرایند تغییر مکان پیکسل‌های یک تصویر و بردن آنها به مکان‌های دیگر در یک تصویر جدید، نگاشت می گویند. می‌توان عملیات نگاشت هر پیکسل $(x,\ y)$ را به صورت زیر نشان داد:

$$g\left( x,\ y \right) = f\left( h\left( x,\ y \right) \right)$$

که $g$ تصویر نگاشت یافته، $f$ تصویر منبع و $h$ هم تابع نگاشت است.

به عنوان مثال فرض کنید عکسی به نام I داریم و می‌خواهیم یک نگاشت به صورت زیر روی آن اعمال کنیم:

$$h\left( x,\ y \right) = \left( I.cols - x,\ y \right)$$

به راحتی می‌توان دید که نتیجهٔ این نگاشت چرخیدن عکس در جهت x است. مثلاً عکس زیر را در نظر بگیرید:

| ![تصویر نگاشت شده](/opencv-book/media/image28.png) | ![تصویر اصلی](/opencv-book/media/image29.png) |
| :------------------------------------------------: | :-------------------------------------------: |
|                  تصویر نگاشت شده                   |                  تصویر اصلی                   |

به تغییر موقعیت دایره قرمز نسبت به X توجه کنید (با توجه به اینکه x جهت افقی است).



## کد

```c++
#include "opencv2/highgui/highgui.hpp"
#include "opencv2/imgproc/imgproc.hpp"
#include <iostream>
#include <stdio.h>

using namespace cv;

/// Global variables
Mat src, dst;
Mat map_x, map_y;
char* remap_window = "Remap demo";
int ind = 0;

/// Function Headers
void update_map( void );

/**
* @function main
*/
int main( int argc, char** argv )
{
    /// Load the image
    src = imread( argv[1], 1 );

    /// Create dst, map_x and map_y with the same size as src:
    dst.create( src.size(), src.type() );
    map_x.create( src.size(), CV_32FC1 );
    map_y.create( src.size(), CV_32FC1 );

    /// Create window
    namedWindow( remap_window, CV_WINDOW_AUTOSIZE );

    /// Loop
    while( true )
    {
        /// Each 1 sec. Press ESC to exit the program
        int c = waitKey( 1000 );

        if( (char)c == 27 )
        { break; }

        /// Update map_x & map_y. Then apply remap
        update_map();
        remap( src, dst, map_x, map_y, CV_INTER_LINEAR, BORDER_CONSTANT, Scalar(0,0, 0) );

        /// Display results
        imshow( remap_window, dst );
    }
    return 0;
}

/**
* @function update_map
* @brief Fill the map_x and map_y matrices with 4 types of mappings
*/
void update_map( void )
{
    ind = ind%4;

    for( int j = 0; j < src.rows; j++ )
    { for( int i = 0; i < src.cols; i++ )
        {
            switch( ind )
            {
            case 0:
                if( i > src.cols*0.25 && i < src.cols*0.75 && j > src.rows*0.25 && j < src.rows*0.75 )
                {
                    map_x.at<float>(j,i) = 2*( i - src.cols*0.25 ) + 0.5 ;
                    map_y.at<float>(j,i) = 2*( j - src.rows*0.25 ) + 0.5 ;
                }
                else
                { map_x.at<float>(j,i) = 0 ;
                    map_y.at<float>(j,i) = 0 ;
                }
                break;
            case 1:
                map_x.at<float>(j,i) = i ;
                map_y.at<float>(j,i) = src.rows - j ;
                break;
            case 2:
                map_x.at<float>(j,i) = src.cols - i ;
                map_y.at<float>(j,i) = j ;
                break;
            case 3:
                map_x.at<float>(j,i) = src.cols - i ;
                map_y.at<float>(j,i) = src.rows - j ;
                break;
            } // end of switch
        }
    }
    ind++;
}
```



## توضیح

در خط 44 برای انجام نگاشت، تابع remap را صدا می‌زنیم. این تابع پنج آرگومان به شرح زیر دارد:

1.  src: تصویر ورودی
2.  dst: تصویر خروجی که همان اندازهٔ src است.
3.  map\_x: تابع نگاشت در جهت x است. معادل قسمت اول تابع $h\left( i,\ j \right)$ است.
4.  map\_y: مشابه بالایی، فقط در جهت y است. توجه کنید که map\_x و map\_y هر دو هم اندازهٔ src هستند.
5.  CV\_INTER\_LINEAR: نوع دورن یابی ای که برای پیکسل‌های غیر صحیح به کار می‌رود. پیش فرض آن همین مقدار است.
6.  BORDER\_CONSTANT: نوع قابی که به اطراف تصویر اضافه می‌شود.

در این برنامه چهار نوع نگاشت به تصویر ورودی اعمال می‌شود. تابع $h\left( i,\ j \right)$ هر کدام از این نگاشت‌ها به صورت زیر است:

i.  **کاهش اندازهٔ تصویر به نصف و نشان دادن آن در مرکز:**

$$h\left( i,\ j \right) = \left( 2*i - \frac{\text{src.cols}}{2} + 0.5,\ 2*j - \frac{\text{src.rows}}{2} + 0.5 \right)$$

​	برای همهٔ زوج‌های $\left( i,\ j \right)$ به صورتی که:

$$\frac{\text{src.cols}}{4} < i < \frac{3*src.cols}{4}\text{\ and}\frac{\text{src.rows}}{4} < j < \frac{3*src.rows}{4}$$

ii. **بالا به پایین کردن تصویر:** $h\left( i,\ j \right) = \left( i,\ src.rows - j \right)$

iii. **چپ به راست کردن تصویر:** $h\left( i,\ j \right) = \left( \ src.cols - i,\ j \right)$

iv. **ترکیب ii و iii:** $h\left( i,\ j \right) = \left( \ src.cols - i,\ \ src.rows - j \right)$

تمام موارد بالا در خطوط 65 تا 88 پیاده سازی شده‌اند. در اینجا map\_x نشان دهندهٔ مختص اول $h\left( i,\ j \right)$ و map\_y مختص دوم آن است.



## خروجی

| ![تصویر ورودی](/opencv-book/media/image30.jpg) | ![بالا به پایین کردن تصویر ورودی](/opencv-book/media/image31.jpeg) |
| :--------------------------------------------: | :----------------------------------------------------------: |
|                  تصویر ورودی                   |                بالا به پایین کردن تصویر ورودی                |


| ![چپ به راست کردن تصویر ورودی](/opencv-book/media/image32.jpeg) | ![بالا به پایین و چپ به راست کردن تصویر ورودی](/opencv-book/media/image33.jpg) |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|                 چپ به راست کردن تصویر ورودی                  |         بالا به پایین و چپ به راست کردن تصویر ورودی          |


| ![کاهش اندازهٔ تصویر به نصف و نشان دادن آن در مرکز](/opencv-book/media/image34.jpg) |
| :----------------------------------------------------------: |
|       کاهش اندازهٔ تصویر به نصف و نشان دادن آن در مرکز        |



