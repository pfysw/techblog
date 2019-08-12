
词法分析器flex和语法分析器lemon的初步使用


自己手写词法分析器和语法分析器是很麻烦的一件事，而且这里面的逻辑非常复杂很容易出错。flex和lemon就是用来帮助生成词法分析器和语法分析器的，只需要写少量规则代码，就可以生成解析的c代码。现在先不关注实现原理，主要看一下这东西是怎么用的，等以后用熟了要实现深度定制的时候再来看实现源码。

 #  1.词法分析器
 首先得安装一个flex，至于怎么安装就不讲了。词法分析器的功能就是把一串字符串按照给定的规则分割成一个个记号（Token）,对于flex来说，需要写一个指明规则的.i文件，然后再生成.c文件。

还是举个例子吧，比如下面这个.i文件，是用来解析形如~A->B->C的字符串，把字符串分割成 ~ 、A、->、B、->、C

```c
%{
    #include "token.h"
%}


%option reentrant
%option noyywrap


%%

[a-zA-Z]+    { return TK_SYM; }
~    { return TK_NEG; }
->    { return TK_IMPL; }

%%
```
在%{和%}的内容会原封不动的搬到c代码里，一般是头文件。%option是指定词法编译时的一些规则，%option reentrant这个定义十分重要，没有这行生成的yy_flex函数是不带参数的int yylex (void)，有了这一行就会生成带参数的yy_flex函数：int yylex (yyscan_t yyscanner)，可以看到没有带参数的只能返回Token值，而带参数的就可以从yyscanner中取得实际值还有行数等其他信息，例如解析到A时，不带参数的只返回TK_SYM，而带参数的还可以返回该Token对应的值A。

在%%包含的范围内左边是要解析的规则，以正则表达式的形式表示，如[a-zA-Z]+表示的是一个或多个字母，右边{}里是解析到对应的规则所要执行的c语言动作，这里是返回对应的Token，如果不返回的话yy_flex()函数将会一直执行下去。

```c
int main(int argc, char** argv)
{

  int token;
  yyscan_t scanner;
  FILE *fd;

  setbuf(stdout, NULL);
  yylex_init(&scanner);

  if (argc > 1)
  {
    if (!(fd = fopen(argv[1], "r")))
    {
      perror(argv[1]);
      return 1;
    }
    else
    {
      yyset_in(fd, scanner);
    }
  }

  token = yylex(scanner);
  while (token)
  {
    printf("tocken %d %s\n", token, yyget_text(scanner));
    token = yylex(scanner);
  }
  printf("end %d %s\n", token, yyget_text(scanner));
  yylex_destroy(scanner);

  return 0;
}
```

输入参数scanner在使用前要初始化，使用完毕后要销毁，如果程序带参数，那么将以第一个参数对应的文件作为输入，通过 yyset_in(fd, scanner);设置。每一次调用yylex(scanner);后返回Token值，Token需要自己宏定义，在前面的.i文件中包含了token.h头文件，在头文件里定义

```c
#define TK_SYM 1
#define TK_NEG 2
#define TK_IMPL 3
```
token值从1开始，不能设为0，因为0是默认返回值，语法分析器也是以0作为结束标志。解析到的Token信息存放在scanner结构体变量里，yyget_text(scanner)是对应的字符串值。

另外yylex()函数的逻辑是先扫描完整个文件也就是扫描到文件结束符后再开始解析，从stdin输入flex会根据输入流自动调整从而变为出现回车符后就解析，但这个在eclipse里调试没用，所以会出现输入字符串加回车键没反应的现象，如果想要扫描到回车键后就解析可以在yy_init_buffer函数里把b->yy_is_interactive手工设为1

另外还有一个状态切换的语法，感觉没什么用，但还是讲一下吧，看下面的代码

```c
["]                     { BEGIN(DOUBLE_QUOTED); }
<DOUBLE_QUOTED>[^"]+    { }
<DOUBLE_QUOTED>["]      { BEGIN(INITIAL); return ARGUMENT; }
<DOUBLE_QUOTED><<EOF>>  { return -1; }
```
这个代码的意思是解析到第1个双引号后进入DOUBLE_QUOTED状态，<DOUBLE_QUOTED>的意思是只有在DOUBLE_QUOTED的状态下才对后面的规则解析，<DOUBLE_QUOTED>["]是解析到第2个双引号后返回初始状态INITIAL，状态需要先通过这一句%x DOUBLE_QUOTED定义好状态名，INITIAL已经默认定义好了不需要用户定义。

 #  2.语法分析器

一般比较常用的语法分析器是bison，而我这里选择的是lemon，lemon本来是为sqlite的语法解析而开发的，当然也可以做其他的语法解析，功能和bison差不多，但是lemon更加小巧，语法更友好，调试信息很丰富，更重要的是有人给lemon的源码写了一本很详细的书，叫《LEMON语法分析生成器(LALR 1类型)源代码情景分析》

lemon源代码就一个lemon.c文件，直接编译就好了。然后用lemon生成语法分析的.c文件时需要用到2个文件，分别是.y后缀的语法文件和模板文件lempar.c。如果语法文件是test.y,经过命令./lemon test.y就生成了test.c的语法分析文件和用来说明状态转换的.out文件，先来看一下api的使用，如下是解析一个计算器语法1+2

```c
int main()
{
    void* pParser = ParseAlloc(malloc);
    
    ParseTrace(stdout, "");
    Parse(pParser, NUM, 1);
    Parse(pParser, PLUS, 0);
    Parse(pParser, NUM, 2);

    Parse(pParser, 0, 0);
    ParseFree(pParser, free);
}
```
这里NUM、PLUS是宏定义好了的Token对象，开始语法分析前先创建一个语法分析对象，并设定要使用的内存分配函数，使用完毕后要记得销毁。 ParseTrace(stdout, "");这一行输出的整个语法解析过程的调试信息，主要是关于LRLA(1)分析时移进和规约的一些细节。然后把然后每次输入一个Token和对应的关联信息给Parse函数进行解析，当然Token可以由词法分析器生成。

下面来看整个语法文件，就是带.y后缀的文件

```c
%include {
#include <iostream>
#include "example1.h"
using namespace std;

}


%token_type { int }
%left PLUS MINUS.
%left DIVIDE TIMES.

program ::= expr(A). {
                            cout << "Result = " << A << "\n" << endl;
                            }

expr(A) ::= expr(B) MINUS expr(C). { A = B - C; }
expr(A) ::= expr(B) PLUS expr(C). { A = B + C; }
expr(A) ::= expr(B) TIMES expr(C). { A = B * C; }
expr(A) ::= expr(B) DIVIDE expr(C). { A = B / C; }

expr(A) ::= NUM(B). { A = B; }
```

%开头的参数指的是Lemon的特殊声明，%include指的是头文件，这部分内容会原封不动移到生成的.c文件开头，%token_type指定Parse第3个参数的类型，一般是Token对应信息参数的类型，解析时每个表达式后的小括号会跟一个对象，这个对象就是传入的第3个参数。当然还可以用%extra_argument指定第4个参数类型，从而可以在解析过程中传递和解析更多信息。%left指定运算是左结合的，同时下一行声明的%left运算符优先级比上一行高。

接下来就是语法声明部分了，如果输入的表达式满足::=的右边时，将被规约到左边，{}里的内容是规约到该语法规则时执行的代码，expr后面的括号是Token绑定的对象，之前说了由Parse()函数的第3个参数输入。表达式又分为终结符和非终结符，终结符用大写字母表示，由Parse()的第2个参数输入，使用前要先定义好

```c
#define PLUS                            1
#define MINUS                           2
#define DIVIDE                          3
#define TIMES                           4
#define NUM                             5
```
0不能定义，这个是默认作为输入终止的符号，非终结符用小写字母表示，由语法产生式规约得到，只在解析器内部使用。

下面简单说一下整个解析的过程，首先输入NUM(1)，这时候语法栈中没有任何符号，不用做规约，直接把NUM压栈即可，再次输入PLUS，这时候栈中有NUM符号，根据语法expr(A) ::= NUM(B)先将其规约为expr(A)，因为当前输入的不是结束符，所以program ::= expr(A)不能规约，这个规则放在第一行，表示当前语法模块已经完全解析完毕，expr不能进一步规约后将PLUS入栈，此时栈中有expr、PLUS。接下来输入NUM(2)，没有要规约的，将NUM入栈，此时栈中有expr、PLUS、NUM。最后输入结束符，先把NUM规约为expr，此时栈中有expr、PLUS、expr再由expr(A) ::= expr(B) PLUS expr( C).语法继续规约为expr，由于现在输入是结束符，所以expr最后还可以规约为program，这时候解析完全结束，会调用yy_accept函数。

当然想要更透彻的理解整个过程还要结合.out文件里状态转换图和 ParseTrace(stdout, "");输出的移进和规约信息，以下是ParseTrace调试信息
```c
Input NUM
Shift 8
Stack: NUM
Input PLUS
Reduce [expr ::= NUM].
Shift 5
Stack: expr
Shift 3
Stack: expr PLUS
Input NUM
Shift 8
Stack: expr PLUS NUM
Input $
Reduce [expr ::= NUM].
Shift 6
Stack: expr PLUS expr
Reduce [expr ::= expr PLUS expr].
Shift 5
Stack: expr
Reduce [program ::= expr].
Result = 3

Accept!
Popping $
```

想要对解析过程做一些修改满足额外需要，主要关注下面这几个函数

```c
yy_find_shift_action()//确认移进和规约的转移状态
yy_shift()//执行移进操作
yy_reduce()//执行规约操作
yy_find_reduce_action()//确认是否可以进一步规约
```

关于这几个函数《LEMON语法分析生成器(LALR 1类型)源代码情景分析》的359页有讲解，当然直接看是不好理解的，需要结合lemon源码和LALR(1)的分析过程整体理解。
