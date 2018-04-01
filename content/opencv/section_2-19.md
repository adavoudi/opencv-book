---
utid: 1000-02-19
chapter: 02
chaptername: فصل دوم - حوزهٔ مکان
part: 19
title: قاب‌های مستطیلی و دایره‌ای برای کانتورها
_index: contours-frames
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

    createTrackbar( " Threshold:", "Source", &thresh, max_thresh, thresh_callback );
    thresh_callback( 0, 0 );

    waitKey(0);
    return(0);
}

/** @function thresh_callback */
void thresh_callback(int, void* )
{
    Mat threshold_output;
    vector<vector<Point> > contours;
    vector<Vec4i> hierarchy;

    /// Detect edges using Threshold
    threshold( src_gray, threshold_output, thresh, 255, THRESH_BINARY );
    /// Find contours
    findContours( threshold_output, contours, hierarchy, CV_RETR_TREE, CV_CHAIN_APPROX_SIMPLE, Point(0, 0) );

    /// Approximate contours to polygons + get bounding rects and circles
    vector<vector<Point> > contours_poly( contours.size() );
    vector<Rect> boundRect( contours.size() );
    vector<Point2f>center( contours.size() );
    vector<float>radius( contours.size() );

    for( int i = 0; i < contours.size(); i++ )
    { approxPolyDP( Mat(contours[i]), contours_poly[i], 3, true );
        boundRect[i] = boundingRect( Mat(contours_poly[i]) );
        minEnclosingCircle( (Mat)contours_poly[i], center[i], radius[i] );
    }


    /// Draw polygonal contour + bonding rects + circles
    Mat drawing = Mat::zeros( threshold_output.size(), CV_8UC3 );
    for( int i = 0; i< contours.size(); i++ )
    {
        Scalar color = Scalar( rng.uniform(0, 255), rng.uniform(0,255), rng.uniform(0,255) );
        drawContours( drawing, contours_poly, i, color, 1, 8, vector<Vec4i>(), 0, Point() );
        rectangle( drawing, boundRect[i].tl(), boundRect[i].br(), color, 2, 8, 0 );
        circle( drawing, center[i], (int)radius[i], color, 2, 8, 0 );
    }

    /// Show in a window
    namedWindow( "Contours", CV_WINDOW_AUTOSIZE );
    imshow( "Contours", drawing );
}
```



## توضیح

در خط 41 تابع thresh_callback تعریف شده است. در این تابع ابتدا یک ماتریس برای نگه داری خروجی تابع threshold درست می‌کنیم. همچنین یک بردار دو بعدی از نوع Point برای نگه داری کانتورهای به دست آمده و یک بردار تک بعدی از نوع Vec4i برای نگه داری خروجی اختیاری تابع findContours، یعنی hierarchy، تعریف می‌کنیم.

در خط 48 با استفاده از تابع threshold تصویر سیاه و سفید را آستانه‌گذاری می‌کنیم و آن را به یک تصویر باینری تبدیل می‌کنیم. برای این کار از آستانه وارد شده توسط کاربر (thresh) استفاده می‌کنیم.

با فراخوانی تابع findContours، کانتورهای تصویر خروجی تابع threshold (یعنی threshold_output) را پیدا می‌کنیم.

در خطوط 53 تا 56 بردارهایی برای نگه داری پارامترهای قاب‌های مستطیلی و دایره‌ای درست می‌کنیم.

در حلقه موجود در خط 58 به ترتیب کارهای زیر را انجام می‌دهیم:

1.  ابتدا در خط 59 با استفاده از تابع approxPolyDP برای `contours[i]` یک چند ضلعی تقریبی پیدا می‌کنیم و آن را در `contours_poly[i]` ذخیره می‌کنیم. عدد 3 دقت تقریب را مشخص می‌کند. به عبارتی می‌توان گفت که این عدد حداکثر فاصلهٔ بین منحنی تقریبی و منحنی اصلی است. true هم به معنی این است که می‌خواهیم یک چند ضلعی بسته داشته باشیم (یعنی در منحنی تقریبی، اولین نقطه به آخرین نقطه متصل باشد).
2.  سپس در خط 60 با استفاده از تابع boundingRect قاب مستطیلی مربوط به چند ضلعی تقریبی به دست آمده برای `contours[i]` را حساب و آن را در `boundRect[i]` ذخیره می‌کنیم. ورودی این تابع همان خروجی تابع approxPolyDP است که در یک ماتریس قرار داده شده (می‌توان آن را در یک بردار هم قرار داد).
3.  بعد در خط 61 با استفاده از تابع minEnclosingCircles، قاب دایره‌ای مربوط به چند ضلعی تقریبی به دست آمده برای `contours[i]` را حساب می‌کنیم و مرکز دایره را در `center[i]` و شعاع آن را در `radius[i]` ذخیره می‌کنیم.

سپس در خطوط 67 تا 73 در یک حلقه حرکت می‌کنیم و تمام کانتورها و قاب‌های دایره‌ای و مستطیلی مربوط به آنها را روی تصویر drawing می‌کشیم (برای کشیدن کانتورها از تابع drawContours و برای کشیدن دایره‌ها و مستطیل‌ها از توابع circle و rectangle استفاده می‌کنیم).



## خروجی

| ![تصویر ورودی](/opencv-book/media/image95.png) | ![تصویر خروجی](/opencv-book/media/image96.png) |
| :--------------------------------------------: | :--------------------------------------------: |
|                  تصویر ورودی                   |                  تصویر خروجی                   |

