---
utid: 1000-01-04
chapter: 01
chaptername: فصل اول - معرفی
part: 04
title: ذخیره سازی و بازیابی اطلاعات
_index: store-retrieve
---

در این بخش می‌خواهیم شما را با شیوه استاندارد ذخیره سازی و بازیابی داده ساختارهای موجود در اُ سی وی آشنا کنیم. البته با این روش می‌توان داده ساختارهایی که به نوعی از داده ساختارهای اُ سی وی درست شده‌اند را نیز ذخیره/بازیابی کرد.



## کد

```c++
#include <opencv2/core/core.hpp>
#include <iostream>
#include <string>

using namespace cv;
using namespace std;

static void help(char** av)
{
    cout << endl
        << av[0] << " shows the usage of the OpenCV serialization functionality." << endl << "usage: " << endl
        <<  av[0] << " outputfile.yml.gz" << endl
        << "The output file may be either XML (xml) or YAML (yml/yaml). You can even compress it by "
        << "specifying this in its extension like xml.gz yaml.gz etc... " << endl
        << "With FileStorage you can serialize objects in OpenCV by using the << and >> operators" << endl
        << "For example: - create a class and have it serialized"                         << endl
        << "             - use it to read and write matrices."                            << endl;
}

class MyData
{
public:
    MyData() : A(0), X(0), id()
    {}
    explicit MyData(int) : A(97), X(CV_PI), id("mydata1234") // explicit to avoid implicit conversion
    {}
    void write(FileStorage& fs) const                        //Write serialization for this class
    {
        fs << "{" << "A" << A << "X" << X << "id" << id << "}";
    }
    void read(const FileNode& node)                          //Read serialization for this class
    {
        A = (int)node["A"];
        X = (double)node["X"];
        id = (string)node["id"];
    }
public:   // Data Members
    int A;
    double X;
    string id;
};

//These write and read functions must be defined for the serialization in FileStorage to work
static void write(FileStorage& fs, const std::string&, const MyData& x)
{
    x.write(fs);
}
static void read(const FileNode& node, MyData& x, const MyData& default_value = MyData()){
    if(node.empty())
        x = default_value;
    else
        x.read(node);
}

// This function will print our custom class to the console
static ostream& operator<<(ostream& out, const MyData& m)
{
    out << "{ id = " << m.id << ", ";
    out << "X = " << m.X << ", ";
    out << "A = " << m.A << "}";
    return out;
}

int main(int ac, char** av)
{
    if (ac != 2)
    {
        help(av);
        return 1;
    }

    string filename = av[1];
    { //write
        Mat R = Mat_<uchar>::eye(3, 3),
            T = Mat_<double>::zeros(3, 1);
        MyData m(1);

        FileStorage fs(filename, FileStorage::WRITE);

        fs << "iterationNr" << 100;
        fs << "strings" << "["; // text - string sequence
        fs << "image1.jpg" << "Awesomeness" << "baboon.jpg";
        fs << "]"; // close sequence

        fs << "Mapping"; // text - mapping
        fs << "{" << "One" << 1;
        fs <<        "Two" << 2 << "}";

        fs << "R" << R; // cv::Mat
        fs << "T" << T;

        fs << "MyData" << m; // your own data structures

        fs.release(); // explicit close
        cout << "Write Done." << endl;
    }

    {//read
        cout << endl << "Reading: " << endl;
        FileStorage fs;
        fs.open(filename, FileStorage::READ);

        int itNr;
        //fs["iterationNr"] >> itNr;
        itNr = (int) fs["iterationNr"];
        cout << itNr;
        if (!fs.isOpened())
        {
            cerr << "Failed to open " << filename << endl;
            help(av);
            return 1;
        }

        FileNode n = fs["strings"]; // Read string sequence - Get node
        if (n.type() != FileNode::SEQ)
        {
            cerr << "strings is not a sequence! FAIL" << endl;
            return 1;
        }

        FileNodeIterator it = n.begin(), it_end = n.end(); // Go through the node
        for (; it != it_end; ++it)
            cout << (string)*it << endl;


        n = fs["Mapping"]; // Read mappings from a sequence
        cout << "Two  " << (int)(n["Two"]) << "; ";
        cout << "One  " << (int)(n["One"]) << endl << endl;


        MyData m;
        Mat R, T;

        fs["R"] >> R; // Read cv::Mat
        fs["T"] >> T;
        fs["MyData"] >> m; // Read your own structure_

        cout << endl
            << "R = " << R << endl;
        cout << "T = " << T << endl << endl;
        cout << "MyData = " << endl << m << endl << endl;

        //Show default behavior for non existing nodes
        cout << "Attempt to read NonExisting (should initialize the data structure with its default).";
        fs["NonExisting"] >> m;
        cout << endl << "NonExisting = " << endl << m << endl;
    }

    cout << endl
        << "Tip: Open up " << filename << " with a text editor to see the serialized data." << endl;

    return 0;
}
```



## 	توضیح

در این بخش فقط در مورد فایل‌های XML و YAML صحبت می‌کنیم. فایل ورودی/خروجی شما ممکن است فقط یکی از این دو فرمت را داشته باشد.

به طور کلی دو نوع داده ساختار قابل سریال سازی[^32] وجود دارد:

-   نگاشت‌ها[^33] (مثل map در STL)
-   دنبالهٔ عناصر[^34] (مثل vector در STL).

تفاوت این دو در این است که در نگاشت‌ها هر عنصر یک کلید یکتا دارد و می‌توان در صورت وجود مستقیماً به آن دسترسی پیدا کرد، ولی در دنباله برای پیدا کردن یک عنصر باید کل دنباله را جستجو کرد.

در ادامه با نحوه ذخیره سازی و بازیابی هر دو نوع داده ساختار بالا را بررسی می‌کنیم.

1.  **باز/بسته کردن فایل:** قبل از اینکه در یک فایل بنویسید، لازم است آن را باز کرده و سپس در انتها آن را به بنید. در اُ سی وی برای کار با فایل‌های XML و YAML از کلاس FileStorage استفاده می‌شود. برای باز کردن یک فایل با استفاده از این کلاس می‌توان از سازنده آن یا تابع open آن استفاده کرد:

    ```c++
    string filname = "I.xml";
    FileStorage fs(filename, FileStorage::READ);
    // OR ...
    FileStorage fs;
    fs.open(filename, FileStorage::READ);
    ```

    آرگومان دوم یک ثابت است که مشخص کنندهٔ عملیاتی است که می‌خواهید روی آن فایل انجام دهید: WRITE (نوشتن)، READ(خواندن) یا APPEND (اضافه کردن). همچنین پسوندی که در اسم فایل مشخص شده است، تعیین کنندهٔ نوع فایل خروجی است. جالب اینکه با اضافه کردن .gz به انتهای اسم فایل، آن فایل فشرده خواهد شد.

    وقتی یک شیء FileStorage از بین برود، فایل مربوط به آن به صورت خودکار بسته می‌شود. البته می‌توان این کار را به صورت دستی با تابع release انجام داد:

    ```c++
    fs.release();
    ```

2.  **ورودی/خروجی عددی و متنی:** کلاس FileStotage مشابه کتابخانهٔ STL برای خروجی از عملگر \<\< استفاده می‌کند. برای چاپ هر ساختار داده‌ای لازم است که ابتدا نام آن مشخص شود. این کار به سادگی با چاپ کردن اسم آن انجام می‌شود. برای داده‌های اولیه می‌توان به روش زیر، هم اسم و هم مقدار آن را چاپ کرد:

    ```c++
    fs << "iterationNr" << 100;
    ```

    برای خواندن مقدار iterationNr باید با استفاده از عملگر [] نام گره‌ای که قصد خواندن آن را داریم (که در اینجا iterationNr است)، مشخص کنیم. سپس می‌توان به یکی از دو روش زیر به مقدار آن دسترسی پیدا کرد:

    ```c++
    int itNr;
    fs["iterationNr"] >> itNr;
    // OR ...
    itNr = (int) fs["iterationNr"];
    ```

3.  **ورودی/خروجی ساختار داده‌های اُ سی وی:** این کار دقیقاً مشابه نوشتن نوع‌های اولیه است:

    ```c++
    Mat R = Mat_<uchar>::eye(3, 3),
    T = Mat_<double>::zeros(3, 1);
           
    fs << "R" << R; // Write cv::Mat
    fs << "T" << T;

    fs["R"] >> R; // Read cv::Mat
    fs["T"] >> T;
    ```

4.  **ورودی/خروجی آرایه‌های ساده و انجمنی:** همان‌طور که قبلاً گفته شد، می‌توان نگاشت‌ها و دنباله‌ها (آرایه‌ها و بردارها) را با استفاده از کلاس FileStorage چاپ کرد. روند چاپ اینگونه داده ساختارها به این صورت است که ابتدا نام متغیر را می‌آوریم و سپس مشخص می‌کنیم که خروجی یک دنباله یا یک نگاشت است.

    برای دنباله‌ها، قبل از آیتم اول کاراکتر [ و بعد از آیتم آخر نیز کاراکتر ] را قرار می‌دهیم:

    ```c++
    fs << "strings" << "[";                // text - string sequence
    fs << "image1.jpg" << "Awesomeness" << "baboon.jpg";
    fs << "]"; // close sequence
    ```

    در مورد نگاشت‌ها هم همین کار را می‌کنیم، با این تفاوت که از کاراکترهای { و } استفاده می‌کنیم:

    ```c++
    fs << "Mapping"; // text - mapping
    fs << "{" << "One" << 1;
    fs <<        "Two" << 2 << "}";
    ```

    برای خواندن هم از ساختار داده‌های FileNode و FileNodeIterator استفاده می‌کنیم. عملگر [] مربوط به کلاس FileStotage یک شیء FileNode را بر می‌گرداند. اگر نود بازگشتی یک دنباله بود، از FileNodeIterator برای خواندن آیتم‌ها استفاده می‌کنیم:

    ```c++
    FileNode n = fs["strings"]; // Read string sequence - Get node
    if (n.type() != FileNode::SEQ)
    {
        cerr << "strings is not a sequence! FAIL" << endl;
        return 1;
    }

    FileNodeIterator it = n.begin(), it_end = n.end(); // Go through the node
    for (; it != it_end; ++it)
        cout << (string)*it << endl;
    ```

    در مورد نگاشت‌ها می‌توانیم از همان عملگر [] برای دسترسی به آیتم‌ها استفاده کنیم:

    ```c++
    n = fs["Mapping"]; // Read mappings from a sequence
    cout << "Two  " << (int)(n["Two"]) << "; ";
    cout << "One  " << (int)(n["One"]) << endl << endl;
    ```

5.  **ورودی/خروجی ساختمان داده‌های ساختگی:** فرض کنید یک ساختار داده به شکل زیر داریم:

    ```c++
    class MyData
    {
    public:
        MyData() : A(0), X(0), id() {}
    public:   // Data Members
        int A;
        double X;
        string id;
    };
    ```

    می‌توان این کلاس را به وسیلهٔ رابط ورودی/خروجی YAML/XML سریال کرد (درست مثل ساختار داده‌های اُ سی وی). برای این کار باید یک تابع read و یک تابع write، داخل و بیرون کلاس مورد نظر اضافه شود. به عنوان مثال قسمت داخلی کلاس بالا را به شکل زیر تعریف می‌کنیم:

    ```c++
    //Write serialization for this class
    void write(FileStorage& fs) const
    {
        fs << "{" << "A" << A << "X" << X << "id" << id << "}";
    }
    //Read serialization for this class
    void read(const FileNode& node)
    {
        A = (int)node["A"];
        X = (double)node["X"];
        id = (string)node["id"];
    }
    ```

    سپس قسمت خارجی را به صورت زیر تعریف می‌کنیم:

    ```c++
    //These write and read functions must be defined for the serialization in FileStorage to work
    static void write(FileStorage& fs, const std::string&, const MyData& x)
    {
        x.write(fs);
    }
    static void read(const FileNode& node, MyData& x, const MyData& default_value = MyData()){
        if(node.empty())
            x = default_value;
        else
            x.read(node);
    }
    ```

    در قسمت read مشخص کرده‌ایم که اگر نود درخواستی وجود نداشته باشد از یک نود پیش فرض استفاده شود (یک شئ خام از کلاس MyData). البته بهتر است که در برنامه‌های واقعی یک خطا پرتاب کنیم.

    بعد از اضافه کردن این چهار تابع از عملگر >> برای خواندن و از عملگر << برای نوشتن استفاده می‌کنیم:

    ```c++
    MyData m(1);

    fs << "MyData" << m; // Write your own data structures
    fs["MyData"] >> m; // Read your own structure_
    ```

[^32]: Serializing

[^33]: Mappings

[^34]: Element sequence



## خروجی

برنامه را به صورت زیر اجرا می‌کنیم:

```powershell
FileRW.exe "1.xml"
```

محتوای فایل `1.xml` به صورت زیر است:

```xml
<?xml version="1.0"?>
<opencv_storage>
<iterationNr>100</iterationNr>
<strings>
  image1.jpg Awesomeness baboon.jpg</strings>
<Mapping>
  <One>1</One>
  <Two>2</Two></Mapping>
<R type_id="opencv-matrix">
  <rows>3</rows>
  <cols>3</cols>
  <dt>u</dt>
  <data>
    1 0 0 0 1 0 0 0 1</data></R>
<T type_id="opencv-matrix">
  <rows>3</rows>
  <cols>1</cols>
  <dt>d</dt>
  <data>
    0. 0. 0.</data></T>
<MyData>
  <A>97</A>
  <X>3.1415926535897931e+000</X>
  <id>mydata1234</id></MyData>
</opencv_storage>
```

برنامه را به صورت زیر اجرا می‌کنیم:

```powershell
FileRW.exe "2.yml"
```

محتوای فایل `2.yml` به صورت زیر است:

```yaml
%YAML:1.0
iterationNr: 100
strings:
   - "image1.jpg"
   - Awesomeness
   - "baboon.jpg"
Mapping:
   One: 1
   Two: 2
R: !!opencv-matrix
   rows: 3
   cols: 3
   dt: u
   data: [ 1, 0, 0, 0, 1, 0, 0, 0, 1 ]
T: !!opencv-matrix
   rows: 3
   cols: 1
   dt: d
   data: [ 0., 0., 0. ]
MyData:
   A: 97
   X: 3.1415926535897931e+000
   id: mydata1234
```

.