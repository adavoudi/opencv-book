---
utid: 1000-02-20
chapter: 02
chaptername: فصل دوم - حوزهٔ مکان
part: 20
title: قاب‌های چرخیدهٔ مستطیلی و بیضوی برای کانتورها
_index: contours-rotated-frames
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

    /// Find the rotated rectangles and ellipses for each contour
    vector<RotatedRect> minRect( contours.size() );
    vector<RotatedRect> minEllipse( contours.size() );

    for( int i = 0; i < contours.size(); i++ )
    { minRect[i] = minAreaRect( Mat(contours[i]) );
        if( contours[i].size() > 5 )
        { minEllipse[i] = fitEllipse( Mat(contours[i]) ); }
    }

    /// Draw contours + rotated rects + ellipses
    Mat drawing = Mat::zeros( threshold_output.size(), CV_8UC3 );
    for( int i = 0; i< contours.size(); i++ )
    {
        Scalar color = Scalar( rng.uniform(0, 255), rng.uniform(0,255), rng.uniform(0,255) );
        // contour
        drawContours( drawing, contours, i, color, 1, 8, vector<Vec4i>(), 0, Point() );
        // ellipse
        ellipse( drawing, minEllipse[i], color, 2, 8 );
        // rotated rectangle
        Point2f rect_points[4]; minRect[i].points( rect_points );
        for( int j = 0; j < 4; j++ )
            line( drawing, rect_points[j], rect_points[(j+1)%4], color, 1, 8 );
    }

    /// Show in a window
    namedWindow( "Contours", CV_WINDOW_AUTOSIZE );
    imshow( "Contours", drawing );
}
```



## توضیح

در خط 41 تابع thresh_callback تعریف شده است. در این تابع ابتدا یک ماتریس برای نگه داری خروجی تابع threshold درست می‌کنیم. همچنین یک بردار دو بعدی از نوع Point برای نگه داری کانتورهای به دست آمده و یک بردار تک بعدی از نوع Vec4i برای نگه داری خروجی اختیاری تابع findContours، یعنی hierarchy، تعریف می‌کنیم.

در خط 48 با استفاده از تابع threshold تصویر سیاه و سفید را آستانه‌گذاری می‌کنیم و آن را به یک تصویر باینری تبدیل می‌کنیم. برای این کار از آستانه وارد شده توسط کاربر (thresh) استفاده می‌کنیم.

در خط 50 با فراخوانی تابع findContours، کانتورهای تصویر خروجی تابع threshold (یعنی threshold_output) را پیدا می‌کنیم.

در خطوط 53 و 54 دو بردار از نوع RotatedRect برای نگه داری مستطیل‌ها و بیضی‌های کشف شده درست می‌کنیم.

در حلقه موجود در خط 56، روی هر کدام از کانتورها عملیات زیر را انجام می‌دهیم:

1.  ابتدا در خط 57 با استفاده از تابع minAreaRect کوچک‌ترین مستطیل در بر گیرندهٔ همهٔ نقاط کانتور `contours[i]` را پیدا می‌کنیم و آن را در `minRect[i]` قرار می‌دهیم.
2.  سپس در خط 59 با استفاده از تابع fitEllipse و به شرطی که تعداد نقاط موجود در `contours[i]` بیشتر از 5 عدد باشد، کوچترین بیضی که در برگیرندهٔ همهٔ نقاط `contours[i]` است را پیدا می‌کنیم و آن را در `minEllipse[i]` قرار می‌دهیم.

سپس در خطوط 64 تا 75 در یک حلقه حرکت می‌کنیم و کانتورها و قاب‌های مستطیلی و بیضوی آنها را روی تصویر drawing می‌کشیم. برای کشیدن قاب‌های مستطیلی چرخیده، چون تابعی برای این کار نداریم، ابتدا نقاط مربوط به چهار گوشهٔ مستطیل را به دست می‌آوریم و سپس مستطیل را به صورت چهار خط می‌کشیم.



## خروجی

| ![تصویر ورودی](/opencv-book/media/image97.png) | ![تصویر خروجی](/opencv-book/media/image98.png) |
| :--------------------------------------------: | :--------------------------------------------: |
|                  تصویر ورودی                   |                  تصویر خروجی                   |

