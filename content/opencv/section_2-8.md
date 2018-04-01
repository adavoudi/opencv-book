---
utid: 1000-02-08
chapter: 02
chaptername: فصل دوم - حوزهٔ مکان
part: 08
title: اضافه کردن قاب به تصویر
---

در بخش قبل با کانولوشن و نحوهٔ اعمال آن روی یک تصویر آشنا شدیم. مسئله بر سر پیکسل‌های کناری تصویر بود. چگونه می‌توان این پیکسل‌ها را کانوال کرد در حالی که لنگر کرنل روی آنها قرار دارد و قسمتی از کرنل خارج از تصویر افتاده است؟!

بیشتر توابع اُ سی وی برای حل این مشکل تصویر را در یک ماتریس بزرگتر کپی می‌کنند و سپس به طور خودکار اطراف تصویر را پر می‌کنند. به این طریق می‌توان بدون هیچ مشکلی کانولوشن را روی تمام پیکسل‌های مورد نیاز اعمال کرد و سپس در انتها پیکسل‌های اضافه را پاک کرد.

در این بخش دو روش اضافه کردن قاب به تصویر بیان خواهد شد:

1.  **BORDER\_CONSTANT:** قاب اطراف تصویر را با یک مقدار ثابت پُر می‌کند (مثلاً با عدد 0).
2.  **BORDER\_REPLICATE:** سطرها و ستون‌های آخر تصویر اصلی را در قسمت قاب کپی می‌کند.



## کد

```c++
#include "opencv2/imgproc/imgproc.hpp"
#include "opencv2/highgui/highgui.hpp"
#include <stdlib.h>
#include <stdio.h>

using namespace cv;

/// Global Variables
Mat src, dst;
int top, bottom, left, right;
int borderType;
Scalar value;
char* window_name = "copyMakeBorder Demo";
RNG rng(12345);

/** @function main  */
int main( int argc, char** argv )
{

  int c;

  /// Load an image
  src = imread( argv[1] );

  if( !src.data )
  { return -1;
    printf(" No data entered, please enter the path to an image file \n");
  }

  /// Brief how-to for this program
  printf( "\n \t copyMakeBorder Demo: \n" );
  printf( "\t -------------------- \n" );
  printf( " ** Press 'c' to set the border to a random constant value \n");
  printf( " ** Press 'r' to set the border to be replicated \n");
  printf( " ** Press 'ESC' to exit the program \n");

  /// Create window
  namedWindow( window_name, CV_WINDOW_AUTOSIZE );

  /// Initialize arguments for the filter
  top = (int) (0.05*src.rows); bottom = (int) (0.05*src.rows);
  left = (int) (0.05*src.cols); right = (int) (0.05*src.cols);
  dst = src;

  imshow( window_name, dst );

  while( true )
    {
      c = waitKey(500);

      if( (char)c == 27 )
        { break; }
      else if( (char)c == 'c' )
        { borderType = BORDER_CONSTANT; }
      else if( (char)c == 'r' )
        { borderType = BORDER_REPLICATE; }

      value = Scalar( rng.uniform(0, 255), rng.uniform(0, 255), rng.uniform(0, 255) );
      copyMakeBorder( src, dst, top, bottom, left, right, borderType, value );

      imshow( window_name, dst );
    }

  return 0;
}
```



## توضیح

در خطوط 41 و 42 متغیرهایی که اندازهٔ قاب را مشخص می‌کنند (بالا، پایین، چپ و راست) تعریف شده است. در اینجا مقدارشان پنج درصدِ اندازهٔ تصویر src قرار داده شده‌اند.

در خط 47 برنامه وارد یک حلقهٔ while می‌شود. با فشردن کلید "c" یا "r"، متغیر borderType به ترتیب یکی از مقادیر BORDER_CONSTANT یا BORDER_REPLICATE را می‌گیرد.

در هر تکرار (هر 500 میلی ثانیه)، متغیر value در خط 58 به روز می‌شود. این مقدار یک عدد تصادفی است که توسط rng و در بازهٔ 0 تا 255 تولید می‌شود.

در انتها در خط 59 برای اضافه کردن قاب، تابع copyMakeBorder صدا زده می‌شود. آرگومان‌های این تابع به شرح زیر هستند:

-   src: تصویر منبع
-   dst: تصویر مقصد
-   top, bottom, left, right: طول قاب‌های هر طرف تصویر در مقیاس پیکسل هستند.
-   borderType: نوع قاب را تعیین می‌کند. می‌تواند ثابت یا تکرار شونده باشد.
-   value: اگر مقدار borderType، BORDER\_CONSTANT باشد، value همان مقدار ثابتی است که در قاب قرار می‌گیرد.



## خروجی

برنامه را به صورت زیر اجرا می‌کنیم:

```powershell
CopyMakeBorder.exe "PICTURES /2.jpg"
```


| ![تصویر ورودی](/opencv-book/media/image42.jpeg) | ![قاب ثابت (بعد از فشردن کلید c)](/opencv-book/media/image43.jpeg) |
| :---------------------------------------------: | :----------------------------------------------------------: |
|                   تصویر ورودی                   |                قاب ثابت (بعد از فشردن کلید c)                |


| ![قاب تکرار شونده (بعد از فشردن کلید r)](/opencv-book/media/image44.jpeg) |
| :----------------------------------------------------------: |
|            قاب تکرار شونده (بعد از فشردن کلید r)             |



