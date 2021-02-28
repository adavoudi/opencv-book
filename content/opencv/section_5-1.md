---
utid: 1000-05-01
chapter: 05
chaptername: فصل پنجم - شناسایی الگو
part: 01
title: مقایسه بافت‌نگارها
_index: compare-histograms
---

برای مقایسهٔ دو بافت‌نگار H1 و H2 باید ابتدا معیاری برای بیان میزان شباهت دو بافت‌نگار انتخاب کرد. در اُ سی وی برای مقایسهٔ بافت‌نگارها از تابع comapreHist استفاده می‌شود. این تابع سه معیار شباهت در اختیار ما قرار می‌دهد:

1. **همبستگی**[^a] **(CV\_COMP\_CORREL):**

   <div>
   $$d\left( H_{1},H_{2} \right) = \frac{\sum_{I}^{}{\left( H_{1}\left( I \right) - H_{1}^{'} \right)\left( H_{2}\left( I \right) - H_{2}^{'} \right)}}{\sqrt{\sum_{I}^{}{\left( H_{1}\left( I \right) - H_{1}^{'} \right)^{2}\ \sum_{I}^{}\left( H_{2}\left( I \right) - H_{2}^{'} \right)^{2}}}}$$
   </div>

   که

   <div>
   $$H_{k}^{'} = \frac{1}{N}\sum_{J}^{}{H_{k}(J)}$$
   </div>

   و N هم تعداد سطل‌های بافت‌نگار است.


2. **کای-اسکوئر**[^b] **(CV\_COMP\_CHISQR):**

   <div>
   $$d\left( H_{1},H_{2} \right) = \sum_{I}^{}\frac{\left( H_{1}\left( I \right) - H_{2}\left( I \right) \right)^{2}}{H_{1}\left( I \right)}$$
   </div>


3. **اشتراک**[^c] **(CV\_COMP\_INTERSECT):**

   $$d\left( H_{1},H_{2} \right) = \sum_{I}^{}{\min\left( H_{1}\left( I \right),\ H_{2}\left( I \right) \right)}$$


4. **فاصلهٔ باتاچاریا**[^d] **(CV\_COMP\_BHATTACHARYYA):**

   <div>
   $$d\left( H_{1},H_{2} \right) = \sqrt{1 - \frac{1}{\sqrt{H_{1}^{'}\ H_{2}^{'}\ N^{2}}}\ \sum_{I}^{}\sqrt{H_{1}\left( I \right)\text{.\ }H_{2}\left( I \right)}}$$
   </div>

[^a]: Correlation

[^b]: Chi-Square

[^c]: Intersection

[^d]: Bhattacharyya



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
    Mat src_base, hsv_base;
    Mat src_test1, hsv_test1;
    Mat src_test2, hsv_test2;
    Mat hsv_half_down;

    /// Load three images with different environment settings
    if( argc < 4 )
    {
    printf("*Error. Usage: ./compareHist_Demo <image_settings0> <image_setting1> <image_settings2>\n");
    return -1;
    }

    src_base = imread( argv[1], 1 );
    src_test1 = imread( argv[2], 1 );
    src_test2 = imread( argv[3], 1 );

    /// Convert to HSV
    cvtColor( src_base, hsv_base, COLOR_BGR2HSV );
    cvtColor( src_test1, hsv_test1, COLOR_BGR2HSV );
    cvtColor( src_test2, hsv_test2, COLOR_BGR2HSV );

    hsv_half_down = hsv_base( Range( hsv_base.rows/2, hsv_base.rows - 1 ), Range( 0, hsv_base.cols - 1 ) );

    /// Using 50 bins for hue and 60 for saturation
    int h_bins = 50; int s_bins = 60;
    int histSize[] = { h_bins, s_bins };

    // hue varies from 0 to 179, saturation from 0 to 255
    float h_ranges[] = { 0, 180 };
    float s_ranges[] = { 0, 256 };

    const float* ranges[] = { h_ranges, s_ranges };

    // Use the o-th and 1-st channels
    int channels[] = { 0, 1 };


    /// Histograms
    MatND hist_base;
    MatND hist_half_down;
    MatND hist_test1;
    MatND hist_test2;

    /// Calculate the histograms for the HSV images
    calcHist( &hsv_base, 1, channels, Mat(), hist_base, 2, histSize, ranges, true, false );
    normalize( hist_base, hist_base, 0, 1, NORM_MINMAX, -1, Mat() );

    calcHist( &hsv_half_down, 1, channels, Mat(), hist_half_down, 2, histSize, ranges, true, false );
    normalize( hist_half_down, hist_half_down, 0, 1, NORM_MINMAX, -1, Mat() );

    calcHist( &hsv_test1, 1, channels, Mat(), hist_test1, 2, histSize, ranges, true, false );
    normalize( hist_test1, hist_test1, 0, 1, NORM_MINMAX, -1, Mat() );

    calcHist( &hsv_test2, 1, channels, Mat(), hist_test2, 2, histSize, ranges, true, false );
    normalize( hist_test2, hist_test2, 0, 1, NORM_MINMAX, -1, Mat() );

    /// Apply the histogram comparison methods
    for( int i = 0; i < 4; i++ )
    {
        int compare_method = i;
        double base_base = compareHist( hist_base, hist_base, compare_method );
        double base_half = compareHist( hist_base, hist_half_down, compare_method );
        double base_test1 = compareHist( hist_base, hist_test1, compare_method );
        double base_test2 = compareHist( hist_base, hist_test2, compare_method );

        printf( " Method [%d] Perfect, Base-Half, Base-Test(1), Base-Test(2) : %f, %f, %f, %f \n",
               i, base_base, base_half , base_test1, base_test2 );
    }

    printf( "Done \n" );

    return 0;
}
```



## توضیح

در خطوط 31 تا 33، تصاویر پایه را به فرمت HSV تبدیل می‌کنیم.

در خط 35 یک ماتریس که به نصف پایینی تصویر پایه اشاره می‌کند تعریف می‌کنیم (در فرمت HSV).

در خطوط 37 تا 48، آرگومان‌های مورد نیاز برای حساب کردن بافت‌نگارها را مقدار دهی می‌کنیم.

در خطوط 58 تا 68، بافت‌نگار تصاویر HSV را حساب و نرمال می‌کنیم.

در خطوط 71 تا 81، به ترتیب چهار بار تابع مقایسه را بین بافت‌نگار تصویر پایه و بافت‌نگار تصاویر دیگر اعمال می‌کنیم.



## خروجی

سه تصویر زیر را به عنوان ورودی به کار می‌بریم:

| ![تصاویر ورودی به برنامه-تصویر سمت چپ، تصویر پایه است](/opencv-book/media/image124.png) |
| :----------------------------------------------------------: |
|    تصاویر ورودی به برنامه - تصویر سمت چپ، تصویر پایه است     |

تصویر سمت چپ را به عنوان تصویر پایه و دو تصویر دیگر را به عنوان تصاویر آزمایشی در نظر می‌گیریم. همچنین توجه کنید که تصویر پایه هم نسبت به خودش و هم نسبت به نصفهٔ پایینی‌اش، مقایسه می‌شود.

وقتی بافت‌نگار تصویر پایه را با خودش مقایسه می‌کنیم انتظار جواب کامل داریم. همچنین هنگامی که این بافت‌نگار را با بافت‌نگار نصفه پایین اش مقایسه می‌کنیم، از آنجا که هر دو تصویر از یک منبع هستند، جواب باید بیانگر میزان زیادی از شباهت باشد. به خاطر اینکه شرایط نور در دو تصویر آزمایشی با تصویر پایه متفاوت است، نتیجه مقایسهٔ بافت‌نگار آن‌ها با بافت‌نگار تصویر پایه نباید خیلی خوب باشد.

خروجی برنامه به صورت زیر است:

| BASE-TEST2 | BASE-TEST1 | BASE-HALF | BASE-BASE | Method        |
| :--------: | :--------: | :-------: | :-------: | ------------- |
|  0.12044   |  0.18207   |  0.93076  | 1.000000  | CORRELATION   |
|  49.2734   |  21.1845   |  4.94046  | 0.000000  | CHI-SQUARE    |
|  5.77508   |  3.88902   |  14.9598  | 24.39154  | INTERSECTION  |
|  0.80186   |  0.64657   |  0.22260  | 0.000000  | BHATTACHARYYA |

در روش‌های همبستگی و اشتراکی هر چه نتیجه بزرگتر باشد، شباهت بیشتر است. همانطور که انتظار داشتیم نتیجهٔ مقایسهٔ پایه با خودش، بزرگ‌ترین مقدار را دارد. همچنین نتیجهٔ مقایسه بافت‌نگار پایه با بافت‌نگار تصویر نصفه، دومین بزرگترین مقدار است. در دو روش دیگر، هر چه نتیجه کوچکتر باشد شباهت بیشتر است.