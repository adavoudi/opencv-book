---
utid: 1000-02-03
chapter: 02
chaptername: فصل دوم - حوزهٔ مکان
part: 03
title: فیلتر خطی
_index: linear-filter
---

در این بخش نحوه اعمال فیلتر خطی روی تصاویر را بررسی می‌کنیم. برای اعمال فیلتر خطی باید یک کرنل داشته باشیم؛ سپس آن کرنل را در تصویر کانوالو می‌کنیم.

به طور کلی، به عملی که بین یک کرنل و همهٔ قسمت‌های یک تصویر انجام می‌شود کانولوشن می گویند. همچنین کرنل آرایه‌ای از ضرایب ثابت است و یک لنگر دارد که معمولاً در مرکز آرایه قرار دارد. لنگر همان درایه‌ای از کرنل است که روی پیکسل مورد بررسی قرار می‌گیرد.

![mage1](/opencv-book/media/image18.png)

یک کرنل نمونه که لنگر در مرکز آن قرار دارد.

برای هر پیکسل (x,y) در تصویر I، اگر K ماتریس کرنل با ابعاد $M*N$ و $(a,b)$ *مختصات لنگر در کرنل باشد، آنگاه* $H\left( x,y \right)$*، یعنی مقدار پیکسل* $\left( x,y \right)$ در تصویر فیلتر شده، به صورت زیر حساب می شود:

$$H\left( x,y \right) = \ \sum_{i = 0}^{M - 1}{\sum_{j = 0}^{N - 1}{I\left( x + i - a,y + j - b \right)K(i,j)}}$$

برای اعمال فیلتر خطی لازم نیست خودمان فرمول بالا را پیاده سازی کنیم چون تابع filter2D در اُ سی وی دقیقاً همین کار را انجام می‌دهد.



## کد

```c++
#include "opencv2/imgproc/imgproc.hpp"
#include "opencv2/highgui/highgui.hpp"

using namespace cv;

/** @function main */
int main ( int argc, char** argv )
{
  /// Declare variables
  Mat src, dst;

  Mat kernel;
  Point anchor;
  double delta;
  int ddepth;
  int kernel_size;
  char* window_name = "filter2D Demo";

  int c;

  /// Load an image
  src = imread( argv[1] );

  if( !src.data )
  { return -1; }

  /// Create window
  namedWindow( window_name, CV_WINDOW_AUTOSIZE );

  /// Initialize arguments for the filter
  anchor = Point( -1, -1 );
  delta = 0;
  ddepth = -1;

  /// Loop - Will filter the image with different kernel sizes each 0.5 seconds
  int ind = 0;
  while( true )
    {
      c = waitKey(500);
      /// Press 'ESC' to exit the program
      if( (char)c == 27 )
        { break; }

      /// Update kernel size for a normalized box filter
      kernel_size = 3 + 2*( ind%5 );
      kernel = Mat::ones( kernel_size, kernel_size, CV_32F )/ (float)(kernel_size*kernel_size);

      /// Apply filter
      filter2D(src, dst, ddepth , kernel, anchor, delta, BORDER_DEFAULT );
      imshow( window_name, dst );
      ind++;
    }

  return 0;
}
```



## توضیح

خطوط 39 تا 51 در یک حلقه بینهایت قرار دارند. در این حلقه اندازهٔ کرنل آپدیت و فیلتر خطی به عکس اعمال می‌شود. جزئیات اعمالی که در این حلقه صورت می‌گیرد به صورت زیر است:

ابتدا در خطوط 45 و 46 کرنل تعریف می‌شود. خط 45 برای آپدیت کردن مقدار kernel_size است؛ این مقدار عددی بین 3 و 11 است. در خط 46 برای ساخت کرنل ابتدا یک ماتریس مربعی با ابعاد kernel_size * kernel_size ساخته می‌شود و سپس آن ماتریس را بر تعداد درایه‌هایش (یعنی kernel_size * kernel_size) تقسیم می‌کند.

بعد از درست کردن کرنل، در خط 49 برای اعمال کرنل ساخته شده به تصویر از تابع filter2D استفاده می‌شود. این تابع 7 آرگومان به شرح زیر دارد:

-   src: تصویر ورودی

-   dst: تصویر خروجی (تصویر فیلتر شده)

-   ddepth: عمق تصویر خروجی است. یک مقدار منفی (مثل 1-) بیان می‌کند که عمق dst باید مشابه عمق src باشد.

-   kernel: کرنل مورد استفاده است.

-   anchor: موقعیت لنگر کرنل است. با قرار دادن `Point(-1,-1)` مقدار پیش فرض (یعنی مختصات مرکز کرنل) استفاده می‌شود.

-   delta: مقداری که نتیجه کانولوشن هر پیکسل اضافه می‌شود. به صورت پیش‌فرض صفر است.

-   BORDER\_DEFAULT: نحوه کانوال کردن پیکسل‌های کناری تصویر را مشخص می‌کند.




## خروجی

برنامه را به صورت زیر اجرا می‌کنیم:

```txt
2DFilter.exe "PICTURES /2.jpg"
```

هر 500 میلی ثانیه اندازهٔ کرنل تغییر می‌کند. نمونه‌ای از اجرای برنامه را در زیر می‌بینید:

| ![اندازهٔ کرنل ۳](/opencv-book/media/image19.jpeg) | ![اندازهٔ کرنل ۵](/opencv-book/media/image20.jpeg) |
| :-----------------------------------------------: | :-----------------------------------------------: |
|                   اندازهٔ کرنل ۳                   |                   اندازهٔ کرنل ۵                   |


| ![اندازهٔ کرنل ۷](/opencv-book/media/image21.jpeg) | ![اندازهٔ کرنل ۹](/opencv-book/media/image22.jpeg) |
| :-----------------------------------------------: | :-----------------------------------------------: |
|                   اندازهٔ کرنل ۷                   |                   اندازهٔ کرنل ۹                   |


| ![اندازهٔ کرنل ۱۱](/opencv-book/media/image23.jpeg) |
| :------------------------------------------------: |
|                   اندازهٔ کرنل ۱۱                   |


