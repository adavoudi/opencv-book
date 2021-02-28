---
utid: 1000-04-01
chapter: 04
chaptername: فصل چهارم - مورفولوژی
part: 01
title: فرسایش و گشایش
_index: erode-dilate
---

به طور کلی به دسته‌ای از عملیات که تصاویر را بر پایهٔ اشکال هندسی پردازش می‌کنند، عملیات مورفولوژی می گویند. عملیات مورفولوژی به تصویر ورودی یک عنصر سازنده[^ a] اعمال می‌کنند و یک تصویر خروجی تولید می‌کنند. در فرایند مورفولوژی میزان شباهت اشکال موجود در تصویر داده شده را با عنصر سازنده که خود یک شکل هندسی است، بررسی می‌شود.

ابتدایی‌ترین عملیات مورفولوژی، عملیات فرسایش[^ b] و گشایش[^ c] هستند که کاربردهای بسیاری دارند، از جمله:

-   از بین بردن نویز

-   جدا کردن عناصر خاص و متصل کردن قسمت‌های جدا از هم موجود در یک تصویر

-   پیدا کردن قسمت‌های بسیار تیره (حفره‌ها[^ d]) و بسیار روشن تصویر

در ادامه با در نظر گرفتن تصویر زیر، عملیات فرسایش و گشایش را بررسی می‌کنیم.

| ![تصویر ورودی](/opencv-book/media/image112.png) |
| :---------------------------------------------: |
|                   تصویر ورودی                   |

**گشایش:**

این عمل از کانولوشن تصویر A با کرنل B که می‌تواند هر اندازه و شکلی داشته باشد (که معمولاً دایره یا مربع است)، تشکیل شده است. کرنل B یک لنگر دارد که معمولاً در مرکز کرنل است.

همین‌طور که کرنل B روی تصویر حرکت می‌کند، پیکسل بیشینه‌ای که در پنجرهٔ B است را پیدا کرده و مقدار آن را به جای مقدار پیکسلی که لنگر روی آن قرار دارد می‌گذارد. پس این عمل باعث رشد نواحی روشن در تصویر می‌شود (نام گشایش از اینجا می‌آید). به عنوان مثال اگر عملگر گشایش را به تصویر ورودی بالا اعمال کنیم، نتیجه به صورت زیر خواهد بود:

![نتیجه اعمال گشایش روی تصویر ورودی](/opencv-book/media/image113.png)

نتیجه اعمال گشایش روی تصویر ورودی

----------

پس زمینهٔ تصویر (قسمت روشن) نسبت به ناحیهٔ تیره مربوط به حرف j، گشایش می‌یابد.

**فرسایش:**

همین‌طور که کرنل B روی تصویر حرکت می‌کند، پیکسل کمینه‌ای که در پنجرهٔ B است را پیدا کرده و مقدار آن را به جای مقدار پیکسلی که لنگر روی آن قرار دارد می‌گذارد.

عملگر فرسایش را روی تصویر ورودی اعمال می‌کنیم. شکل زیر نتیجهٔ این کار را نشان می‌دهد. در این تصویر می‌بینید که ناحیه‌های روشن تصویر (یعنی پس زمینه) نازک‌تر شده‌اند (یعنی فرسایش یافته‌اند)، در حالی که ناحیه‌های تیره بزرگتر شده‌اند.

| ![نتیجه اعمال فرسایش روی تصویر ورودی](/opencv-book/media/image114.png) |
| :----------------------------------------------------------: |
|              نتیجه اعمال فرسایش روی تصویر ورودی              |



[^a]: Structuring element

[^b]: Erode

[^c]: Dilate

[^d]: holes



## کد

```c++
#include "opencv2/imgproc/imgproc.hpp"
#include "opencv2/highgui/highgui.hpp"
#include <stdlib.h>
#include <stdio.h>

using namespace cv;

/// Global variables
Mat src, erosion_dst, dilation_dst;

int erosion_elem = 0;
int erosion_size = 0;
int dilation_elem = 0;
int dilation_size = 0;
int const max_elem = 2;
int const max_kernel_size = 21;

/** Function Headers */
void Erosion( int, void* );
void Dilation( int, void* );

/** @function main */
int main( int argc, char** argv )
{
  /// Load an image
  src = imread( argv[1] );

  if( !src.data )
  { return -1; }

  /// Create windows
  namedWindow( "Erosion Demo", CV_WINDOW_AUTOSIZE );
  namedWindow( "Dilation Demo", CV_WINDOW_AUTOSIZE );
  cvMoveWindow( "Dilation Demo", src.cols, 0 );

  /// Create Erosion Trackbar
  createTrackbar( "Element:\n 0: Rect \n 1: Cross \n 2: Ellipse", "Erosion Demo",
                  &erosion_elem, max_elem,
                  Erosion );

  createTrackbar( "Kernel size:\n 2n +1", "Erosion Demo",
                  &erosion_size, max_kernel_size,
                  Erosion );

  /// Create Dilation Trackbar
  createTrackbar( "Element:\n 0: Rect \n 1: Cross \n 2: Ellipse", "Dilation Demo",
                  &dilation_elem, max_elem,
                  Dilation );

  createTrackbar( "Kernel size:\n 2n +1", "Dilation Demo",
                  &dilation_size, max_kernel_size,
                  Dilation );

  /// Default start
  Erosion( 0, 0 );
  Dilation( 0, 0 );

  waitKey(0);
  return 0;
}

/**  @function Erosion  */
void Erosion( int, void* )
{
  int erosion_type;
  if( erosion_elem == 0 ){ erosion_type = MORPH_RECT; }
  else if( erosion_elem == 1 ){ erosion_type = MORPH_CROSS; }
  else if( erosion_elem == 2) { erosion_type = MORPH_ELLIPSE; }

  Mat element = getStructuringElement( erosion_type,
                                       Size( 2*erosion_size + 1, 2*erosion_size+1 ),
                                       Point( erosion_size, erosion_size ) );

  /// Apply the erosion operation
  erode( src, erosion_dst, element );
  imshow( "Erosion Demo", erosion_dst );
}

/** @function Dilation */
void Dilation( int, void* )
{
  int dilation_type;
  if( dilation_elem == 0 ){ dilation_type = MORPH_RECT; }
  else if( dilation_elem == 1 ){ dilation_type = MORPH_CROSS; }
  else if( dilation_elem == 2) { dilation_type = MORPH_ELLIPSE; }

  Mat element = getStructuringElement( dilation_type,
                                       Size( 2*dilation_size + 1, 2*dilation_size+1 ),
                                       Point( dilation_size, dilation_size ) );
  /// Apply the dilation operation
  dilate( src, dilation_dst, element );
  imshow( "Dilation Demo", dilation_dst );
}
```



## توضیح

**تابع Erosion (خط 63):**

تابعی که عمل گشایش را انجام می‌دهد، تابع erode است (خط 75). این تابع سه آرگومان دریافت می‌کند:

-   src: تصویر ورودی

-   erosion\_dst: تصویر خروجی

-   element: کرنلی که برای عمل گشایش از آن استفاده خواهد شد. اگر آن را تعیین نکنیم به صورت پیشفرض یک ماتریس 3 در 3 استفاده می‌شود. در غیر این صورت برای مشخص کردن شکل آن از تابع getStructuringElement استفاده می‌شود. می‌توان یکی از سه شکل زیر را به عنوان کرنل انتخاب کرد:


-   مستطیل: MORPH\_RECT

-   صلیب: MORPH\_CROSS

-   بیضی: MORPH\_ELLIPSE

سپس فقط باید اندازه و مختصات لنگر را انتخاب کرد. اگر مختصات لنگر مشخص نشود، به صورت پیش‌فرض نقطهٔ مرکزی کرنل به عنوان محل لنگر در نظر گرفته می‌شود.

**تابع Dilation (خط 80):**

کد این تابع بسیار مشابه با کد تابع Erosion است. اینجا هم باید نوع کرنل، مختصات لنگر و اندازهٔ کرنل را مشخص کرد.



## خروجی

| ![تصویر ورودی](/opencv-book/media/image115.png) |
| :---------------------------------------------: |
|                   تصویر ورودی                   |



| ![نمایی از برنامه – سمت راست عمل گشایش و سمت چپ عمل فرسایش](/opencv-book/media/image116.png) |
| :----------------------------------------------------------: |
|   نمایی از برنامه – سمت راست عمل گشایش و سمت چپ عمل فرسایش   |

