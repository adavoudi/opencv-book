---
utid: 1000-02-16
chapter: 02
chaptername: فصل دوم - حوزهٔ مکان
part: 16
title: عملگر کَنی
_index: canny-operator
---

لبه‌یاب کَنی در سال 1986 توسط جان اِف. کَنی[^ a] توسعه یافت. همچنین خیلی‌ها آن را با نام **یابندهٔ بهینه**[^ b] می‌شناسند. الگوریتم کَنی سه هدف اساسی زیر را دنبال می‌کند:

1.  **نرخ خطای پایین:** یعنی اینکه فقط لبه‌های واقعی را کشف کند (نه نقاطی که لبه نیستند).

2.  **مکان‌یابی خوب:** فاصلهٔ میان پیکسل‌های کشف شدهٔ لبه و پیکسل‌های واقعی لبه باید کمینه باشد.

3.  **پاسخ کمینه:** به ازای هر لبه فقط یک پاسخ وجود داشته باشد.

رویه حساب لبه‌ها در لبه‌یاب کَنی به صورت زیر است:

1.  نویزها را فیلتر می‌کند. بدین منظور از فیلتر گوسی استفاده می‌شود. به عنوان مثال می‌توان از کرنل زیر استفاده کرد:

    ![mage8](/opencv-book/media/image87.png)

2.  گرادیان تابع روشنایی تصویر را به دست می‌آورد. بدین منظور از روشی مشابه با عملگر سابل استفاده می‌شود:

    i. دو کرنل زیر را در تصویر کانوال می‌کند (یکی برای جهت x و یکی برای جهت y):
   
    <span>
    $$
    G_{x} = \begin{bmatrix}
        - 1 & 0 & + 1 \\\\
        - 2 & 0 & + 2 \\\\
        - 1 & 0 & + 1
    \end{bmatrix}
    $$
    </span>
    
    <span>
    $$
    G_{y} = \begin{bmatrix}
        - 1 & - 2 & - 1 \\\\
        0 & 0 & 0 \\\\
        + 1 & + 2 & + 1
    \end{bmatrix}
    $$
    </span>

    ii. از روش زیر، دامنه و جهت آن را پیدا می‌کند:

    <div>
    $$G = \sqrt{G_{x}^{2} + G_{y}^{2}}$$
    </div>

    $$\theta = \arctan\left( \frac{G_{y}}{G_{x}} \right)$$

    جهت‌ها در یکی از چهار گروه 0، 45، 90 یا 135 قرار می‌گیرند.

3.  فیلتری جهت حذف نقاط غیر بیشینه اعمال می‌کند. با این کار پیکسل‌هایی که جزئی از لبه نیستند، حذف می‌شوند. بنابراین فقط لبه‌های نازک باقی می‌مانند.

4.  در قدم آخر کَنی از یک روش آستانه گذاری با دو آستانه بالا و پایین استفاده می‌کند:

    I.  اگر مقدار پیکسلی از آستانهٔ بالایی بیشتر باشد، به عنوان لبه پذیرفته می‌شود.

    II. اگر مقدار پیکسلی از آستانهٔ پایینی کمتر باشد، رد می‌شود.

    III. اگر مقدار پیکسلی بین آستانهٔ بالایی و آستانهٔ پایینی بود، فقط در صورتی به عنوان لبه پذیرفته می‌شود که همسایهٔ پیکسلی باشد که مقدارش بالاتر از آستانهٔ بالایی است.

    پیشنهاد می‌شود که نسبت آستانهٔ بالایی: پایینی یکی از دو مقدار 1:2 یا 1:3 باشد.

[^a]: John F. Canny

[^b]: Optimal detector



## کد

```c++
#include "opencv2/imgproc/imgproc.hpp"
#include "opencv2/highgui/highgui.hpp"
#include <stdlib.h>
#include <stdio.h>

using namespace cv;

/// Global variables

Mat src, src_gray;
Mat dst, detected_edges;

int edgeThresh = 1;
int lowThreshold;
int const max_lowThreshold = 100;
int ratio = 3;
int kernel_size = 3;
char* window_name = "Edge Map";

/**
 * @function CannyThreshold
 * @brief Trackbar callback - Canny thresholds input with a ratio 1:3
 */
void CannyThreshold(int, void*)
{
    /// Reduce noise with a kernel 3x3
    blur( src_gray, detected_edges, Size(3,3) );

    /// Canny detector
    Canny( detected_edges, detected_edges, lowThreshold, lowThreshold*ratio, kernel_size );

    /// Using Canny's output as a mask, we display our result
    dst = Scalar::all(0);

    src.copyTo( dst, detected_edges);
    imshow( window_name, dst );
}


/** @function main */
int main( int argc, char** argv )
{
    /// Load an image
    src = imread( argv[1] );

    if( !src.data )
    { return -1; }

    /// Create a matrix of the same type and size as src (for dst)
    dst.create( src.size(), src.type() );

    /// Convert the image to grayscale
    cvtColor( src, src_gray, CV_BGR2GRAY );

    /// Create a window
    namedWindow( window_name, CV_WINDOW_AUTOSIZE );

    /// Create a Trackbar for user to enter threshold
    createTrackbar( "Min Threshold:", window_name, &lowThreshold,
        max_lowThreshold, CannyThreshold );

    /// Show the image
    CannyThreshold(0, 0);

    /// Wait until user exit program by pressing a key
    waitKey(0);

    return 0;
}
```



## توضیح

به نکات زیر توجه کنید:

1.  در خط 16 نسبت آستانهٔ پایین: بالا را 3:1 انتخاب کرده‌ایم.

2.  در خط 17 اندازهٔ کرنل را 3 می‌گذاریم (این کرنل برای عملیات سابلی است که در تابع Canny استفاده می‌شود).

3.  در خط 15 بیشنه مقدار مجاز برای آستانهٔ پایینی را 100 می‌گذاریم.

در تابع CannyThreshold در خط 27، برای کاهش نویز تصویر ورودی را هموار می‌کنیم.

سپس در خط 30 تابع Canny را صدا می‌زنیم. این تابع 5 آرگومان به شرح زیر دارد:

1.  **detected\_edges:** تصویر ورودی است. این تصویر باید سیاه و سفید باشد.

2.  **detected\_edges:** تصویر خروجی است. می‌توان همان تصویر ورودی را به عنوان خروجی در نظر گرفت.

3.  **lowThreshold:** مقداری که کاربر به وسیلهٔ ترک‌بار مشخص کرده است.

4.  **highThreshold:** مقدار این متغیر سه برابر آستانهٔ پایینی است (یعنی سه برابر lowThreshold).

5.  **kernel\_size:** مقدارش را 3 می‌گذاریم (این اندازهٔ کرنل Sobel ای است که در تابع Canny استفاده می‌شود).

در نهایت در خط 35 از تابع copyTo برای نگاشت نواحی‌ای که به عنوان لبه کشف شده‌اند به یک پس زمینهٔ سیاه استفاده می‌کنیم. لازم به ذکر است که این تابع فقط مقادیری از عکس منبع را کپی می‌کند که صفر نباشند. از آنجایی که خروجی Canny به صورت کانتور های لبه روی یک زمینهٔ سیاه است، تمام نواحی dst به‌جز نواحی‌ای که لبه‌های کشف شده قرار داشته باشند، سیاه خواهند بود.



## خروجی

| ![تصویر ورودی](/opencv-book/media/image88.png) | ![تصویر خروجی با آستانه پایینی 100](/opencv-book/media/image89.png) |
| :--------------------------------------------: | :----------------------------------------------------------: |
|                  تصویر ورودی                   |               تصویر خروجی با آستانه پایینی 100               |

