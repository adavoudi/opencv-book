---
utid: 1000-05-05
chapter: 05
chaptername: فصل پنجم - شناسایی الگو
part: 05
title: تبدیل دایره هُو
---

تبدیل دایرهٔ هُو بسیار شبیه به تبدیل خط هُو که در بخش قبل توضیح داده شد عمل می‌کند. در مورد کشف خط، یک خط با دو پارامتر $\left( r,\ \theta \right)$ تعریف می‌شود ولی برای تعریف دایره به سه پارامتر نیاز داریم:

$$C:\ (x_{\text{center}},\ y_{\text{center}},\ r)$$

که $(x_{\text{center}},\ y_{\text{center}})$ مرکز دایره (نقطهٔ سبز رنگ در شکل زیر) و r هم شعاع آن را مشخص می‌کند. با این سه پارامتر می‌توانیم هر دایره‌ای با هر مختصات و شعاعی تعریف کنیم. دایره قرمز در شکل زیر مثالی از کشف دایره است:

| ![مثالی از کشف دایره](/opencv-book/media/image141.png) |
| :----------------------------------------------------: |
|                   مثالی از کشف دایره                   |

به منظور رعایت بهینگی، اُ سی وی این تبدیل با استفاده از روش گرادیان هُو پیاده سازی کرده است که متفاوت با روش مورد استفاده در تبدیل استاندارد هُو است.



## کد

```c++
#include "opencv2/highgui/highgui.hpp"
#include "opencv2/imgproc/imgproc.hpp"
#include <iostream>
#include <stdio.h>

using namespace cv;

/** @function main */
int main(int argc, char** argv)
{
    Mat src, src_gray;

    /// Read the image
    src = imread( argv[1], 1 );

    if( !src.data )
    { return -1; }

    /// Convert it to gray
    cvtColor( src, src_gray, CV_BGR2GRAY );

    /// Reduce the noise so we avoid false circle detection
    GaussianBlur( src_gray, src_gray, Size(9, 9), 2, 2 );

    vector<Vec3f> circles;

    /// Apply the Hough Transform to find the circles
    HoughCircles( src_gray, circles, CV_HOUGH_GRADIENT, 1, src_gray.rows/8, 200, 100, 0, 0 );

    /// Draw the circles detected
    for( size_t i = 0; i < circles.size(); i++ )
    {
        Point center(cvRound(circles[i][0]), cvRound(circles[i][1]));
        int radius = cvRound(circles[i][2]);
        // circle center
        circle( src, center, 3, Scalar(0,255,0), -1, 8, 0 );
        // circle outline
        circle( src, center, radius, Scalar(0,0,255), 3, 8, 0 );
    }

    /// Show your results
    namedWindow( "Hough Circle Transform Demo", CV_WINDOW_AUTOSIZE );
    imshow( "Hough Circle Transform Demo", src );

    waitKey(0);
    return 0;
}
```



## توضیح

در خط 23 به منظور کاهش نویز و جلوگیری از کشف دایره‌های اشتباه، یک فیلتر گوسی اعمال می‌کنیم.

در خط 28 با استفاده از تابع HoughCircles تبدیل دایره هُو را اعمال می‌کنیم. این تابع دارای 9 آرگومان به شرح زیر است:

1.  **src\_gray:** تصویر ورودی که باید سیاه و سفید باشد.
2.  **circles:** یک بردار که مجموعه‌ای از سه مقدار $x_{c},y_{c},\ r$ را برای هر دایره کشف شده نگه می دارد.
3.  **CV\_HOUGH\_GRADIENT:** روش کشف را مشخص می‌کند. در حال حاضر فقط همین روش در اُ سی وی وجود دارد.
4.  **1:** نسبت معکوس وضوح است.
5.  **src\_gray.rows/8:** حدقل فاصلهٔ بین دایره‌های کشف شده است.
6.  **200:** آستانهٔ بالایی برای لبه‌یاب کَنی که در این تابع استفاده می‌شود.
7.  **100:** آستانه‌ای برای کشف مرکز دایره است.
8.  **0:** حداقل شعاع مجاز برای دایره‌های کشف شده است. اگر معلوم نیست، مقدارش را صفر بگذارید.
9.  **0:** حداکثر شعاع مجاز برای دایره‌های کشف شده است. اگر معلوم نیست، مقدارش را صفر بگذارید.


## خروجی

| ![خروجی برنامه](/opencv-book/media/image142.png) |
| :----------------------------------------------: |
|                   خروجی برنامه                   |

