---
utid: 1000-02-17
chapter: 02
chaptername: فصل دوم - حوزهٔ مکان
part: 17
title: کانتورهای تصویر
---

## کد

```c++
#include "opencv2/highgui/highgui.hpp"
#include "opencv2/imgproc/imgproc.hpp"
#include <iostream>
#include <stdio.h>
#include <stdlib.h>

using namespace cv;
using namespace std;

Mat src; Mat src_gray;
int thresh = 100;
int max_thresh = 255;
RNG rng(12345);

/// Function header
void thresh_callback(int, void* );

/** @function main */
int main( int argc, char** argv )
{
    /// Load source image and convert it to gray
    src = imread( argv[1], 1 );

    /// Convert image to gray and blur it
    cvtColor( src, src_gray, CV_BGR2GRAY );
    blur( src_gray, src_gray, Size(3,3) );

    /// Create Window
    char* source_window = "Source";
    namedWindow( source_window, CV_WINDOW_AUTOSIZE );
    imshow( source_window, src );

    createTrackbar( " Canny thresh:", "Source", &thresh, max_thresh, thresh_callback );
    thresh_callback( 0, 0 );

    waitKey(0);
    return(0);
}

/** @function thresh_callback */
void thresh_callback(int, void* )
{
    Mat canny_output;
    vector<vector<Point> > contours;
    vector<Vec4i> hierarchy;

    /// Detect edges using canny
    Canny( src_gray, canny_output, thresh, thresh*2, 3 );
    /// Find contours
    findContours( canny_output, contours, hierarchy, CV_RETR_TREE, CV_CHAIN_APPROX_SIMPLE, Point(0, 0) );

    /// Draw contours
    Mat drawing = Mat::zeros( canny_output.size(), CV_8UC3 );
    for( int i = 0; i< contours.size(); i++ )
    {
        Scalar color = Scalar( rng.uniform(0, 255), rng.uniform(0,255), rng.uniform(0,255) );
        drawContours( drawing, contours, i, color, 2, 8, hierarchy, 0, Point() );
    }

    /// Show in a window
    namedWindow( "Contours", CV_WINDOW_AUTOSIZE );
    imshow( "Contours", drawing );
}
```



## توضیح

به منظور کاهش نویز، در خط 26 تصویر سیاه و سفید را با فیلتر 3 در 3 هموار می‌کنیم.

در 41 تابع thresh_callback تعریف شده است. در این تابع ابتدا یک ماتریس برای نگه داری خروجی لبه‌یاب کَنی درست می‌کنیم. همچنین یک بردار دو بعدی از نوع Point برای نگه داری کانتورهای به دست آمده و یک بردار تک بعدی از نوع Vec4i برای نگه داری خروجی اختیاری تابع findContours، یعنی hierarchy، تعریف می‌کنیم.

**نکته: Vec4i یک بردار 1 در 4 است که همهٔ عناصر آن از نوع int هستند.**

در خط 48 با استفاده از تابع Canny لبه‌های تصویر سیاه و سفید را پیدا می‌کنیم. از آستانه‌ای استفاده می‌کنیم که کاربر وارد کرده است (یعنی متغیر thresh).

سپس در خط 50 با فراخوانی تابع findContours، کانتورهای تصویر خروجی از تابع Canny (یعنی canny_output) را پیدا می‌کنیم. این تابع 6 آرگومان به شرح زیر دارد:

1.  **canny\_output:** تصویر ورودی است. این تصویر باید از نوع CV\_8UC1 باشد. در اینجا از خروجی تابع Canny به عنوان ورودی این تابع استفاده می‌کنیم. از آنجا که خروجی تابع Canny یک عکس تک بعدی 8 بیتی است پس مشکلی وجود ندارد.
2.  **contours:** کانتورهای کشف شده هستند. هر کانتور به شکل برداری از نقاط در این متغیر ذخیره می‌شود.
3.  **hierarchy:** بردار خروجی اختیاری که اطلاعاتی در مورد توپولوژی تصویر به ما می‌دهد.
4.  **CV\_RET\_TREE:** شیوهٔ بازیابی کانتورها است. می‌تواند به جای این مقدار، هر کدام از مقادیر زیر باشد:

    -   CV\_RETR\_EXTERNAL

    -   CV\_RETR\_LIST

    -   CV\_RETR\_CCOMP

    -   CV\_RET\_TREE
5.  **CV\_CHAIN\_APPROX\_SIMPLE:** شیوهٔ تقریب کانتورها است. می‌تواند به جای این مقدار، هر کدام از مقادیر زیر باشد:

    1.  CV\_CHAIN\_APPROX\_NONE

    2.  CV\_CHAIN\_APPROX\_SIMPLE

    3.  CV\_CHAIN\_APPROX\_TC89\_L1 و CV\_CHAIN\_APPROX\_TC89\_KCOS
6.  **`Point(0,0)`:** یک آفست اختیاری که هر کدام از کانتورها با آن جمع می‌شوند. در اینجا نقطهٔ (0,0) را به عنوان آفست در نظر گرفته‌ایم. آفست زمانی مفید است که کانتورها از روی یک ROI به دست آمده باشند و قرار باشد بعداً کانتورها را روی عکس اصلی نشان دهیم.

بعد از پیدا کردن کانتورها، هم می‌توانیم آنها را به صورت یکجا (و با یک رنگ) بکشیم و هم اینکه هر کدام از کانتورها را با یک رنگ خاص نمایش دهیم. به هر حال در هر دو حالت از تابع drawContours استفاده می‌کنیم. در حالت اول با یک بار فراخوانی تابع drawContours و در حالت دوم، تابع drawContours را در یک حلقه می‌گذاریم و به تعداد کانتورهای کشف شده حلقه را تکرار می‌کنیم و هر دفعه یکی از کانتورها را می‌کشیم. در اینجا به روش دوم عمل می‌کنیم. تابع drawContours در خط 57 دارای 9 آرگومان به شرح زیر است:

1.  **drawing:** تصویر ورودی است.
2.  **contours:** بردار کانتورها است. هر کانتور به صورت برداری از نقاط در contours قرار می‌گیرند.
3.  **i:** شمارهٔ کانتوری که باید کشیده شود. اگر یک عدد منفی در اینجا قرار دهیم، همهٔ کانتورها کشیده می‌شوند.
4.  **color:** رنگ کانتور است.
5.  **2:** نازکی کانتور است.
6.  **8:** نوع خطی که بین نقاط کانتور کشیده می‌شود.
7.  **hierarchy:** اطلاعات اختیاری در مورد سلسله مراتب عکس است. فقط زمانی نیاز است که بخواهیم بعضی از کانتورها را بکشیم (مثلاً در اینجا که هر دفعه فقط یکی از کانتورها را می‌کشیم، این متغیر نیاز است).
8.  **0:** حداکثر سطح برای کانتورهای کشیده شده است. اگر صفر باشد فقط کانتور مشخص شده کشیده می‌شود. اگر یک باشد کانتور مشخص شده و تمام کانتورهای تو در توی آن را می‌کشد.
9.  **`Point()`:** آفست اختیاری‌ای است که مختصات کانتورها ابتدا با آن جمع و سپس روی عکس کشیده می‌شوند.


## خروجی

| ![تصویر ورودی](/opencv-book/media/image90.png) | ![تصویر خروجی](/opencv-book/media/image91.png) |
| :--------------------------------------------: | :--------------------------------------------: |
|                  تصویر ورودی                   |                  تصویر خروجی                   |

دقت کنید که رنگ کانتورها به صورت تصادفی انتخاب شده‌اند و هیچ ربطی به رنگ‌های تصویر ورودی ندارند.