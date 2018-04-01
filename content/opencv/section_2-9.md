---
utid: 1000-02-09
chapter: 02
chaptername: فصل دوم - حوزهٔ مکان
part: 09
title: مولد عدد تصادفی و ترسیم متن
_index: drawing
---

در بخش ***شکل‌های هندسی ساده***، با دادن پارامترهایی مثل مختصات، رنگ، نازکی و... به توابع هندسی، شکل‌های متفاوتی کشیدیم. در آنجا مقادیر ثابتی را به عنوان پارامتر به آن توابع ارسال می‌کردیم. در این بخش می‌خواهیم از مقادیر تصادفی برای آن پارامترها استفاده کنیم و همچنین عکس مورد نظرمان را با تعداد زیادی شکل هندسی پر کنیم.

برای تولید اعداد تصادفی از کلاس RNG استفاده خواهیم کرد.



## کد

```c++
#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>
#include <iostream>
#include <stdio.h>

using namespace cv;

/// Global Variables
const int NUMBER = 100;
const int DELAY = 5;

const int window_width = 900;
const int window_height = 600;
int x_1 = -window_width/2;
int x_2 = window_width*3/2;
int y_1 = -window_width/2;
int y_2 = window_width*3/2;

/// Function headers
static Scalar randomColor( RNG& rng );
int Drawing_Random_Lines( Mat image, char* window_name, RNG rng );
int Drawing_Random_Rectangles( Mat image, char* window_name, RNG rng );
int Drawing_Random_Ellipses( Mat image, char* window_name, RNG rng );
int Drawing_Random_Polylines( Mat image, char* window_name, RNG rng );
int Drawing_Random_Filled_Polygons( Mat image, char* window_name, RNG rng );
int Drawing_Random_Circles( Mat image, char* window_name, RNG rng );
int Displaying_Random_Text( Mat image, char* window_name, RNG rng );
int Displaying_Big_End( Mat image, char* window_name, RNG rng );


/**
 * @function main
 */
int main( void )
{
  int c;

  /// Start creating a window
  char window_name[] = "Drawing_2 Tutorial";

  /// Also create a random object (RNG)
  RNG rng( 0xFFFFFFFF );

  /// Initialize a matrix filled with zeros
  Mat image = Mat::zeros( window_height, window_width, CV_8UC3 );
  /// Show it in a window during DELAY ms
  imshow( window_name, image );
  waitKey( DELAY );

  /// Now, let's draw some lines
  c = Drawing_Random_Lines(image, window_name, rng);
  if( c != 0 ) return 0;

  /// Go on drawing, this time nice rectangles
  c = Drawing_Random_Rectangles(image, window_name, rng);
  if( c != 0 ) return 0;

  /// Draw some ellipses
  c = Drawing_Random_Ellipses( image, window_name, rng );
  if( c != 0 ) return 0;

  /// Now some polylines
  c = Drawing_Random_Polylines( image, window_name, rng );
  if( c != 0 ) return 0;

  /// Draw filled polygons
  c = Drawing_Random_Filled_Polygons( image, window_name, rng );
  if( c != 0 ) return 0;

  /// Draw circles
  c = Drawing_Random_Circles( image, window_name, rng );
  if( c != 0 ) return 0;

  /// Display text in random positions
  c = Displaying_Random_Text( image, window_name, rng );
  if( c != 0 ) return 0;

  /// Displaying the big end!
  c = Displaying_Big_End( image, window_name, rng );
  if( c != 0 ) return 0;

  waitKey(0);
  return 0;
}

/// Function definitions

/**
 * @function randomColor
 * @brief Produces a random color given a random object
 */
static Scalar randomColor( RNG& rng )
{
  int icolor = (unsigned) rng;
  return Scalar( icolor&255, (icolor>>8)&255, (icolor>>16)&255 );
}


/**
 * @function Drawing_Random_Lines
 */
int Drawing_Random_Lines( Mat image, char* window_name, RNG rng )
{
  Point pt1, pt2;

  for( int i = 0; i < NUMBER; i++ )
  {
    pt1.x = rng.uniform( x_1, x_2 );
    pt1.y = rng.uniform( y_1, y_2 );
    pt2.x = rng.uniform( x_1, x_2 );
    pt2.y = rng.uniform( y_1, y_2 );

    line( image, pt1, pt2, randomColor(rng), rng.uniform(1, 10), 8 );
    imshow( window_name, image );
    if( waitKey( DELAY ) >= 0 )
      { return -1; }
  }

  return 0;
}

/**
 * @function Drawing_Rectangles
 */
int Drawing_Random_Rectangles( Mat image, char* window_name, RNG rng )
{
  Point pt1, pt2;
  int lineType = 8;
  int thickness = rng.uniform( -3, 10 );

  for( int i = 0; i < NUMBER; i++ )
  {
    pt1.x = rng.uniform( x_1, x_2 );
    pt1.y = rng.uniform( y_1, y_2 );
    pt2.x = rng.uniform( x_1, x_2 );
    pt2.y = rng.uniform( y_1, y_2 );

    rectangle( image, pt1, pt2, randomColor(rng), MAX( thickness, -1 ), lineType );

    imshow( window_name, image );
    if( waitKey( DELAY ) >= 0 )
      { return -1; }
  }

  return 0;
}

/**
 * @function Drawing_Random_Ellipses
 */
int Drawing_Random_Ellipses( Mat image, char* window_name, RNG rng )
{
  int lineType = 8;

  for ( int i = 0; i < NUMBER; i++ )
  {
    Point center;
    center.x = rng.uniform(x_1, x_2);
    center.y = rng.uniform(y_1, y_2);

    Size axes;
    axes.width = rng.uniform(0, 200);
    axes.height = rng.uniform(0, 200);

    double angle = rng.uniform(0, 180);

    ellipse( image, center, axes, angle, angle - 100, angle + 200,
             randomColor(rng), rng.uniform(-1,9), lineType );

    imshow( window_name, image );

    if( waitKey(DELAY) >= 0 )
      { return -1; }
  }

  return 0;
}

/**
 * @function Drawing_Random_Polylines
 */
int Drawing_Random_Polylines( Mat image, char* window_name, RNG rng )
{
  int lineType = 8;

  for( int i = 0; i< NUMBER; i++ )
  {
    Point pt[2][3];
    pt[0][0].x = rng.uniform(x_1, x_2);
    pt[0][0].y = rng.uniform(y_1, y_2);
    pt[0][1].x = rng.uniform(x_1, x_2);
    pt[0][1].y = rng.uniform(y_1, y_2);
    pt[0][2].x = rng.uniform(x_1, x_2);
    pt[0][2].y = rng.uniform(y_1, y_2);
    pt[1][0].x = rng.uniform(x_1, x_2);
    pt[1][0].y = rng.uniform(y_1, y_2);
    pt[1][1].x = rng.uniform(x_1, x_2);
    pt[1][1].y = rng.uniform(y_1, y_2);
    pt[1][2].x = rng.uniform(x_1, x_2);
    pt[1][2].y = rng.uniform(y_1, y_2);

    const Point* ppt[2] = {pt[0], pt[1]};
    int npt[] = {3, 3};

    polylines(image, ppt, npt, 2, true, randomColor(rng), rng.uniform(1,10), lineType);

    imshow( window_name, image );
    if( waitKey(DELAY) >= 0 )
      { return -1; }
  }
  return 0;
}

/**
 * @function Drawing_Random_Filled_Polygons
 */
int Drawing_Random_Filled_Polygons( Mat image, char* window_name, RNG rng )
{
  int lineType = 8;

  for ( int i = 0; i < NUMBER; i++ )
  {
    Point pt[2][3];
    pt[0][0].x = rng.uniform(x_1, x_2);
    pt[0][0].y = rng.uniform(y_1, y_2);
    pt[0][1].x = rng.uniform(x_1, x_2);
    pt[0][1].y = rng.uniform(y_1, y_2);
    pt[0][2].x = rng.uniform(x_1, x_2);
    pt[0][2].y = rng.uniform(y_1, y_2);
    pt[1][0].x = rng.uniform(x_1, x_2);
    pt[1][0].y = rng.uniform(y_1, y_2);
    pt[1][1].x = rng.uniform(x_1, x_2);
    pt[1][1].y = rng.uniform(y_1, y_2);
    pt[1][2].x = rng.uniform(x_1, x_2);
    pt[1][2].y = rng.uniform(y_1, y_2);

    const Point* ppt[2] = {pt[0], pt[1]};
    int npt[] = {3, 3};

    fillPoly( image, ppt, npt, 2, randomColor(rng), lineType );

    imshow( window_name, image );
    if( waitKey(DELAY) >= 0 )
       { return -1; }
  }
  return 0;
}

/**
 * @function Drawing_Random_Circles
 */
int Drawing_Random_Circles( Mat image, char* window_name, RNG rng )
{
  int lineType = 8;

  for (int i = 0; i < NUMBER; i++)
  {
    Point center;
    center.x = rng.uniform(x_1, x_2);
    center.y = rng.uniform(y_1, y_2);

    circle( image, center, rng.uniform(0, 300), randomColor(rng),
            rng.uniform(-1, 9), lineType );

    imshow( window_name, image );
    if( waitKey(DELAY) >= 0 )
      { return -1; }
  }

  return 0;
}

/**
 * @function Displaying_Random_Text
 */
int Displaying_Random_Text( Mat image, char* window_name, RNG rng )
{
  int lineType = 8;

  for ( int i = 1; i < NUMBER; i++ )
  {
    Point org;
    org.x = rng.uniform(x_1, x_2);
    org.y = rng.uniform(y_1, y_2);

    putText( image, "Testing text rendering", org, rng.uniform(0,8),
             rng.uniform(0,100)*0.05+0.1, randomColor(rng), rng.uniform(1, 10), lineType);

    imshow( window_name, image );
    if( waitKey(DELAY) >= 0 )
      { return -1; }
  }

  return 0;
}

/**
 * @function Displaying_Big_End
 */
int Displaying_Big_End( Mat image, char* window_name, RNG )
{
  Size textsize = getTextSize("OpenCV forever!", FONT_HERSHEY_COMPLEX, 3, 5, 0);
  Point org((window_width - textsize.width)/2, (window_height - textsize.height)/2);
  int lineType = 8;

  Mat image2;

  for( int i = 0; i < 255; i += 2 )
  {
    image2 = image - Scalar::all(i);
    putText( image2, "OpenCV forever!", org, FONT_HERSHEY_COMPLEX, 3,
             Scalar(i, i, 255), 5, lineType );

    imshow( window_name, image2 );
    if( waitKey(DELAY) >= 0 )
       { return -1; }
  }

  return 0;
}
```



## توضیح

از تابع main شروع می‌کنیم. اولین کاری که در این تابع انجام داده‌ایم، ساخت یک شیء RNG است (خط 42). کلاس RNG یک سازندهٔ عدد تصادفی است. در این مثال rng یک شیء از نوع RNG است که با 0xFFFFFFFF مقداردهی اولیه شده است.

سپس یک ماتریس با مقدار اولیهٔ صفر (که یعنی به رنگ سیاه است) و با طول، عرض و نوع CV_8UC3 درست می‌کنیم (خط 45).

در ادامه شکل‌های تصادفی را رسم می‌کنیم. اگر به خطوط 51 تا 80 نگاه کنید خواهید دید که این خطوط به هشت قسمت، که در واقع هشت تابع هستند، تقسیم شده‌اند. پیاده سازی این توابع تقریباً یکسان است، پس فقط سه تا از این توابع را بررسی می‌کنیم:

-   **تابع Drawing\_Random\_Lines**

    در این تابع:

  -     حلقهٔ for (خطوط 106 تا 114) به تعداد NUMBER تکرار می‌شود. از آنجایی که تابع line در این حلقه قرار دارد، پس یعنی به تعداد NUMBER، خط رسم خواهد شد.

  -      دو سَرِ خط با نقاط pt1 و pt2 مشخص می‌شوند. مثلاً pt1 به شکل زیر مقدار دهی می‌شود:

        ```c++
        pt1.x = rng.uniform( x_1, x_2 );
        pt1.y = rng.uniform( y_1, y_2 );
        ```
    
    -       می دانیم که rng یک شیء RNG است. در کد بالا تابع `rng.uniform(a,b)` را صدا می‌زنیم. این تابع یک عدد تصادفی با توزیع یکنواخت بین مقادیر a و b تولید می‌کند (شامل a و فاقد b).
    -       از توضیحات بالا متوجه می‌شویم که رأس‌های pt1 و pt2 کاملاً تصادفی خواهند بود؛ پس موقعیت خط‌ها غیر قابل پیش بینی می‌شود و این یک جلوهٔ بصری زیبا تولید می‌کند.

  -     توجه کنیدکه در تابع line به جای رنگ، تابع `randomColor(rng)` را قرار داده‌ایم. این تابع به شکل زیر پیاده سازی شده است:

        ```c++
        static Scalar randomColor( RNG& rng )
        {
        int icolor = (unsigned) rng;
        return Scalar( icolor&255, (icolor>>8)&255, (icolor>>16)&255 );
        }
        ```

        همان طور که می‌بینید، خروجی این تابع یک Scalar است که با سه مقدار تصادفی پر شده و این سه مقدار نشان دهندهٔ R، G و B هستند. پس رنگ خط‌ها هم تصادفی خواهد بود!

    توضیحات بالا برای توابع دیگر که دایره، مستطیل، چند ضلعی و... تولید می‌کنند یکسان است. فقط پارامترهای دیگر مثل مرکز دایره یا بیضی و رأس‌های مستطیل و چند ضلعی نیز تصادفی خواهند بود.

-   **تابع Display\_Random\_Text**

    در این تابع همه چیز آشنا به نظر می‌رسد، اما عبارت زیر جدید است:

    ```c++
    putText( image, "Testing text rendering", org, rng.uniform(0,8),
            rng.uniform(0,100)*0.05+0.1, randomColor(rng),
            rng.uniform(1, 10), lineType );
    ```

    این تابع چه کاری انجام می‌دهد؟! در مورد این مثال:

    -   متن "Testing text rendering" را روی عکس image رسم می‌کند.
    -   گوشهٔ پایین و چپ متن در نقطهٔ org قرار دارد.
    -   نوع فونت، یک عدد تصادفی در بازهٔ 0 تا 8 است.
    -   مقیاس فونت با عبارت `rng.uniform(0, 100) * 0.05 + 0.1` که یک عدد تصادفی بین 0.1 تا 5.1 است، مشخص شده.
    -   رنگ متن تصادفی است (با تابع `randomColor(rng)` به دست می‌آید).
    -   ضخامت متن بین 0 تا 10 است که با تابع `rng.uniform(1, 10)` مشخص می‌شود.

        در نتیجه به تعداد NUMBER، متن روی عکسمان خواهیم داشت که در مکان‌های تصادفی قرار گرفته‌اند.

-   **تابع Display\_Big\_End**

    به جز تابع getTextSize (که اندازهٔ متن داده شده را بر می‌گرداند)، تنها قطعه کد زیر جدید است:

    ```c++
    image2 = image - Scalar::all(i);
    ```

    یعنی image2 برابر است با تفریق `Scalar::all(i)` از image. در حقیقت مقدار هر پیکسل image2 برابر است با تفریق مقدار i از image (به یاد داشته باشید که هر پیکسل متشکل از سه مقدار R، G و B است، پس این عمل روی همهٔ آنها تأثیر می‌گذارد).

    همچنین به یاد داشته باشید که عمل تفریق خودش به صورت خودکار عمل saturate_cast را انجام می‌دهد تا همیشه مقادیر معتبر (که در این مثال بین 0 تا 255 است) را تولید کند.



## خروجی

روند اجرای برنامه به صورت زیر است:

1.   ابتدا به تعداد NUMBER خط تصادفی روی صفحه نشان داده می‌شود:

    ![mage4](/opencv-book/media/image45.png)

2.  سپس تعدادی مستطیل تصادفی نشان داده می‌شود.

3.  حالا تعدادی بیضی با مختصات، اندازه، نازکی و اندازهٔ قوس تصادفی کشیده می‌شود.

    ![mage4](/opencv-book/media/image46.png)

4.  بعد تعدادی چند ضلعی تو خالی کشیده می‌شود:

    ![mage4](/opencv-book/media/image47.png)

5.  سپس چند ضلعی‌های توپُر.

6.  و آخرین شکل هندسی، دایره:

    ![mage4](/opencv-book/media/image48.png)

7.  بعد از آن، متن "Testing text rendering" چندین بار با فونت، اندازه، رنگ و مختصات متفاوت نشان داده می‌شود.

8.  در نهایت عبارت OpenCV For Ever روی تصویر به تنهایی نمایش داده می‌شود.

    ![mage4](/opencv-book/media/image49.png)

   ​

