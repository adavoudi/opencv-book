---
utid: 1000-02-18
chapter: 02
chaptername: فصل دوم - حوزهٔ مکان
part: 18
title: پوش محدب
---

پوشِ محدبِ مجموعه‌ای از نقاط صفحه، کوچک‌ترین چند ضلعی محدبی است که تمامی نقاط درون آن یا روی محیط آن قرار داشته باشند.

برای این که تصور بهتری از پوش محدب به دست آورید، نقاط صفحه را مانند میخ‌هایی در نظر بگیرید که به دیوار کوبیده شده‌اند. حال کش تنگی را درنظر بگیرید که همه میخ‌ها را احاطه کرده است. در این صورت پوش محدب نقاط شکلی خواهد بود که کش به خود می‌گیرد.

| ![نحوه پیدا کردن پوش مخدب مجموعه‌ای از نقاط](/opencv-book/media/image92.png) |
| :----------------------------------------------------------: |
|           نحوه پیدا کردن پوش محدب مجموعه‌ای از نقاط           |



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

    createTrackbar( " Threshold:", "Source", &thresh, max_thresh, thresh_callback );
    thresh_callback( 0, 0 );

    waitKey(0);
    return(0);
}

/** @function thresh_callback */
void thresh_callback(int, void* )
{
    Mat src_copy = src.clone();
    Mat threshold_output;
    vector<vector<Point> > contours;
    vector<Vec4i> hierarchy;

    /// Detect edges using Threshold
    threshold( src_gray, threshold_output, thresh, 255, THRESH_BINARY );

    /// Find contours
    findContours( threshold_output, contours, hierarchy, CV_RETR_TREE, CV_CHAIN_APPROX_SIMPLE, Point(0, 0) );

    /// Find the convex hull object for each contour
    vector<vector<Point> >hull( contours.size() );
    for( int i = 0; i < contours.size(); i++ )
    {  convexHull( Mat(contours[i]), hull[i], false ); }

    /// Draw contours + hull results
    Mat drawing = Mat::zeros( threshold_output.size(), CV_8UC3 );
    for( int i = 0; i< contours.size(); i++ )
    {
        Scalar color = Scalar( rng.uniform(0, 255), rng.uniform(0,255), rng.uniform(0,255) );
        drawContours( drawing, contours, i, color, 1, 8, vector<Vec4i>(), 0, Point() );
        drawContours( drawing, hull, i, color, 1, 8, vector<Vec4i>(), 0, Point() );
    }

    /// Show in a window
    namedWindow( "Hull demo", CV_WINDOW_AUTOSIZE );
    imshow( "Hull demo", drawing );
}
```



## توضیح

در خط 41 تابع thresh_callback تعریف شده است. در این تابع ابتدا یک کپی از ماتریس src به نام src_copy درست می‌کنیم. سپس یک ماتریس برای نگه داری خروجی تابع threshold درست می‌کنیم. همچنین یک بردار دو بعدی از نوع Point برای نگه داری کانتورهای به دست آمده و یک بردار تک بعدی از نوع Vec4i برای نگه داری خروجی اختیاری تابع findContours، یعنی hierarchy، تعریف می‌کنیم.

**نکته: Vec4i یک بردار 1 در 4 است که همهٔ عناصر آن از نوع int هستند.**

در خط 49 با استفاده از تابع threshold تصویر سیاه و سفید را آستانه‌گذاری می‌کنیم و آن را به یک تصویر باینری تبدیل می‌کنیم. برای این کار از آستانه وارد شده توسط کاربر (thresh) استفاده می‌کنیم

با فراخوانی تابع findContours در خط 52، کانتورهای تصویر خروجی تابع threshold (یعنی threshold_output) را پیدا می‌کنیم.

در خط 55 یک بردار دو بعدی به نام hull از نوع Point برای نگه داری پوش محدب هر کدام از کانتورها درست می‌کنیم. سپس در یک حلقه، تابع convexHull را روی هر کدام از کانتورها صدا می‌زنیم و خروجی‌ها را در بردار hull ذخیره می‌کنیم. تابع convexHull دارای 3 آرگومان به شرح زیر است:

1.  **`Mat(contours[i])`:** آرایهٔ ورودی از نقاط است. می‌توانیم آنها را در یک بردار یا یک ماتریس قرار دهیم.
2.  **`hull[i]`:** پوش محدب مربوط به نقاط ورودی در این بردار قرار می‌گیرند.
3.  **false:** جهت پوش محدب را مشخص می‌کند. اگر true باشد به معنی این است که جهت به صورت موافق با عقربه‌های ساعت است و اگر false باشد به معنی خلاف عقربه‌های ساعت است. در هر حالت، نقطهٔ (0,0) در گوشهٔ سمت چپ و بالا قرار دارد.


## خروجی

| ![ورودی](/opencv-book/media/image93.png) | ![خروجی](/opencv-book/media/image94.png) |
| :--------------------------------------: | :--------------------------------------: |
|                  ورودی                   |                  خروجی                   |

چند ضلعی بنفش رنگ که همهٔ تصویر را در برگرفته، همان پوش محدب تصویر دست است.