# <center>实验五-文件系统 实验报告</center>

> <center>孙汉武	16281047	安全1601</center>

[TOC]

## 一 概要设计

$\qquad$本次的实验的实验目的是在模拟的I/O系统中开发一个简单的文件系统，并且提供一些借口给用户用于交互，从实验目的可以看出，本实验重点在于构建模拟的I/O系统和基于I/O系统的文件系统。所以，在概要设计中，将详细介绍模拟I/O系统的设计、文件系统的设计和测试模块的设计这三个部分。

### 1.1 I/O系统设计

 $\qquad$IO系设计首先要解决的是需要有一个物理磁盘，为此，我们通过定义一个三维的磁盘块结构体数组表示物理磁盘，该结构体数组的每一个维度分别表示物理磁盘中的一个层次。

<div align="center"><img src="http://ipic-picgo.oss-cn-beijing.aliyuncs.com/2019-06-14-082923.jpg" width="400" ></div>

$\qquad$我们模拟的磁盘如上所示，第一维表柱面，第二维表示磁头，最后一维表示扇区。磁盘定义好之后需要定义一系列的函数用于操作磁盘。IO系统工作的流程和结构如下所示：

<div align="center"><img src="http://ipic-picgo.oss-cn-beijing.aliyuncs.com/2019-06-14-090054.jpg" width="400" ></div>

$\qquad$IO系统提供的操作磁盘的API如上图所示，主要分为五个大类，分别是初始化磁盘、磁盘搜索、磁盘读写、磁盘位图处理、磁盘与文件转化等，下面的表格分别介绍了各个类别提供的API的详细信息。

+ 磁盘初始化

|   函数名   | 参数 | 返回值 |      功能      |
| :--------: | :--: | :----: | :------------: |
| InitDisk() |  无  |   无   | 初始化磁盘数组 |

+ 磁盘搜索 

|    函数名    | 参数 |    返回值    |             功能             |
| :----------: | :--: | :----------: | :--------------------------: |
| SearchBitMap |  无  | 空闲磁盘块号 | 搜索并返回最小的空闲磁盘块号 |

+ 磁盘读写

|   函数名   |                         参数                          | 返回值 |         功能         |
| :--------: | :---------------------------------------------------: | :----: | :------------------: |
| ReadBlock  | int i :指定读取的磁盘块号<br />char *p ：返回读取内容 |   无   |  读取指定磁盘块内容  |
| WriteBlock |   int i：指定写入的磁盘块号<br />char *p:写入的内容   |   无   | 写入内容到指定磁盘块 |

+ 磁盘位图处理

|    函数名    |                        参数                        | 返回值 |        功能        |
| :----------: | :------------------------------------------------: | :----: | :----------------: |
|  InitBitMap  |                         无                         |   无   |     初始化位图     |
| ChangeBitMap | int i:要修改的磁盘块号<br />char p:修改的内容(Y/N) |   无   | 修改位图每一位的值 |

+ 磁盘与文件的转化

$\qquad$由于是在内从中创建数组模拟物理磁盘，所以这种方式无法模拟物理磁盘断电不丢失信息的特性。为了满足这个要求，设计一下的函数用于将数组中的信息存储到文件中和文件中读取相关信息。

|   函数名   |          参数           | 返回值 |           功能           |
| :--------: | :---------------------: | :----: | :----------------------: |
| DiskToFile | char filename[]：文件名 |   无   | 将磁盘数组信息存储为文件 |
| FileToDsik | char filename[]:文件名  |   无   |  将文件读取到磁盘数组中  |

### 1.2 文件系统设计

$\qquad$在上一节设计的IO系统的基础上，进行文件系统的设计，文件系统设计时有两个很重要的概念，分别是文件描述符和目录项这两个数据结构的定以及在这个基础上进行的一系列操作。

1. 用户接口

$\qquad$文件系统提供了一系列便于用户操作的接口，用于对文件系统中的文件进行增删改查，具体接口信息如下：

| 函数名  |                             参数                             | 返回值 |       功能       |
| :-----: | :----------------------------------------------------------: | :----: | :--------------: |
| create  |                       char filename[]                        |   无   |     创建文件     |
| destroy |                       char filename[]                        |   无   |     删除文件     |
|  open   |                       char  filename[]                       |   无   |     打开文件     |
|  close  |                       char filename[]                        |   无   |     关闭文件     |
|  read   | index:文件描述符号<br />mem_area:读取的位置<br />count:读取的字节数 |   无   |   读取文件内容   |
|  write  | index:文件描述符<br />mem_area:x写入的位置<br />count:写入字节数 |   无   | 向文件中写入信息 |
|  lseek  |              index:文件描述符 号<br />pos:位置               |   无   | 移动文件读写指针 |

$\qquad$文件系统提供的上述接口已经可以满足对文件系统的常规操作。上面的接口中提到的文件描述符在文件系统中是一个很重要的概念，每个文件都必须 通过一个文件描述符来表示 ，其文件长度信息 ，文件存储位置等常规 信息都存储在文件描述符中。

$\qquad$为此我设计了一个结构体`FileDescriptor`用于表示文件描述符，并且将该结构体进行4字节对齐，方便后续以二进制形式存储在文件中。

$\qquad$在文件系统中另外一个重要概念就是目录，因此定义一个结构体表示目录项，用于存储文件名和文件描述符号。

<div align="center"><img src="http://ipic-picgo.oss-cn-beijing.aliyuncs.com/2019-06-14-100056.jpg" width="400" ></div>

$\qquad$文件描述符和目录项在磁盘中存储的位置如上图所示，其中磁盘块的第一和第二块用于存储磁盘块的位图，第3-13块用于存储文描述符，14块用于存储目录项信息。

$\qquad$经过4字节对齐之后的文件描述符结构体大小为24字节，而一个磁盘块有512字节存储空间，所以一个磁盘块最多存储21个文件描述符，而本实验中设置的用于存储文件描述符的磁盘块为10块，最多可以存储210个文件描述符。

$\qquad$目录也可以看做一个文件，所以也会占据一个文件文件描述符，本实验中目录占用的文件描述符是第一个文件描述符。并且存储目录信息的磁盘是14号磁盘，经过4字节对齐的目录项结构体大小为16B，所以最多有32个文件。

<div align="center"><img src="http://ipic-picgo.oss-cn-beijing.aliyuncs.com/2019-06-14-102802.jpg" width="400" ></div>

$\qquad$文件系统的API如上图所示，我将文件系统的API分为初始化、用户接口、搜索和其他这四类，每一个大类具体的函数如下表所示：

+ 文件系统初始化

|       函数名       | 参数 | 返回值 |         功能         |
| :----------------: | :--: | :----: | :------------------: |
| InitFileDescriptor |  无  |   无   | 初始化文件描述符数组 |
|      InitMenu      |  无  |   无   |   初始化目录项数组   |

+ 文件系统用户接口(上面已经介绍过，不在赘述)
+ 文件系统搜索

|        函数名        | 参数 | 返回值 |         功能         |
| :------------------: | :--: | :----: | :------------------: |
| SearchFileDescriptor |  无  |   无   | 搜索空闲的文件描述符 |
|    SearchMenuItem    |  无  |   无   |   搜索空闲的目录项   |

+ 其他

$\qquad$这部分主要定义的是一些方便操作的函数，例如将文件描述符数组写入磁盘块中去，将目录系统写入磁盘块等等。

|        函数名        | 参数 | 返回值 |                  功能                  |
| :------------------: | :--: | :----: | :------------------------------------: |
| DiskToFileDescriptor |  无  |   无   |      从磁盘块中恢复文件描述符数组      |
| FileDescriptorToDisk |  无  |   无   |     将文件描述符数组信息写入磁盘块     |
| MenuToFileDescriptor |  无  |   无   |    将目录项数组写入第一个文件描述符    |
| FileDescriptorToMenu |  无  |   无   | 将第一个文件描述符内容恢复到目录项数组 |

$\qquad$到此，文件系统部分所有数据结构和函数均介绍完毕

### 1.3 菜单驱动系统

$\qquad$完成IO系统的设计和文件系统的设计，需要对上述的功能设计一个外壳程序，即一个用户界面便于使用，总结文件系统的所有功能，设计的菜单驱动程序包含如下两层菜单。

1. 一级菜单
    + 创建新磁盘系统
    + 从文件中恢复历史磁盘系统
2. 二级菜单
    + 查看目录
    + 创建文件
    + 删除文件
    + 打开文件
    + 修改文件
    + 查看位图
    + 保存磁盘
    + 退出

## 二 I/O系统

### 2.1 磁盘块结构体

> I/O系统部分全部代码都在IO.h文件中

1. 磁盘块结构体BLOCK

$\qquad$设计整个I/O系统的基础就是设计模拟磁盘块的结构体，并用结构体数组代表磁盘，通过定义三维的结构体数组来模拟出整个磁盘的物理结构，这三维分别代表柱面、磁头和扇区。下面是磁盘块(逻辑块)结构体的定义：

```cpp
typedef struct BLOCK
{
    char Content[512]; //逻辑块存储的内容
    int BlockNnum; //逻辑块号
    int c; // 柱面号
    int h; //磁头号
    int b; //扇区号
}BLOCK;
```

$\qquad$磁盘块结构体最重要的成员就是用于存储信息Content，这是一个字符型数组，大小为512个字节。而另外的逻辑块号，柱面号，磁头号和扇区号这四个成员是为了便于后续程序设计的。下面的表格详细表示每个成员的信息：

|  成员名  |   类型   |  大小   |       作用       |
| :------: | :------: | :-----: | :--------------: |
| Content  | 字符数组 | 512字节 |  存储磁盘块内容  |
| BlockNum |   整型   |  4字节  | 磁盘块的逻辑块号 |
|    c     |   整型   |  4字节  |      柱面号      |
|    h     |   整型   |  4字节  |      磁头号      |
|    b     |   整型   |  4字节  |      扇区号      |

2. 物理磁盘(BLOCK数组)

$\qquad$磁盘块结构体定义好之后，可以模拟出一个磁盘块，但是完整的磁盘是一个三维的磁盘块结构体数组构成的。

```cpp
#define C 10 // 柱面号
#define H 10 //磁头号
#define B 10 //扇区号
BLOCK ldisk[C][H][B];//磁盘模型
```

$\qquad$在`IO.h`直接定义ldisk数组，模拟物理磁盘。并且通过宏定义C、H和B三个量调整物理磁盘的大小。

3. 磁盘初始化函数

$\qquad$在模拟物理磁盘的三维结构体数组定义好之后，需要对该数组进行初始化，对数组中的每个元素，即每个磁盘块进行初始化。

```cpp
void InitDisk(void)
{
    for(int i=0;i<C;i++)
        for(int j=0;j<H;j++)
            for(int k=0;k<B;k++)
            {
                ldisk[i][j][k].c=i;
                ldisk[i][j][k].h=j;
                ldisk[i][j][k].b=k;
                ldisk[i][j][k].BlockNnum=DiskNumToBlock(i,j,k);//计算对应的逻辑块号
            }
}
```

$\qquad$可以看到对磁盘块数组进行初始化的方式非常简单，主要包含两个工作，一个就是顺序编号其柱面号、磁头号和扇区号；还有一个功能就是计算逻辑块号

### 2.2 磁盘读写

$\qquad$说道磁盘系统的API函数，最重要的两个函数就是对磁盘块进行读写操作的两个函数。下面分别介绍这个两个函数的详细内容。

1. 读磁盘块函数：`ReadBlock(int i,char *p):`

```cpp
void ReadBlock(int i,char *p)
{
    int c,h,b;//磁盘的柱面 磁道 扇区
    c = i % (H*B);//
    h = (i -c*H*B) % B;//
    b = i-c*H*B - h*B;//
    memcpy(p,ldisk[c][h][b].Content,512);
}
```

$\qquad$3-6行计算逻辑号为i的逻辑块在磁盘系统中的柱面号、磁头号和扇区号；

$\qquad$第7行可以看作此函数的核心操作，完成的工作就是将制定磁盘块中的内容通过`memcpy`函数复制到字符型指针p中去。

> 注意：这里不能使用strcpy函数复制字符串，因为strcpy函数在复制的时候会遇到第一个\0就会停止复制，但是磁盘块中存储的信息可能不是连续的字符串，可能是一些其他信息，例如文件描述符等，这个时候就会碰到一些空位置用\0补充，但是后面还有有用的信息，所以使用memcp函数，按照指定字节数复制，而不考虑\0的问题

2. 写磁盘块函数：`WriteBlock(int i,char *p):`

```cpp
void WriteBlock(int i,char *p)
{
    int c,h,b;
    b = i % B;
    h = ((i - b) / B) % H;
    c = (i -b -h*B) / (H*B);
    b = i -c*H*B -h*B ;
    memcpy(ldisk[c][h][b].Content,p,512);
}
```

$\qquad$3-6行操作与上面相似，就是计算柱面号、磁头号和扇区号这三个参数

$\qquad$8第8行主要用于将参数p指针中的内容复制到磁盘块中

### 2.3 磁盘位图

$\qquad$为了方便查询磁盘中的空闲磁盘块，直接遍历查询的效率非常低，所以本实验中采用了位图的方式来表示磁盘块的占用与否，一个字符表示一个磁盘块的占用与否，其中Y表示占用，N表示空闲。而位图编号就是磁盘块的逻辑块号数。

1. 初始化磁盘位图函数：`InitBitMap(void)：`

```cpp
void InitBitMap(void)
{
    //第0，1号磁盘已经被占用
    ChangeBitMap(0,'Y');
    ChangeBitMap(1,'Y');
    for (int i=2;i<C*H*B;i++)
    {
        ChangeBitMap(i,'N');
    }
}
```

$\qquad$可以看到初始化函数中将磁盘块数量(C\*B\*H)个单位的字符修改为N，表示为占用。4-5两行表示第一块和第二块物理磁盘初始化就被占用，因为这两块物理磁盘用于存储位图。

2. 修改位图函数：`ChangeBitMap(int i,char p):`

```cpp
void ChangeBitMap(int i,char p)
{
    if(i <512)
        ldisk[0][0][0].Content[i] = p;
    else
        ldisk[0][0][1].Content[i-512] = p;
}
```

$\qquad$这个函数使用一个if结构判断逻辑块号，如果大于512的话存入 第二个磁盘块中，否则存入第一个磁盘块中。

### 2.4 磁盘文件的存取

$\qquad$在内存中定义的磁盘块结构体数组无法满足断电后信息还能保存的特性，因此需要内存中的磁盘块数组中的信息保存到文件中去，需要的时候再加载出来。

1. 将磁盘数组保存为文件：`DiskToFile(char filename[])：`

```cpp
void DiskToFile(char filename[])
{
//    FileDescriptorToDisk();
    FILE *fp;
    fp = fopen(filename,"wb");
    //判断fp打开成功
    if (fp ==NULL)
    {
        printf("File Open Fail");
        exit(1);
    }
    // 循环遍历，将磁盘块内容写入二进制文件中去
    for(int i=0;i<C;i++)
        for(int j=0;j<H;j++)
            for(int k=0;k<B;k++)
                //以二进制的形式写入二进制文件中
                fwrite(ldisk[i][j][k].Content,512,1,fp);
    fclose(fp);
}
```

$\qquad$3-5行以二进制写入形式打开指定文件名的文件

$\qquad$7-11行判断文件是否打开成功，没打开成功的话输出错误信息并退出程序

$\qquad$13-17行，遍历磁盘数组，以二进制的形式将磁盘块中存储的内容写入到文件中去。最后关闭文件

2. 加载文件到磁盘块数组：`FileToDisk(char filename[])：`

```cpp
//从文件中读取数据，恢复磁盘系统
void FileToDisk(char filename[])
{
     FILE *fp;
     fp = fopen(filename,"rb");
     if(fp == NULL)
     {
         printf("File Open Fail");
         exit(1);
     }
     int index = 0;
     while(!feof(fp))
     {
         int c,h,b;
         b = index % B;
         h = ((index - b) / B) % H;
         c = (index -b -h*B) / (H*B);
         b = index -c*H*B -h*B ;
         fread(ldisk[c][h][b].Content,512,1,fp);
         ldisk[c][h][b].c = c;
         ldisk[c][h][b].h = h;
         ldisk[c][h][b].b = b;
         index++;
     }
     fclose(fp);
}
```

$\qquad$这段程序除了打开文件等常规操作之外，核心的代码是while循环中的，首先计算逻辑号为index的逻辑块的柱面号、磁头号和扇区号；之后每次读取512字节数据到对应磁盘块数组中的磁盘块中去。

### 2.5 空闲磁盘块搜索

$\qquad$2.3节中定义了位图，用于存储磁块的空闲状态，所以当需要使用磁盘块的时候 ，需要查询位图找到一个空闲磁盘块号返回。

1. 空闲磁盘块搜索函数：`SearchBitMap(void)：

```cpp
//搜索位图，找到空闲磁盘块号
int SearchBitMap(void)
{
    for(int i=14;i<C*H*B;i++)
    {
        if(i<512)
        {
            if(ldisk[0][0][0].Content[i]=='N')
                return i;
        }
        else
        {
            if(ldisk[0][0][1].Content[i-512]=='N')
                return i;
        }
    }
}
```

 $\qquad$本函数的实现方式就是通过遍位图知道找到一个空闲的磁盘块。不过由于所有的位图 信息并不是全部存储在一个磁盘块中，而是两个磁盘块，所以在遍历的时候需要判断在哪个磁盘块。

## 三 文件系统

> 文件系统全部代码存储在FS.h文件中

### 3.1 文件描述符结构体 & 目录项结构体

1. 文件描述符结构体定义

 $\qquad$文件系统采用文件描述符来记录每一个文件的信息，下面是文件描述符的结构体定义：

```cpp
#pragma pack(4)
typedef struct FileDescriptor //此文件描述符总共占据磁盘24字节
{
    int Length;//文件长度
    int DiskNum[DiskNumLen]; //第二个3只是表示每个磁盘块好最大长度是3位
    int Num; //文件描述符号
    char IsFree; //表示此文件描述符是否空闲
}FileDescriptor;
#pragma pack(pop)
```

> 在这个结构体定义有一个特别的地方需要注意，就是使用了4字节对齐机制，因为后来这些结构体需要存储到字符型数组中，如果不采用对齐的话可能会导致不同结构体的长度不同，读取的时候就没办法读取.

$\qquad$下面是该结构体各个成员的详细解释：

| 成员名  |   类型   |     大小      |            作用            |
| :-----: | :------: | :-----------: | :------------------------: |
| Length  |   整型   |     4字节     |        存储文件大小        |
| DiskNum | 整型数组 |    12字节     | 存储文件内容的磁盘块好数组 |
|   Num   |   整型   |     4字节     |        文件描述符号        |
| IsFree  |  字符型  | 4字节(对齐后) |   表示当前描述符时候空闲   |

2. 目录项结构体定义

$\qquad$目录是文件系统必不可缺的组成部分，本实验中通过目录项数组组成一个目录，而每个目录项由文件名和文件描述符号组成。下面是目录项结构体定义：

```cpp
#pragma pack(4)
typedef struct MenuItem //目录对应0号文件描述符,一个目录项占据16字节，所以一个文件描述符可以存储96个文件
{
    char FileName[12]; //目录项中文件名的最大长度为16字节
    int FileDescriptorNum;//文件描述符号
}MenuItem;
```
