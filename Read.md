###阶段性报告 吴楚凡 计算机1802 20184314
1. 课程设计进展：
词法分析，语法分析已完成，符号表数据结构已完成，语义分析尚处于算法阶段。
问题：
由于前期未准备进行后端设计，若需要实现后端则需要更多时间。
符号表尚未经过与语义分析的接合，容易在过程中出现问题，需要维护。
下一步的计划：
完成语义分析的剩余内容，开展后端实现工作。
若有剩余时间，尝试数组、结构体、多文件编译的内容。
我完成的工作：
1.词法分析（自动机实现）可读取KT，PT中的字符与名称并输出Token串（到word.txt），并同时输出标识符表（IT），常量表（CT）（包含了各种常量）。
2.由于词法分析由我负责，所以顺便更改程序处理了与语法分析间的接口。
3.同理，出于设计接口的目的，设计了符号表的大部分内容（除了函数表和代码记录部分），由于语法内未含有数组与结构体部分（在我看来可以将二者类型表部分一起处理），展示只是留下接口，若有剩余时间再做处理。
2. 代码工作部分：
1. 	Word.h:
自动机实现的词法分析，可读取KT，PT中的字符与名称并输出Token串（到word.txt），并同时输出标识符表（IT），常量表（CT）（包含了各种常量）。
class Word{
public:
    Word();//构造函数
    void AddString(string str,int num);//读取字符串并转化为自动机，原用于读取文件中的关键字与符号并转化为自动机
    void Initialize();//初始化（构造由关键字与字符组成的自动机，打开待编译）
    void operate();//启动自动机
    void AddTrans(char Char,int Num);//用于构造自动机的辅助函数，若可转移则优先转移，否则创建新节点，构建新状态转移并转移到该节点
    bool AddStr(char name,int num);//用于一个一个读取字符并构造自动机
    void ReadTable();//读取关键字与字符表
    void shownode();//展示所有节点的状态转移
    void process();//从待编译文件中读取char并转移
    void wordreturn(char choice);//规约
    void Readstr();//字符串处理函数
    void Readchar();//字符处理函数
    void Readnum();//数字处理函数
    void Readki();//由于无法分别关键字与标识符，在同一个函数进行处理
    void Readp();//符号处理函数
private:
    ifstream input;//输入文件流
    ofstream ITout;
    ofstream CTout;
    ofstream output;//输出文件流
    char currentchar;//当前字符w
    wordnode WordNode[1024];//自动机
    int currentnode;//当前节点
    int nodesum;//节点数量
    int snum;
    int cnum;
    int inum;
    int nnum;
    int knum;
    int pnum;//各可规约项数量
    string str;//记录input中写入的字符串，用于输入到常数与标识符表
    string KT[100];
    string PT[100];//关键字与符号输出字符储存
    string IT[100];
    string CT[100];
    string name[4];//常量输出字符储存
};
	形成的自动机：
2. MainTable.h：
符号表总表类：
由于一个符号表总表对应一个函数，采用一个符号表总表对应一个函数表和多个符号表总表节点的方式。其中函数表为另一个同学设计。
class MainTable//符号表总表类
{
public:
    MainTable();//构造函数
    void setFuc(int Level,int Off,int Fn);//设置函数那啥
void setType(TypeList *newType,int newCat,int order);//设置类型order是数组序号
    void setVar(int var,int order);
    void setVar(float var,int order);
    void setVar(char var,int order);
    void setVar(bool var,int order);//赋值重载函数
   void addPara(string id,TypeList *newType,Cat newCat);//增加参数表 不推荐使用
    void addPara(int order);//增加参数表 用序号 如<I,9>为参数则 addPara(9);
    ParameterTable* retParaList();//返回参数
    //CodeList *retCodeList();
    char gettype(int order);//返回对应序号的类型
    void showTable();//输出所有已经赋值的表
    ~MainTable();//析构函数 负责删除所有申请的空间
private:
    bool usable[200];//记录是否已赋值 （不知道是否真的需要）
    PFINFL FucList;//符号表总表对应的函数 的函数表
MainNode MainT[200];//变量表 初始化时输入所有标识符名 关于大小 可添加overflow处理，改为链表等方式来改进
};
符号表总表节点：
由于需要储存许多类型采用如下形式
Private:
string ID;//标识符名称
TypeList *Type;//类型表指针 由于当前语法只含有基本类型，未设计功能
Cat aCat;//枚举类
void *ADDR;//万能指针 指向基本数据类型（考虑数组）在变量被声明时申请空间
//在符号表总表的析构函数中delete空间，不会发生空间未释放的错误
ps：由于开始打算使用万能指针来指向函数表，在调试时多花了一点时间才发现问题，为了让做完语法分析开始语义分析的同学能尽可能少等待，采用了一个符号表总表对应一个函数表，最上层总表记录函数和全局变量的数据结构，可能需要在后续实验中进行维护。
3. TypeList.h;
简单的类型表，只包含基本数据类型，后端指针全部设置为空。
class TypeList
{
public:
    void setTypeNode(char NewType,void *NewAddr);//
    char getType();
private:
    char Type;
    void *Addr;// 是否需要新建结构体类（包含结构体和数组的）
};
4.ParameterTable.h;
参数表，和符号表总表类似（除了未包含函数表）。
但是与符号表总表相比，增加了增加参数的函数
5.ConstTable.h;
常量表，由于输出Token串为<TokenType,TokenN>类型,采用了记录所有常量于一个常量表，通过Token的后一个字符为指针，通过调用函数（以Token后一个字符为地址）的方式来取得常量。且由于常量种类不唯一，采用字符串处理，读取时根据类型转化为对应常量的方式。
class ConstTable//常数表，用于将扫描到的常数储存到变量指向的地址里面
{
public://通过对不同函数的调用，来将字符串转化为数值，字符或字符串返回即可 *****************
    ConstTable();//直接打开CT进行读取
    float getfloat(int i);
    int getint(int i);//两者都是针对数值 但是一个取所有并转化为float 另一个取整数
    string getstring(int i);
    char getchar(int i);
    void addconst(string a);
private:
    int k;
    string CT[100];//100是否足够？
                   //因为使用了数组，会导致占用大量连续空间，但是string（常量）保持不变，所以不需要改动（改动需要移动大量数据）
};
由于语义分析部分还没有正式开始，符号表部分并没有可以展示的部分并且可能需要更改。

