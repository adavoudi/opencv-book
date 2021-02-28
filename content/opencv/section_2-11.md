---
utid: 1000-02-11
chapter: 02
chaptername: فصل دوم - حوزهٔ مکان
part: 11
title: آستانه‌گذاری
_index: threshold
---

در این بخش به بررسی چگونگی انجام عملیات آستانه‌گذاری تصویر به کمک تابع threshold می‌پردازیم.

آستانه‌گذاری ساده‌ترین روش گروه بندی است. از آستانه‌گذاری می‌توان جهت جدا سازی نواحی مختلف تصویرِ استفاده کرد. در اُ سی وی پنج نوع عمل آستانه‌گذاری وجود دارد.

برای ادامه این بخش فرض کنید یک تصویر مرجع با مقادیر روشنایی `src(x,y)` برای پیکسل‌های آن داریم. خط افقی آبی رنگ مقدار آستانه را نمایش می‌دهد.

| ![تصویر مرجع و خط آستانه](/opencv-book/media/image55.png) |
| :-------------------------------------------------------: |
|                  تصویر مرجع و خط آستانه                   |

**آستانه‌گذاری باینری**[^a]**:**

در این نوع به صورت زیر عمل می‌شود:

<span>
$$
dst \\left( x,y \\right) = \\left\\{ \\begin{matrix}
maxVal \\hspace{3em}  if \thickspace src\\left( x,y \\right) > thresh \\\\
0 \\hspace{10em} otherwise
\\end{matrix} \\right.
$$
</span>

یعنی اگر روشنایی پیکسل `src(x,y)` بیشتر از مقدار thresh بود، آنگاه مقدار آن پیکسل برابر با MaxVal قرار داده می‌شود. در غیر این صورت مقدار آن برابر صفر قرار داده می‌شود.

| ![نتیجه آستانه‌گذاری باینری](/opencv-book/media/image56.png) |
| :---------------------------------------------------------: |
|                  نتیجه آستانه‌گذاری باینری                   |

**آستانه‌گذاری باینری معکوس**[^b]**:**

در این نوع به صورت زیر عمل می‌شود:

<span>
$$
dst \\left( x,y \\right) = \\left\\{ \\begin{matrix}
0 \\hspace{3em} if \\thickspace src \\left( x,y \\right) > thresh \\\\
maxVal \\hspace{5em} otherwise
\\end{matrix} \\right.
$$
</span>

یعنی اگر روشنایی پیکسل `src(x,y)` بیشتر از thresh بود، آنگاه مقدار آن پیکسل برابر با صفر قرار داده می‌شود. در غیر این صورت مقدار آن برابر MaxVal خواهد بود.

| ![نتیجه آستانه‌گذاری باینری معکوس](/opencv-book/media/image57.png) |
| :----------------------------------------------------------: |
|                نتیجه آستانه‌گذاری باینری معکوس                |

**بریدن**[^c]**:**

در این نوع به صورت زیر عمل می‌شود:

<span>
$$
dst \\left( x,y \\right) = \\left\\{ \\begin{matrix}
threshold  \\hspace{3em}  if \\thickspace src \\left( x,y \\right) > thresh \\\\
src\\left( x,y \\right) \\hspace{8em} otherwise
\\end{matrix} \\right.
$$
</span>

بیشترین مقدار مجاز برای پیکسل‌ها، مقدار thresh است. اگر مقدار پیکسلی بیشتر از thresh بود، مقداری از آن بریده می‌شود.

| ![نتیجه بریدن](/opencv-book/media/image58.png) |
| :--------------------------------------------: |
|                  نتیجه بریدن                   |

**آستانه‌گذاری به صفر**[^d]**:**

در این نوع به صورت زیر عمل می‌شود:

<span>
$$
dst \\left( x,y \\right) = \\left\\{ \begin{matrix}
src(x,y) \\hspace{3em} if \\thickspace src \\left( x,y \\right) > thresh \\\\
0\hspace{10em} otherwise
\\end{matrix} \\right.
$$
</span>

اگر `src(x,y)` کوچک‌تر از thresh بود، مقدار آن صفر قرار داده خواهد شد.

| ![نتیجه آستانه‌گذاری به صفر](/opencv-book/media/image59.png) |
| :---------------------------------------------------------: |
|                  نتیجه آستانه‌گذاری به صفر                   |

**آستانه‌گذاری به صفر معکوس**[^e]**:**

در این نوع به صورت زیر عمل می‌شود:

<span>
$$
dst \\left( x,y \\right) = \\left\\{ \\begin{matrix}
0 \\hspace{3em} if \\thickspace src\\left( x,y \\right) > thresh \\\\
src(x,y) \\hspace{5em} otherwise
\\end{matrix} \\right.
$$
</span>

اگر `src(x,y)` بزرگ‌تر از thresh بود، مقدار آن را صفر می‌گذاریم.

| ![نتیجه آستانه‌گذاری به صفر معکوس](/opencv-book/media/image60.png) |
| :----------------------------------------------------------: |
|                نتیجه آستانه‌گذاری به صفر معکوس                |



[^a]: Threshold binary

[^b]: Threshold binary, inverted

[^c]: Truncate

[^d]: Threshold to zero

[^e]: Threshold to zero, inverted



## کد

```c++
#include "opencv2/imgproc/imgproc.hpp"
#include "opencv2/highgui/highgui.hpp"
#include <stdlib.h>
#include <stdio.h>

using namespace cv;

/// Global variables

int threshold_value = 0;
int threshold_type = 3;
int const max_value = 255;
int const max_type = 4;
int const max_BINARY_value = 255;

Mat src, src_gray, dst;
char* window_name = "Threshold Demo";

char* trackbar_type = "Type: \n"
    " 0: Binary \n"
    " 1: Binary Inverted \n"
    " 2: Truncate \n"
    " 3: To Zero \n"
    " 4: To Zero Inverted";
char* trackbar_value = "Value";

/// Function headers
void Threshold_Demo( int, void* );

/**
 * @function main
 */
int main( int argc, char** argv )
{
  /// Load an image
  src = imread( argv[1], 1 );

  /// Convert the image to Gray
  cvtColor( src, src_gray, CV_RGB2GRAY );

  /// Create a window to display results
  namedWindow( window_name, CV_WINDOW_AUTOSIZE );

  /// Create Trackbar to choose type of Threshold
  createTrackbar( trackbar_type,
                  window_name, &threshold_type,
                  max_type, Threshold_Demo );

  createTrackbar( trackbar_value,
                  window_name, &threshold_value,
                  max_value, Threshold_Demo );

  /// Call the function to initialize
  Threshold_Demo( 0, 0 );

  /// Wait until user finishes program
  while(true)
  {
    int c;
    c = waitKey( 20 );
    if( (char)c == 27 )
      { break; }
   }

}


/**
 * @function Threshold_Demo
 */
void Threshold_Demo( int, void* )
{
  /* 0: Binary
     1: Binary Inverted
     2: Threshold Truncated
     3: Threshold to Zero
     4: Threshold to Zero Inverted
   */

  threshold( src_gray, dst, threshold_value, max_BINARY_value,threshold_type );

  imshow( window_name, dst );
}
```



## توضیح

در خط 39 تصویر ورودی را با استفاده از تابع cvtColor به فضای رنگی سیاه و سفید می‌بریم.

در خطوط 45 و 49 دو تِرَک‌بار برای دریافت نوع آستانه‌گذاری و مقدار آستانه از کاربر درست می‌کنیم. هر وقت کاربر مقدار یکی از ترک‌بارها را تغییر دهد، تابع Threshold_Demo صدا زده می‌شود.

در خط 80 از تابع threshold استفاده شده است. این تابع دارای 5 آرگومان است:

-   src\_gray: تصویر ورودی
-   dst: تصویر خروجی
-   threshold\_value: مقدار آستانه است.
-   max\_BINARY\_value: مقداری که در آستانه‌گذاری باینری استفاده می‌شود.
-   threshold\_type: نوع عمل آستانه‌گذاری است. در خطوط 73 تا 77 این نوع‌ها آورده شده‌اند.


## خروجی

| ![تصویر ورودی](/opencv-book/media/image61.jpeg) | ![نتیجه آستانه‌گذاری صفر با آستانه 100](/opencv-book/media/image62.jpeg) |
| :---------------------------------------------: | :----------------------------------------------------------: |
|                   تصویر ورودی                   |             نتیجه آستانه‌گذاری صفر با آستانه 100              |



| ![نتیجه آستانه‌گذاری صفر معکوس با آستانه 100](/opencv-book/media/image63.jpeg) | ![نتیجه آستانه‌گذاری باینری با آستانه 100](/opencv-book/media/image64.jpeg) |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|          نتیجه آستانه‌گذاری صفر معکوس با آستانه 100           |            نتیجه آستانه‌گذاری باینری با آستانه 100            |



| ![نتیجه آستانه‌گذاری باینری معکوس با آستانه 100](/opencv-book/media/image65.jpeg) | ![نتیجه آستانه‌گذاری بریدن با آستانه 100](/opencv-book/media/image66.jpeg) |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|         نتیجه آستانه‌گذاری باینری معکوس با آستانه 100         |            نتیجه آستانه‌گذاری بریدن با آستانه 100             |

