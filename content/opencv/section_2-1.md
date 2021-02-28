---
utid: 1000-02-01
chapter: 02
chaptername: فصل دوم - حوزهٔ مکان
part: 01
title: شکل‌های هندسی ساده
_index: simple-shapes
---

در این بخش می‌خواهیم طریقه ترسیم شکل‌های هندسی ساده را آموزش دهیم. این شکل‌ها عبارتند از خط، دایره، بیضی، مستطیل، چند ضلعی.



## کد

```c++
#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>
#include <iostream>
#define w 400

using namespace cv;

/// Function headers
void MyEllipse( Mat img, double angle );
void MyFilledCircle( Mat img, Point center );
void MyPolygon( Mat img );
void MyLine( Mat img, Point start, Point end );

/**
 * @function main
 * @brief Main function
 */
int main( void ){

    /// Windows names
    char atom_window[] = "Drawing 1: Atom";
    char rook_window[] = "Drawing 2: Rook";

    /// Create black empty images
    Mat atom_image = Mat::zeros( w, w, CV_8UC3 );
    Mat rook_image = Mat::zeros( w, w, CV_8UC3 );

    /// 1. Draw a simple atom:
    /// -----------------------

    /// 1.a. Creating ellipses
    MyEllipse( atom_image, 90 );
    MyEllipse( atom_image, 0 );
    MyEllipse( atom_image, 45 );
    MyEllipse( atom_image, -45 );

    /// 1.b. Creating circles
    MyFilledCircle( atom_image, Point( w/2, w/2) );

    /// 2. Draw a rook
    /// ------------------

    /// 2.a. Create a convex polygon
    MyPolygon( rook_image );

    /// 2.b. Creating rectangles
    rectangle(rook_image, Point( 0, 7*w/8 ), Point( w, w), Scalar( 0, 255, 255 ), -1, 8);

    /// 2.c. Create a few lines
    MyLine( rook_image, Point( 0, 15*w/16 ), Point( w, 15*w/16 ) );
    MyLine( rook_image, Point( w/4, 7*w/8 ), Point( w/4, w ) );
    MyLine( rook_image, Point( w/2, 7*w/8 ), Point( w/2, w ) );
    MyLine( rook_image, Point( 3*w/4, 7*w/8 ), Point( 3*w/4, w ) );

    /// 3. Display your stuff!
    imshow( atom_window, atom_image );
    moveWindow( atom_window, 0, 200 );
    imshow( rook_window, rook_image );
    moveWindow( rook_window, w, 200 );

    waitKey( 0 );
    return(0);
}

/// Function Declaration

/**
 * @function MyEllipse
 * @brief Draw a fixed-size ellipse with different angles
 */
void MyEllipse( Mat img, double angle )
{
    int thickness = 2;
    int lineType = 8;

    ellipse( img, Point( w/2, w/2 ), Size( w/4, w/16 ),
        angle, 0, 360, Scalar( 255, 0, 0 ), thickness, lineType );
}

/**
 * @function MyFilledCircle
 * @brief Draw a fixed-size filled circle
 */
void MyFilledCircle( Mat img, Point center )
{
    int thickness = -1;
    int lineType = 8;

    circle(img, center, w/32, Scalar( 0, 0, 255 ), thickness, lineType);
}

/**
 * @function MyPolygon
 * @function Draw a simple concave polygon (rook)
 */
void MyPolygon( Mat img )
{
    int lineType = 8;

    /** Create some points */
    Point rook_points[1][20];
    rook_points[0][0]  = Point(    w/4,   7*w/8 );
    rook_points[0][1]  = Point(  3*w/4,   7*w/8 );
    rook_points[0][2]  = Point(  3*w/4,  13*w/16 );
    rook_points[0][3]  = Point( 11*w/16, 13*w/16 );
    rook_points[0][4]  = Point( 19*w/32,  3*w/8 );
    rook_points[0][5]  = Point(  3*w/4,   3*w/8 );
    rook_points[0][6]  = Point(  3*w/4,     w/8 );
    rook_points[0][7]  = Point( 26*w/40,    w/8 );
    rook_points[0][8]  = Point( 26*w/40,    w/4 );
    rook_points[0][9]  = Point( 22*w/40,    w/4 );
    rook_points[0][10] = Point( 22*w/40,    w/8 );
    rook_points[0][11] = Point( 18*w/40,    w/8 );
    rook_points[0][12] = Point( 18*w/40,    w/4 );
    rook_points[0][13] = Point( 14*w/40,    w/4 );
    rook_points[0][14] = Point( 14*w/40,    w/8 );
    rook_points[0][15] = Point(    w/4,     w/8 );
    rook_points[0][16] = Point(    w/4,   3*w/8 );
    rook_points[0][17] = Point( 13*w/32,  3*w/8 );
    rook_points[0][18] = Point(  5*w/16, 13*w/16 );
    rook_points[0][19] = Point(    w/4,  13*w/16 );

    const Point* ppt[1] = { rook_points[0] };
    int npt[] = { 20 };

    fillPoly(img, ppt, npt, 1, Scalar( 255, 255, 255 ), lineType);
}

/**
 * @function MyLine
 * @brief Draw a simple line
 */
void MyLine( Mat img, Point start, Point end )
{
    int thickness = 2;
    int lineType = 8;
    line(img, start, end, Scalar( 0, 0, 0 ), thickness, lineType);
}
```



## توضیح

از آنجایی که می‌خواهیم دو شکل رسم کنیم (یک اتم و یک قلعه)، باید دو ماتریس به عنوان صفحهٔ ترسیم و دو پنجره برای نمایش آنها درست کنیم (خطوط 25 و 26).

برای کشیدن هر شکل یک تابع جداگانه درست کرده‌ایم. به عنوان مثال برای کشیدن اتم از توابع MyEllipse و MyFilledCircle استفاده کرده‌ایم (خطوط 28 تا 39).

برای کشیدن قلعه از توابع MyLine، rectangle و یک MyPolygon استفاده کرده‌ایم (خطوط 40 تا 53).

بهتر است ببینیم درون این توابع چه می‌گذرد:

- تابع MyLine (خطوط 133 تا 138)

  همان طور که می‌بینید این تابع فقط به روش زیر چندین بار از تابع line (که در ماژول Core موجود است) استفاده می‌کند:

  -   یک خط از نقطهٔ start به نقطهٔ end می‌کشد.
  -   خط بالا را در تصویر img رسم می‌کند.
  -   رنگ خط با یک اسکالر به فرم `Scalar(0,0,0)`، که نشان دهندهٔ رنگ سیاه RGB است، به تابع line فرستاده شده.
  -   میزان نازکی خط با متغیر thickness مشخص شده است.
  -   این خط از نوع متصل است و این مورد با متغیر lineType که مقدار 2 دارد مشخص شده است.

- تابع MyEllipse (خطوط 71 تا 78)

  این تابع به شکل زیر یک بیضی می‌کشد:

  -   بیضی در عکس img ترسیم می‌شود.
  -   مرکز بیضی نقطهٔ (w/2,w/2) است و در یک مستطیل به اندازهٔ (w/4,w/16) محدود شده.
  -   بیضی به اندازه angle درجه چرخانده می‌شود.
  -   بیضی به صورت یک قطاع از زاویهٔ 0 تا 360 است.
  -   رنگ بیضی با `Scalar(255,0,0)` که در RGB یعنی آبی، مشخص شده است.
  -   نازکی و نوع خط بیضی با متغیرهای thickness و lineType مشخص شده‌اند.

- تابع MyFilledCircle (خطوط 84 تا 90)

  آرگومان‌های تابع circle به شرح زیر است:

  -   img عکسی است که دایره در آن کشیده می‌شود.
  -   مرکز دایره با نقطهٔ center مشخص شده است.
  -   شعاع دایره W/32 است.
  -   رنگ دایره با `Scalar(0,0,255)` که در RGB به معنی قرمز است، مشخص شده است.
  -   به خاطر اینکه مقدار thickness عدد 1- قرار داده شده، دایره توپُر کشیده می‌شود.

- تابع MyPolygon (خطوط 96 تا 127)

  برای کشیدن یک چند ضلعی توپر، از تابع fillPoly که در ماژول Core موجود است استفاده می‌کنیم. قابل ذکر است که:

  -   چندضلعی روی عکس img کشیده می‌شود.
  -   رأس‌های چند ضلعی همان مجموعه نقاطی هستند که در ppt قرار داده شده‌اند.
  -   تعداد رأس‌هایی که باید رسم شوند در متغیر npt قرار داده شده است.
  -   تعداد چند ضلعی‌هایی که باید رسم شوند، 1 عدد است.
  -   رنگ چند ضلعی با `Scalar(255,255,255)` که در RGB به معنای سفید است، مشخص شده است.

- تابع rectangle (خط 47)

  در آخر تابع rectangle را معرفی می‌کنیم. در این تابع:

  -   مستطیل روی عکس rook\_image کشیده می‌شود.
  -   دو رأس مخالف مستطیل با `Point(0,7\*w/8.0)` و `Point(w,w)` مشخص شده‌اند.
  -   رنگ مستطیل با `Scalar(0,255,255)` که نشان دهندهٔ رنگ زرد در سیستم RGB است، مشخص شده است.
  -   به خاطر اینکه مقدار thickness عدد 1- قرار داده شده است، مستطیل توپُر خواهد بود.



## خروجی

خروجی برنامه به شکل زیر است:

| ![mage1](/opencv-book/media/image11.jpg) | ![mage1](/opencv-book/media/image12.jpg) |
| :--------------------------------------: | :--------------------------------------: |
|                                          |                                          |

