---
utid: 1000-05-03
chapter: 05
chaptername: فصل پنجم - شناسایی الگو
part: 03
title: تطبیق الگو
_index: pattern-matching
---

تطبیق الگو تکنیکی برای پیدا کردن ناحیه‌هایی از یک تصویر است که مطابق (مشابه) با یک تصویر الگو (تکه تصویر) هستند.

برای انجام این عمل به دو عنصر اولیه نیاز است:

1.  تصویر منبع (I): تصویری که می‌خواهیم یک تطبیق در آن پیدا کنیم.

2.  تصویر الگو (T): تکه تصویری که با تصویر منبع مقایسه می‌شود.

هدف پیدا کردن بزرگترین ناحیهٔ تطبیق است.

| ![دو عنصر مورد نیاز برای انجام یک تطبیق الگو](/opencv-book/media/image129.png) |
| :----------------------------------------------------------: |
|          دو عنصر مورد نیاز برای انجام یک تطبیق الگو          |

برای پیدا کردن ناحیهٔ تطبیق باید تصویر الگو را روی تصویر منبع حرکت دهیم و هر قسمت از تصویر منبع را با تصویر الگو مقایسه کنیم.

| ![روند پیدا کردن یک تطبیق در تصویر منبع](/opencv-book/media/image130.png) |
| ------------------------------------------------------------ |
| روند پیدا کردن یک تطبیق در تصویر منبع                        |

منظور از حرکت دادن این است که تکه تصویر را هر بار یک پیکسل حرکت دهیم (از چپ به راست و از بالا به پایین). به هر حال، در هر توقف یک مقدار حساب می‌شود که مشخص می‌کند تطبیق در آن محل چقدر خوب یا بد بوده است (به زبان دیگر بیان می‌کند تکه تصویر چقدر شبیه به آن ناحیه است).

به ازای هر محل T روی I، مقدار حساب شده را در ماتریس R (ماتریس نتیجه) ذخیره می‌کنیم. هر محل (x,y) در R حاوی مقدار حساب شده برای آن محل است.

| ![مثالی از ماتریس R](/opencv-book/media/image131.png) |
| ----------------------------------------------------- |
| مثالی از ماتریس R                                     |

تصویر بالا ماتریس R است که با حرکت دادن الگو روی تصویر ورودی با معیار TM_CCORR_NORMED به دست آمده است. هر چه یک محل روشن‌تر باشد به این معنی است که شباهتش بیشتر است. همانطور که می‌بینید محلی که با دایرهٔ قرمز مشخص شده، احتمالاً همان محلی است که بیشترین مقدار را دارد؛ پس آن محل (یعنی مستطیلی که گوشه بالا و سمت چپ آن روی این نقطه قرار دارد و طول و عرضش هم اندازه با طول و عرض الگو است) را به عنوان ناحیه تطبیق در نظر می‌گیریم.

در عمل برای پیدا کردن محل عنصر بیشنه (یا کمینه، بسته به نوع معیار استفاده شده،) در ماتریس R، از تابع minMaxLoc استفاده می‌کنیم.

اُ سی وی عملیات تطبیق الگو را در تابع matchTemplate پیاده سازی کرده است. روش‌های موجود به صورت زیر هستند:

1. **CV\_TM\_SQDIFF**

   $$R\left( x,\ y \right) = \sum_{x^{'},y^{'}}^{}\left( T\left( x^{'},\ y^{'} \right) - I\left( x + x^{'},\ y + y^{'} \right) \right)^{2}$$


2. **CV\_TM\_SQDIFF\_NORMED**

   $$R\left( x,\ y \right) = \frac{\sum_{x^{'},\ y^{'}}^{}\left( T\left( x^{'},\ y^{'} \right) - I\left( x + x^{'},\ y + y^{'} \right) \right)^{2}}{\sqrt{\sum_{x^{'},\ y^{'}}^{}{{T\left( x^{'},\ y^{'} \right)}^{2}\ }.\sum_{x^{'},\ y^{'}}^{}{I\left( x + x^{'},\ y + y^{'} \right)}^{2}}}$$


3. **CV\_TM\_CCORR**

   $$R\left( x,\ y \right) = \sum_{x^{'},y^{'}}^{}\left( T\left( x^{'},\ y^{'} \right).I\left( x + x^{'},\ y + y^{'} \right) \right)$$


4. **CV\_TM\_CCORR\_NORMED**

   $$R\left( x,\ y \right) = \frac{\sum_{x^{'},\ y^{'}}^{}{T\left( x^{'},\ y^{'} \right).I^{'}\left( x + x^{'},\ y + y^{'} \right)}}{\sqrt{\sum_{x^{'},\ y^{'}}^{}{{T\left( x^{'},\ y^{'} \right)}^{2}\ }.\sum_{x^{'},\ y^{'}}^{}{I\left( x + x^{'},\ y + y^{'} \right)}^{2}}}$$


5. **CV\_TM\_CCOEFF**

   $$R\left( x,\ y \right) = \sum_{x^{'},y^{'}}^{}\left( T^{'}\left( x^{'},\ y^{'} \right)\text{\ .\ \ I}\left( x + x^{'},\ y + y^{'} \right) \right)$$

   که

   $$T^{'}\left( x^{'},\ y^{'} \right) = T\left( x^{'},\ y^{'} \right) - \frac{1}{w. h}.\sum_{x^{''},y^{''}}^{}{T\left( x^{''},\ y^{''} \right)}$$

   $$I^{'}\left( x + x^{'},\ y + y^{'} \right) = I\left( x + x^{'},\ y + y^{'} \right) - \frac{1}{w.h}.\sum_{x^{''},y^{''}}^{}{I\left( x + x^{''},\ y + y^{''} \right)}$$


6. **CV\_TM\_CCOEFF\_NORMED**

   $$R\left( x,\ y \right) = \frac{\sum_{x^{'},\ y^{'}}^{}\left( T^{'}\left( x^{'},\ y^{'} \right).I^{'}\left( x + x^{'},\ y + y^{'} \right) \right)}{\sqrt{\sum_{x^{'},\ y^{'}}^{}{{T^{'}\left( x^{'},\ y^{'} \right)}^{2}\ }.\sum_{x^{'},\ y^{'}}^{}{I^{'}\left( x + x^{'},\ y + y^{'} \right)}^{2}}}$$



## کد

```c++
#include "opencv2/highgui/highgui.hpp"
#include "opencv2/imgproc/imgproc.hpp"
#include <iostream>
#include <stdio.h>

using namespace std;
using namespace cv;

/// Global Variables
Mat img; Mat templ; Mat result;
char* image_window = "Source Image";
char* result_window = "Result window";

int match_method;
int max_Trackbar = 5;

/// Function Headers
void MatchingMethod( int, void* );

/** @function main */
int main( int argc, char** argv )
{
    /// Load image and template
    img = imread( argv[1], 1 );
    templ = imread( argv[2], 1 );

    /// Create windows
    namedWindow( image_window, CV_WINDOW_AUTOSIZE );
    namedWindow( result_window, CV_WINDOW_AUTOSIZE );

    /// Create Trackbar
    char* trackbar_label = "Method: \n 0: SQDIFF \n 1: SQDIFF NORMED \n 2: TM CCORR \n \
                            3: TM CCORR NORMED \n 4: TM COEFF \n 5: TM COEFF NORMED";
    createTrackbar( trackbar_label, image_window, &match_method, max_Trackbar, MatchingMethod );

    MatchingMethod( 0, 0 );

    waitKey(0);
    return 0;
}

/**
 * @function MatchingMethod
 * @brief Trackbar callback
 */
void MatchingMethod( int, void* )
{
    /// Source image to display
    Mat img_display;
    img.copyTo( img_display );

    /// Create the result matrix
    int result_cols =  img.cols - templ.cols + 1;
    int result_rows = img.rows - templ.rows + 1;

    result.create( result_cols, result_rows, CV_32FC1 );

    /// Do the Matching and Normalize
    matchTemplate( img, templ, result, match_method );
    normalize( result, result, 0, 1, NORM_MINMAX, -1, Mat() );

    /// Localizing the best match with minMaxLoc
    double minVal; double maxVal; Point minLoc; Point maxLoc;
    Point matchLoc;

    minMaxLoc( result, &minVal, &maxVal, &minLoc, &maxLoc, Mat() );

    /// For SQDIFF and SQDIFF_NORMED, the best matches are lower values.
    /// For all the other methods, the higher the better
    if( match_method  == CV_TM_SQDIFF || match_method == CV_TM_SQDIFF_NORMED )
    { matchLoc = minLoc; }
    else
    { matchLoc = maxLoc; }

    /// Show me what you got
    rectangle( img_display, matchLoc, Point( matchLoc.x + templ.cols , matchLoc.y + templ.rows ), 
              Scalar::all(0), 2, 8, 0 );
    rectangle( result, matchLoc, Point( matchLoc.x + templ.cols , matchLoc.y + templ.rows ), 
              Scalar::all(0), 2, 8, 0 );

    imshow( image_window, img_display );
    imshow( result_window, result );

    return;
}
```



## توضیح

در خط 59 با استفاده از تابع matchTemplate عملیات تطبیق الگو را اجرا می‌کنیم. آرگومان‌های این تابع تصویر ورودی (I)، الگو (T)، ماتریس نتیجه (R) و نوع تطبیق (که از ترک‌بار به دست می‌آید) هستند.

در خط 60 نتیجه را نرمال می‌کنیم.

در خط 66، با استفاده از تابع minMaxLoc محل مقادیر بیشنیه و کمینهٔ ماتریس R را پیدا می‌کنیم. این تابع 6 آرگومان دارد که به شرح زیر هستند:

1.  **result:** آرایهٔ ورودی (آرایه‌ای که مورد جستجو قرار می‌گیرد).
2.  **`&minVal` و `&maxVal`:** متغیرهایی برای ذخیرهٔ مقادیر کمینه و بیشنهٔ موجود در result.
3.  **`&minLoc` و `&maxLoc`:** محل مقادیر بیشینه و کمینه در این نقاط قرار می‌گیرند.
4.  **`Mat()`:** ماسک آرایه ورودی.

در دو روش تطبیق CV_SQDIFF و CV_SQDIFF_NORMED، بهترین تطبیق‌ها دارای کمترین مقادیر هستند در حالی که در دیگر روش‌ها بهترین تطبیق‌ها دارای بیشترین مقادیر هستند. پس برای سادگی، در خطوط 70 تا 73 بسته به روش مورد استفاده، متغیر matchLoc را مقدار دهی می‌کنیم.



## خروجی



| ![تصویر ورودی](/opencv-book/media/image132.png) | ![تصویر الگو](/opencv-book/media/image133.png) |
| ----------------------------------------------- | ---------------------------------------------- |
| تصویر ورودی                                     | تصویر الگو                                     |

ماتریس‌های نتیجه زیر تولید می‌شوند (ردیف اول روش‌های SQDIFF، CCORR و CCOEFF و ردیف دوم هم همین روش‌ها، ولی نسخهٔ نرمال شدهٔ آنها هستند). در ستون اول، تاریک‌ترین محل، بهترین تطبیق است و در دو ستون دیگر روشن‌ترین محل، بهترین تطبیق است.

| ![ماتریس‌های نتیجه تولید شده توسط فرایند تطبیق الگو](/opencv-book/media/image134.png) |
| :----------------------------------------------------------: |
|       ماتریس‌های نتیجه تولید شده توسط فرایند تطبیق الگو       |

نتیجه نهایی در شکل زیر نشان داده شده است. دقت کنید که CCORR و CCDEFF جواب نادرستی را به عنوان پاسخ داده‌اند؛ البته نسخه نرمال شده آنها درست عمل کرده است. این ممکن است به خاطر این واقعیت باشد که ما فقط اولین بزرگترین تطبیق را در نظر می‌گیریم و به تطبیق‌های بزرگ دیگر کاری نداریم.

| ![نتیجه نهایی](/opencv-book/media/image135.png) |
| :---------------------------------------------: |
|                   نتیجه نهایی                   |





