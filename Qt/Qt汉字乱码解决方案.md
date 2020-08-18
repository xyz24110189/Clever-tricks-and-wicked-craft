彻底解决Qt中文乱码以及汉字编码的问题(UTF-8/GBK)

一、Qt Creator环境设置
1、cpp或h文件从window上传到Ubuntu后会显示乱码,原因是因为ubuntu环境设置默认是utf-8,Windows默认都是GBK.

我们使用的Windows系统本地字符集编码为GBK。

2、Windows环境下,Qt Creator,菜单->工具->选项->文本编辑器->行为->文件编码->默认编码,常用的选项有以下几个:

System(简体中文windows系统默认指的是GBK编码)

GBK/windows-936-2000/CP936/MS936/windows-936

UTF-8

 

二、编码知识科普
Qt常见的两种编码是:UTF-8和GBK
★UTF-8：Unicode TransformationFormat-8bit，允许含BOM，但通常不含BOM。是用以解决国际上字符的一种多字节编码，它对英文使用8位（即一个字节），中文使用24为（三个字节）来编码。UTF-8包含全世界所有国家需要用到的字符，是国际编码，通用性强。UTF-8编码的文字可以在各国支持UTF8字符集的浏览器上显示。如果是UTF8编码，则在外国人的英文IE上也能显示中文，他们无需下载IE的中文语言支持包。
★GBK是国家标准GB2312基础上扩容后兼容GB2312的标准。GBK的文字编码是用双字节来表示的，即不论中、英文字符均使用双字节来表示，为了区分中文，将其最高位都设定成1。GBK包含全部中文字符，是国家编码，通用性比UTF8差，不过UTF8占用的数据库比GBD大。GBK是GB2312的扩展，除了兼容GB2312外，它还能显示繁体中文，还有日文的假名。
★GBK、GB2312等与UTF8之间都必须通过Unicode编码才能相互转换：
GBK、GB2312－－Unicode－－UTF8
UTF8－－Unicode－－GBK、GB2312
★在简体中文windows系统下，ANSI编码代表GBK/GB2312编码，ANSI通常使用0x80~0xFF范围的2个字节来表示1个中文字符。0x00~0x7F之间的字符，依旧是1个字节代表1个字符。Unicode(UTF-16)编码则所有字符都用2个字节表示。

 

三、编码转换
Windows自带的记事本，无法查看UTF-8编码的文件到底有无BOM，需要使用其他文件编辑器，比如EditPlus或者SublimeText。
UTF-8与ANSI(即GBK)的互转,可以使用EditPlus工具"文件另存为"或者Encodersoft编码转换工具对.cpp和.h源文件文本进行批量转换.

 

四、QString显示中文乱码的原因
我们使用的Windows系统本地字符编码(Local字符集)为GBK。编译器分析出源文件字符编码之后，会进行解码再编码，将源字符集转码成执行字符集。执行字符集一般默认为使用本地字符编码(Local字符集)。

Qt5可以设置Local字符集，GBK/UTF-8

QTextCodec *codec = QTextCodec::codecForName("UTF-8");//或者"GBK",不分大小写
QTextCodec::setCodecForLocale(codec);
Qt5中QString内部采用unicode字符集，utf-16编码。构造函数QString::QString(const char *str)默认使用fromUtf8()，将str所指的执行字符集从utf-8转码成utf-16。
由上面fromUtf8()可知，QString需要执行字符集编码为utf-8，然后以utf-8进行解码，再编码为utf-16才能获得正确的字符编码。显示中文乱码的原因其实就是QString转码方式与执行字符集不一致。(比如，源字符集为本地字符集GBK编码，QString以utf-8的方式进行解码，会导致获得错误的二进制编码，再将错误二进制转为utf-16就会出现乱码。)
 

五、Qt编码指定
Qt需要在main()函数指定使用的字符编码:
```
#include <QTextCodec>
 
int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
 
    //设置中文字体  
    a.setFont(QFont("Microsoft Yahei", 9));
 
    //设置中文编码
#if (QT_VERSION <= QT_VERSION_CHECK(5,0,0))
#if _MSC_VER
    QTextCodec *codec = QTextCodec::codecForName("GBK");
#else
    QTextCodec *codec = QTextCodec::codecForName("UTF-8");
#endif
    QTextCodec::setCodecForLocale(codec);
    QTextCodec::setCodecForCStrings(codec);
    QTextCodec::setCodecForTr(codec);
#else
    QTextCodec *codec = QTextCodec::codecForName("UTF-8");
    QTextCodec::setCodecForLocale(codec);
#endif
 
    return a.exec();
}

```
这里只列举大家最常用的3个编译器(微软VC++的cl编译器，Mingw中的g++，Linux下的g++)，源代码分别采用GBK和无BOM的UTF-8以及有BOM的UTF-8这3种编码进行保存,发生的现象如下表所示。

情况1：指的是Local字符集为GBK

情况2：指的是Local字符集为UTF-8

源代码的编码

编译器

显示正常

显示乱码
GBK

win vs cl

情况1

情况2

win mingw-g++

情况1

情况2

linux g++

情况1

情况2
UTF-8(无BOM)

win vs cl

编译失败

error C2001: 常量中有换行符

编译失败
error C2001: 常量中有换行符
win mingw-g++

情况2

情况1
linux g++

情况2

情况1

UTF-8(有BOM)

win vs cl

情况1

情况2(有#pragma预处理)

情况2(没有#pragma预处理)
win mingw-g++

情况2

情况1
linux g++

情况2

情况1
如果您使用的是Visual C++编译器，则默认情况下不会将您的源代码视为utf-8编码。除非有BOM，否则它将使用您当前的代码页进行解释。就是说，当使用Visual C++编译程序的时候，它会分析源文件采用何种编码，有BOM标识符则可以正确识别其编码是UTF-8，若没有BOM标识符则认为其使用本地字符集编码(Local字符集)。Local字符集是什么？取决于你的设置QTextCodec *codec = QTextCodec::codecForName(???);
如果源文件是UTF-8+BOM的编码方式，还需要在头文件加入
#if defined(_MSC_VER) && (_MSC_VER >= 1600)    
# pragma execution_character_set("utf-8")    
#endif
或者添加QMAKE_CXXFLAGS += /utf-8到您的.pro文件中。

如果源文件是UTF-8+无BOM的编码方式，则一定不能加#pragma execution_character_set(“utf-8”)，不然会产生乱码。
 

六、测试案例
案例1、中文字符串测试
```
#include <QApplication>
#include <QTextCodec>
#include <QPushButton>
#include <QDebug>
#include <QString>
 
int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
 
    //设置中文字体  
    a.setFont(QFont("Microsoft Yahei", 9));
 
    //设置中文编码
#if (QT_VERSION <= QT_VERSION_CHECK(5,0,0))
#if _MSC_VER
    QTextCodec *codec = QTextCodec::codecForName("gbk");
#else
    QTextCodec *codec = QTextCodec::codecForName("utf-8");
#endif
    QTextCodec::setCodecForLocale(codec);
    QTextCodec::setCodecForCStrings(codec);
    QTextCodec::setCodecForTr(codec);
#else
    QTextCodec *codec = QTextCodec::codecForName("utf-8");
    QTextCodec::setCodecForLocale(codec);
#endif
 
    QString str(QObject::tr("1中文"));
    qDebug() << str;
    qDebug() << QStringLiteral("2中文");
    qDebug() << QString::fromLatin1("3中文");
    qDebug() << QString::fromLocal8Bit("4中文");
    qDebug() << QString::fromUtf8("5中文");
    qDebug() << QString::fromWCharArray(L"6中文");
 
    return a.exec();
}

```
当QTextCodec::codecForName("utf-8");时，

QString::fromLocal8Bit和QString::fromUtf8是等效的。

当QTextCodec::codecForName("gbk");时，

QString::fromLocal8Bit和QString::fromUtf8是不等效的。

案例2、QCom跨平台串口调试助手(http://www.qter.org/?page_id=203)
源代码qcom\mainwindow.cpp,aboutdialog.cpp等文件用的是UTF-8编码(无BOM);但是qcom\qextserial\*.*文件用的是ANSI编码.在linux环境编译完全OK.
笔者Windows环境的Qt Creator+微软VC++编译器,环境设置用的是ANSI(即GBK)编码.编译源文件会报错.
错误提示"fatal error C1018: 意外的 #elif".


解决方法由两种：

方法1：



把qcom\的所有cpp和h文件都用工具转换成ANSI编码,main()函数使用QTextCodec::setCodecForTr(QTextCodec::codecForName("GBK"));

方法2：

先把Qt Creator环境设置用的是UTF-8编码,



再把qcom\的所有cpp和h文件都用工具转换成UTF-8+BOM编码,请注意,如果文件转换成UTF-8(无BOM),编译仍会失败.main()函数使用QTextCodec::setCodecForTr(QTextCodec::codecForName("GBK"));//注意,此处仍是"GBK",不是"UTF-8"
重新编译,OK!

其它：

 

七、结论
Windows环境下,Qt Creator+微软VC++编译器,新建工程,

1、如果该工程不需要跨平台使用(只在win),那么工程设置请使用GBK的编码方式.

2、如果该工程要跨平台使用(win+linux),那么工程设置请使用UTF-8+BOM的编码方式.



另外，还需要在头文件加入

#if defined(_MSC_VER) && (_MSC_VER >= 1600)    
# pragma execution_character_set("utf-8")    
#endif
或者添加QMAKE_CXXFLAGS += /utf-8到您的.pro文件中。

3、Linux环境下,Qt Creator+gcc,新建工程,

没有GBK编码可选,默认是UTF-8(无BOM)编码方式,考虑到跨平台,建议选择UTF-8+BOM的编码方式.

★★★★★综上所述，概括如下：★★★★★

1、把Qt Creator IDE的环境设置为“UTF-8+BOM”编码。

2、所有源文件和头文件都保存为“UTF-8+BOM”编码。

3、预编译头文件加入

#if defined(_MSC_VER) && (_MSC_VER >= 1600)    
# pragma execution_character_set("utf-8")    
#endif

4、如此一来，不管是MSVC编译器还是MinGW编译器，都能编译通过，且支持中文！

★★★★★★★★★★

 

x、参考文献
Qt官网文档 

https://wiki.qt.io/Strings_and_encodings_in_Qt

https://doc.qt.io/qt-5/unicode.html

ASCII，Unicode和UTF-8完全搞清楚 https://blog.csdn.net/Deft_MKJing/article/details/79460485

Qt中文乱码原因及解决方案 https://blog.csdn.net/qq_35905572/article/details/95042444

Qt中文乱码问题 http://blog.csdn.net/brave_heart_lxl/article/details/7186631

尊重作者，支持原创，如需转载，请附上原地址：https://blog.csdn.net/libaineu2004/article/details/19245205

 
