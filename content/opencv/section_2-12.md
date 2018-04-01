---
utid: 1000-02-12
chapter: 02
chaptername: فصل دوم - حوزهٔ مکان
part: 12
title: بافت‌نگار تصویر
_index: image-histogram
---

در علم آمار **هیستوگرام** یا **بافت‌نگار** یک نمودار ستونی و پله‌ای برای نشان دادن داده‌ها است. نمودار بافت‌نگار همانند نمودار ستونی است و تنها اختلافی که بین این دو وجود دارد، نمایش ستون‌هاست.

در پردازش تصویر به نموداری که توسط آن تعداد پیکسل‌های هر سطح روشنایی در تصویر ورودی مشخص می‌شود بافت‌نگار تصویر می گویند. به عنوان مثال به شکل زیر توجه کنید:

| ![تصویری از مقادیر موجود در درایه‌های یک ماتریس](/opencv-book/media/image67.png) |
| :----------------------------------------------------------: |
|         تصویری از مقادیر موجود در درایه‌های یک ماتریس         |

از آنجایی که دامنه مقادیر این داده‌ها معلوم است (بین 0 تا 255 است)، می‌توان آن دامنه را به زیر قسمت‌هایی که سطل[^a] نام دارند، تقسیم کرد:

$$\left\lbrack 0,\ 255 \right\rbrack = \left\lbrack 0,\ 15 \right\rbrack \cup \left\lbrack 16,\ 31 \right\rbrack \cup \ldots\  \cup \left\lbrack 240,\ 255 \right\rbrack$$

$$domain = bin_{1} \cup bin_{2} \cup \ldots \cup bin_{n} = 15$$

بدین ترتیب می‌توان تعداد پیکسل‌هایی که در سطل $bin_{i}$ قرار می‌گیرند را به دست آورد. با اجرای این روش روی ماتریس بالا، نمودار زیر را به دست می‌آید:

| ![محور افقی نشان دهنده سطل‌ها و محور عمودی نشان دهنده تعداد پیکسل‌های موجود در آن سطل است](/opencv-book/media/image68.png) |
| :----------------------------------------------------------: |
| محور افقی نشان دهنده سطل‌ها و محور عمودی نشان دهنده تعداد پیکسل‌های موجود در آن سطل است. |

این فقط یک مثال از طرز کار بافت‌نگار و همچنین دلیل مفید بودن آن بود. کاربرد بافت‌نگار فقط مختص شمارش روشنایی پیکسل‌های تصویر نیست، بلکه می‌توان از آن برای اندازه گیری ویژگی‌های دیگر تصویر هم استفاده کرد (مثل گرادیان، جهت‌ها و...).

یک بافت‌نگار از قسمت‌های زیر تشکیل شده است:

1.  ابعاد[^b]: تعداد پارامترهایی که قرار است برای آنها داده جمع کنیم. در مثال ما این مقدار یک است؛ چون فقط مقادیر روشنایی هر پیکسل را شمارش می‌کنیم (در یک تصویر سیاه و سفید).
2.  تعداد سطل‌ها: تعداد زیر قسمت‌های موجود در هر بُعد است. در مثال ما این مقدار برابر با 16 عدد است.
3.  بازه دامنه[^c]: حدود مقادیر مورد اندازه گیری است. در مثال ما به صورت $\[0, 255\]$ است.

در صورتی که دو ویژگی اندازه گیری شود، بافت‌نگار به دست آمده به صورت سه بعدی در می‌آید (محور x و محور y نشان دهندهٔ $bin_{x}$ و $bin_{y}$ و محور z نشان دهندهٔ عدد شمارش شده برای هر ترکیب $(bin_{x},bin_{y})$ است). برای ابعاد بیشتر به همین صورت محور های بیشتری داریم.

[^a]: bin

[^b]: dims

[^c]: Domain



## کد

```c++
#include "opencv2/highgui/highgui.hpp"
#include "opencv2/imgproc/imgproc.hpp"
#include <iostream>
#include <stdio.h>

using namespace std;
using namespace cv;

/**
 * @function main
 */
int main( int argc, char** argv )
{
    Mat src, dst;

    /// Load image
    src = imread( argv[1], 1 );

    if( !src.data )
    { return -1; }

    /// Separate the image in 3 places ( B, G and R )
    vector<Mat> bgr_planes;
    split( src, bgr_planes );

    /// Establish the number of bins
    int histSize = 256;

    /// Set the ranges ( for B,G,R) )
    float range[] = { 0, 256 } ;
    const float* histRange = { range };

    bool uniform = true; bool accumulate = false;

    Mat b_hist, g_hist, r_hist;

    /// Compute the histograms:
    calcHist( &bgr_planes[0], 1, 0, Mat(), b_hist, 1, &histSize, &histRange, uniform, accumulate );
    calcHist( &bgr_planes[1], 1, 0, Mat(), g_hist, 1, &histSize, &histRange, uniform, accumulate );
    calcHist( &bgr_planes[2], 1, 0, Mat(), r_hist, 1, &histSize, &histRange, uniform, accumulate );

    // Draw the histograms for B, G and R
    int hist_w = 512; int hist_h = 400;
    int bin_w = cvRound( (double) hist_w/histSize );

    Mat histImage( hist_h, hist_w, CV_8UC3, Scalar( 0,0,0) );

    /// Normalize the result to [ 0, histImage.rows ]
    normalize(b_hist, b_hist, 0, histImage.rows, NORM_MINMAX, -1, Mat() );
    normalize(g_hist, g_hist, 0, histImage.rows, NORM_MINMAX, -1, Mat() );
    normalize(r_hist, r_hist, 0, histImage.rows, NORM_MINMAX, -1, Mat() );

    /// Draw for each channel
    for( int i = 1; i < histSize; i++ )
    {
        line( histImage, Point( bin_w*(i-1), hist_h - cvRound(b_hist.at<float>(i-1)) ) ,
              Point( bin_w*(i), hist_h - cvRound(b_hist.at<float>(i)) ),
              Scalar( 255, 0, 0), 2, 8, 0  );
        line( histImage, Point( bin_w*(i-1), hist_h - cvRound(g_hist.at<float>(i-1)) ) ,
              Point( bin_w*(i), hist_h - cvRound(g_hist.at<float>(i)) ),
              Scalar( 0, 255, 0), 2, 8, 0  );
        line( histImage, Point( bin_w*(i-1), hist_h - cvRound(r_hist.at<float>(i-1)) ) ,
              Point( bin_w*(i), hist_h - cvRound(r_hist.at<float>(i)) ),
              Scalar( 0, 0, 255), 2, 8, 0  );
    }

    /// Display
    namedWindow("calcHist Demo", CV_WINDOW_AUTOSIZE );
    imshow("calcHist Demo", histImage );

    waitKey(0);

    return 0;
}
```



## توضیح

در خط 24 با استفاده از تابع split تصویر ورودی را به صفحات R، G و B تقسیم می‌کنیم. ورودی اول این تابع تصویری است که می‌خواهیم آن را تقسیم کنیم و ورودی دوم آن برداری است که صفحات (ماتریس‌های) خروجی در آن قرار می‌گیرند.

حالا همه چیز برای حساب کردن بافت‌نگارهای صفحات آماده است. از آنجایی که با صفحات B، G و R کار می‌کنیم، پس می دانیم که برد مقادیر در بازهٔ $\[0, 255\]$ است.

در خط 27 تعداد سطل‌ها را مشخص می‌کنیم.

در خطوط 30 و 31 برد مقادیر را مشخص می‌کنیم (همانطور که گفتیم در بازهٔ $\[0, 255\]$ است).

چون می‌خواهیم که سطل‌ها هم اندازه و بافت‌نگارها هم در ابتدا خالی باشند، در خط 33 متغیر uniform را برابر true و متغیر accumulate را برابر false قرار می‌دهیم.

در ادامه برای حساب کردن بافت‌نگارها از تابع calcHist استفاده می‌کنیم:

در خطوط 38 تا 40 برای حساب کردن بافت‌نگار هر صفحه از تابع calcHist استفاده می‌کنیم. این تابع 10 آرگومان دارد:

1.  **`&bgr_planes[0]`:** صفحهٔ (یا آرایهٔ) ورودی است (که در این مورد صفحهٔ 0 است).

2.  **1:** تعداد آرایه‌های ورودی است (در اینجا مقدارش 1 است. می‌توان لیستی از آرایه‌ها به ورودی تابع داد و مقدار بیشتری را در اینجا مشخص کرد).

3.  **0:** کانالی که باید اندازه گیری شود. در این مورد فقط روشنایی است (آرایه‌ها به صورت تک کاناله هستند)، پس باید 0 گذاشت.

4.  **`Mat()`:** ماتریسی که به عنوان ماسک برای ماتریس ورودی استفاده می‌شود (مقدار صفر در ماسک بدین معنی است که پیکسل متناظر آن در ماتریس ورودی نباید در شمارش شرکت داشته باشد). اگر این ماتریس تعریف نشود، از ماسکی استفاده نخواهد شد.

5.  **b\_hist:** بافت‌نگار خروجی در این ماتریس ذخیره می‌شود.

6.  **1:** ابعاد بافت‌نگار

7.  **histSize:** تعداد سطل‌ها در هر کدام از بُعدها

8.  **histRange:** برد مقادیر مورد اندازه گیری در هر بُعد

9.  **uniform:** مشخص می‌کند که اندازه سطل‌ها برابر باشند یا نه.

10.  **accumulate:** مشخص می‌کند که اندازه سطل‌ها در ابتدا خالی باشند یا نه.

در خطوط 52 تا 57، قبل از ترسیم بافت‌نگار ها، مقادیر آنها را با استفاده از تابع normalize نرمال می‌کنیم تا بین دو مقدار تعیین شده قرار بگیرند. این تابع 6 آرگومان دارد:

1.  **b\_hist:** آرایه ورودی

2.  **b\_hist:** آرایه خروجی

3.  **0:** حد پایینی برای نرمال سازی مقادیر آرایه‌های ورودی

4.  **histImage.rows:** حد بالایی برای نرمال سازی مقادیر آرایه‌های ورودی

5.  **NORM\_MINMAX:** این آرگومان بیان کنندهٔ نوع فرایند نرمال سازی است (در این نوع، مقادیر بین دو مقدار MIN و MAX قرار می‌گیرند).

6.  **1-:** نوع داده آرایه خروجی را مشخص می‌کند. در اینجا با قرار دادن -1 نوع آرایهٔ خروجی با نوع آرایهٔ ورودی یکی است.

7.  **`Mat()`:** ماسک آرایه ورودی


## خروجی

| ![تصویر ورودی](/opencv-book/media/image69.png) |
| :--------------------------------------------: |
|                  تصویر ورودی                   |



| ![بافت‌نگار هر کدام از کانال‌های تصویر ورودی](/opencv-book/media/image70.png) |
| :----------------------------------------------------------: |
|           بافت‌نگار هر کدام از کانال‌های تصویر ورودی           |


