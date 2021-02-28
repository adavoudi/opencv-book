---
utid: 1000-04-02
chapter: 04
chaptername: فصل چهارم - مورفولوژی
part: 02
title: تبدیل‌های مورفولوژی پیچیده
_index: morphology-transformations
---

در این بخش به بررسی نحوه اعمال عملیات‌های مورفولوژی پیچیده مثل باز کردن[^ a]، بستن[^ b]، گرادیان به روش مورفولوژی، تاپ هَت[^ c]و بلک هَت[^ d] می‌پردازیم. تمام این اعمال بر پایه عملیات فرسایش و گشایش به وجود آمده‌اند. در ادامه توضیح مختصری در مورد هر کدام از این اعمال می‌دهیم.

**باز کردن:**

از طریقِ گشایش گرفتن از فرسایش یافتهٔ یک تصویر به دست می‌آید:

$$dst = open\left( src,\ element \right) = dilate\left(erode\left( src,\ element \right) \right)$$

این عمل برای از بین بردن اشیای کوچک مناسب است (بافرض اینکه اشیا به رنگ روشن و روی زمینهٔ تیره قرار داشته باشند). به شکل زیر نگاه کنید. تصویر سمت چپ تصویر ورودی و تصویر سمت راست نتیجهٔ عملیات باز کردن است. همانطور که می‌بینید ناحیه‌های کوچک بین حرف ناپدید شده‌اند.

| ![نتیجه اعمال باز کردن روی تصویر سمت چپ](/opencv-book/media/image117.png) |
| :----------------------------------------------------------: |
|            نتیجه اعمال باز کردن روی تصویر سمت چپ             |

**بستن:**

از طریق فرسایش گرفتن از گشایش یافتهٔ یک تصویر به دست می‌آید:

$$dst = close\left( src,\ element \right) = erode\left( dilate\left( src,\ element \right) \right)$$

این عمل برای از بین بردن حفره‌های کوچک مناسب است (نواحی تیره).

| ![نتیجه اعمال بستن روی تصویر سمت چپ](/opencv-book/media/image118.png) |
| :----------------------------------------------------------: |
|              نتیجه اعمال بستن روی تصویر سمت چپ               |

**گرادیان به روش مورفولوژی:**

از اختلاف بین گشایش و فرسایش یک تصویر به دست می‌آید:

$$dst = morph_{\text{grad}}\left( src,element \right) = dilate\left( src,\ element \right) - erode(src,\ element)$$

این عمل برای پیدا کردن محیط یک شیء مناسب است.

| ![نتیجه اعمال گرادیان به روش مورفولوژی روی تصویر سمت چپ](/opencv-book/media/image119.png) |
| :----------------------------------------------------------: |
|    نتیجه اعمال گرادیان به روش مورفولوژی روی تصویر سمت چپ     |

**تاپ هَت:**

از اختلاف بین یک تصویر با باز شدهٔ آن به دست می‌آید:

$$dst = tophat\left( src,\ element \right) = src - open(src,\ element)$$

| ![نتیجه اعمال تاپ هَت روی تصویر سمت چپ](/opencv-book/media/image120.png) |
| :----------------------------------------------------------: |
|             نتیجه اعمال تاپ هَت روی تصویر سمت چپ              |

**بلک هَت:**

از اختلاف بین بسته شدهٔ یک تصویر با خودش به دست می‌آید:

$$dst = blackhat\left( src,\ element \right) = close\left( src,\ element \right) - src$$

| ![نتیجه اعمال بلک هَت روی تصویر سمت چپ](/opencv-book/media/image121.png) |
| :----------------------------------------------------------: |
|             نتیجه اعمال بلک هَت روی تصویر سمت چپ              |



[^a]: Opening

[^b]: Closing

[^c]: Top Hat

[^d]: Black Hat



## کد

```c++
#include "opencv2/imgproc/imgproc.hpp"
#include "opencv2/highgui/highgui.hpp"
#include <stdlib.h>
#include <stdio.h>

using namespace cv;

/// Global variables
Mat src, dst;

int morph_elem = 0;
int morph_size = 0;
int morph_operator = 0;
int const max_operator = 4;
int const max_elem = 2;
int const max_kernel_size = 21;

char* window_name = "Morphology Transformations Demo";

/** Function Headers */
void Morphology_Operations( int, void* );

/** @function main */
int main( int argc, char** argv )
{
  /// Load an image
  src = imread( argv[1] );

  if( !src.data )
  { return -1; }

 /// Create window
 namedWindow( window_name, CV_WINDOW_AUTOSIZE );

 /// Create Trackbar to select Morphology operation
 createTrackbar("Operator:\n 0: Opening - 1: Closing \n 2: Gradient - 3: Top Hat \n 4: Black Hat",
                window_name, &morph_operator, max_operator, Morphology_Operations );

 /// Create Trackbar to select kernel type
 createTrackbar( "Element:\n 0: Rect - 1: Cross - 2: Ellipse", window_name,
                 &morph_elem, max_elem,
                 Morphology_Operations );

 /// Create Trackbar to choose kernel size
 createTrackbar( "Kernel size:\n 2n +1", window_name,
                 &morph_size, max_kernel_size,
                 Morphology_Operations );

 /// Default start
 Morphology_Operations( 0, 0 );

 waitKey(0);
 return 0;
 }

 /**
  * @function Morphology_Operations
  */
void Morphology_Operations( int, void* )
{
  // Since MORPH_X : 2,3,4,5 and 6
  int operation = morph_operator + 2;

  Mat element = getStructuringElement( morph_elem, Size( 2*morph_size + 1, 2*morph_size+1 ),
                                      Point( morph_size, morph_size ) );

  /// Apply the specified morphology operation
  morphologyEx( src, dst, operation, element );
  imshow( window_name, dst );
}
```



## توضیح

در خط 68 از تابع MorphologyEx برای انجام عملیات مورفولوژی روی تصویر ورودی استفاده می‌شود. این تابع چهار آرگومان دارد:

- src: تصویر ورودی

- dst: تصویر خروجی

- operation: نوع تبدیل مورفولوژی‌ای که باید اعمال شود. از پنج مقدار زیر می‌توان استفاده کرد:

  -   بازکردن: MORPH\_OPEN: 2

  -   بستن: MORPH\_CLOSE: 3

  -   گرادیان: MORPH\_GRADIENT: 4

  -   تاپ هَت: MORPH\_TOPHAT: 5

  -   بلک هَت: MORPH\_BLACKHAT: 6

  به خاطر اینکه مقدارها بین دو و شش است، در خط 62 دو واحد به مقدار گرفته شده از ترک‌بار اضافه می‌کنیم.

- element: کرنل مورد استفاده است. از تابع getStructuringElement برای ساخت آن استفاده شده است.


## خروجی

| ![تصویر ورودی](/opencv-book/media/image122.png) |
| :---------------------------------------------: |
|                   تصویر ورودی                   |



| ![نمایی از برنامه – سمت راست عمل بلک هَت و سمت چپ عمل باز کردن](/opencv-book/media/image123.png) |
| :----------------------------------------------------------: |
| نمایی از برنامه – سمت راست عمل بلک هَت و سمت چپ عمل باز کردن  |

