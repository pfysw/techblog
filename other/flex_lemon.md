
�ʷ�������flex���﷨������lemon�ĳ���ʹ��


�Լ���д�ʷ����������﷨�������Ǻ��鷳��һ���£�������������߼��ǳ����Ӻ����׳���flex��lemon���������������ɴʷ����������﷨�������ģ�ֻ��Ҫд����������룬�Ϳ������ɽ�����c���롣�����Ȳ���עʵ��ԭ����Ҫ��һ���ⶫ������ô�õģ����Ժ�������Ҫʵ����ȶ��Ƶ�ʱ��������ʵ��Դ�롣

 #  1.�ʷ�������
 ���ȵð�װһ��flex��������ô��װ�Ͳ����ˡ��ʷ��������Ĺ��ܾ��ǰ�һ���ַ������ո����Ĺ���ָ��һ�����Ǻţ�Token��,����flex��˵����Ҫдһ��ָ�������.i�ļ���Ȼ��������.c�ļ���

���Ǿٸ����Ӱɣ������������.i�ļ�����������������~A->B->C���ַ��������ַ����ָ�� ~ ��A��->��B��->��C

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
��%{��%}�����ݻ�ԭ�ⲻ���İᵽc�����һ����ͷ�ļ���%option��ָ���ʷ�����ʱ��һЩ����%option reentrant�������ʮ����Ҫ��û���������ɵ�yy_flex�����ǲ���������int yylex (void)��������һ�оͻ����ɴ�������yy_flex������int yylex (yyscan_t yyscanner)�����Կ���û�д�������ֻ�ܷ���Tokenֵ�����������ľͿ��Դ�yyscanner��ȡ��ʵ��ֵ����������������Ϣ�����������Aʱ������������ֻ����TK_SYM�����������Ļ����Է��ظ�Token��Ӧ��ֵA��

��%%�����ķ�Χ�������Ҫ�����Ĺ�����������ʽ����ʽ��ʾ����[a-zA-Z]+��ʾ����һ��������ĸ���ұ�{}���ǽ�������Ӧ�Ĺ�����Ҫִ�е�c���Զ����������Ƿ��ض�Ӧ��Token����������صĻ�yy_flex()��������һֱִ����ȥ��

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

�������scanner��ʹ��ǰҪ��ʼ����ʹ����Ϻ�Ҫ���٣�����������������ô���Ե�һ��������Ӧ���ļ���Ϊ���룬ͨ�� yyset_in(fd, scanner);���á�ÿһ�ε���yylex(scanner);�󷵻�Tokenֵ��Token��Ҫ�Լ��궨�壬��ǰ���.i�ļ��а�����token.hͷ�ļ�����ͷ�ļ��ﶨ��

```c
#define TK_SYM 1
#define TK_NEG 2
#define TK_IMPL 3
```
tokenֵ��1��ʼ��������Ϊ0����Ϊ0��Ĭ�Ϸ���ֵ���﷨������Ҳ����0��Ϊ������־����������Token��Ϣ�����scanner�ṹ������yyget_text(scanner)�Ƕ�Ӧ���ַ���ֵ��

����yylex()�������߼�����ɨ���������ļ�Ҳ����ɨ�赽�ļ����������ٿ�ʼ��������stdin����flex������������Զ������Ӷ���Ϊ���ֻس�����ͽ������������eclipse�����û�ã����Ի���������ַ����ӻس���û��Ӧ�����������Ҫɨ�赽�س�����ͽ���������yy_init_buffer�������b->yy_is_interactive�ֹ���Ϊ1

���⻹��һ��״̬�л����﷨���о�ûʲô�ã������ǽ�һ�°ɣ�������Ĵ���

```c
["]                     { BEGIN(DOUBLE_QUOTED); }
<DOUBLE_QUOTED>[^"]+    { }
<DOUBLE_QUOTED>["]      { BEGIN(INITIAL); return ARGUMENT; }
<DOUBLE_QUOTED><<EOF>>  { return -1; }
```
����������˼�ǽ�������1��˫���ź����DOUBLE_QUOTED״̬��<DOUBLE_QUOTED>����˼��ֻ����DOUBLE_QUOTED��״̬�²ŶԺ���Ĺ��������<DOUBLE_QUOTED>["]�ǽ�������2��˫���ź󷵻س�ʼ״̬INITIAL��״̬��Ҫ��ͨ����һ��%x DOUBLE_QUOTED�����״̬����INITIAL�Ѿ�Ĭ�϶�����˲���Ҫ�û����塣

 #  2.�﷨������

һ��Ƚϳ��õ��﷨��������bison����������ѡ�����lemon��lemon������Ϊsqlite���﷨�����������ģ���ȻҲ�������������﷨���������ܺ�bison��࣬����lemon����С�ɣ��﷨���Ѻã�������Ϣ�ܷḻ������Ҫ�������˸�lemon��Դ��д��һ������ϸ���飬�С�LEMON�﷨����������(LALR 1����)Դ�����龰������

lemonԴ�����һ��lemon.c�ļ���ֱ�ӱ���ͺ��ˡ�Ȼ����lemon�����﷨������.c�ļ�ʱ��Ҫ�õ�2���ļ����ֱ���.y��׺���﷨�ļ���ģ���ļ�lempar.c������﷨�ļ���test.y,��������./lemon test.y��������test.c���﷨�����ļ�������˵��״̬ת����.out�ļ���������һ��api��ʹ�ã������ǽ���һ���������﷨1+2

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
����NUM��PLUS�Ǻ궨����˵�Token���󣬿�ʼ�﷨����ǰ�ȴ���һ���﷨�������󣬲��趨Ҫʹ�õ��ڴ���亯����ʹ����Ϻ�Ҫ�ǵ����١� ParseTrace(stdout, "");��һ������������﷨�������̵ĵ�����Ϣ����Ҫ�ǹ���LRLA(1)����ʱ�ƽ��͹�Լ��һЩϸ�ڡ�Ȼ���Ȼ��ÿ������һ��Token�Ͷ�Ӧ�Ĺ�����Ϣ��Parse�������н�������ȻToken�����ɴʷ����������ɡ�

�������������﷨�ļ������Ǵ�.y��׺���ļ�

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

%��ͷ�Ĳ���ָ����Lemon������������%includeָ����ͷ�ļ����ⲿ�����ݻ�ԭ�ⲻ���Ƶ����ɵ�.c�ļ���ͷ��%token_typeָ��Parse��3�����������ͣ�һ����Token��Ӧ��Ϣ���������ͣ�����ʱÿ�����ʽ���С���Ż��һ���������������Ǵ���ĵ�3����������Ȼ��������%extra_argumentָ����4���������ͣ��Ӷ������ڽ��������д��ݺͽ���������Ϣ��%leftָ�����������ϵģ�ͬʱ��һ��������%left��������ȼ�����һ�иߡ�

�����������﷨���������ˣ��������ı��ʽ����::=���ұ�ʱ��������Լ����ߣ�{}��������ǹ�Լ�����﷨����ʱִ�еĴ��룬expr�����������Token�󶨵Ķ���֮ǰ˵����Parse()�����ĵ�3���������롣���ʽ�ַ�Ϊ�ս���ͷ��ս�����ս���ô�д��ĸ��ʾ����Parse()�ĵ�2���������룬ʹ��ǰҪ�ȶ����

```c
#define PLUS                            1
#define MINUS                           2
#define DIVIDE                          3
#define TIMES                           4
#define NUM                             5
```
0���ܶ��壬�����Ĭ����Ϊ������ֹ�ķ��ţ����ս����Сд��ĸ��ʾ�����﷨����ʽ��Լ�õ���ֻ�ڽ������ڲ�ʹ�á�

�����˵һ�����������Ĺ��̣���������NUM(1)����ʱ���﷨ջ��û���κη��ţ���������Լ��ֱ�Ӱ�NUMѹջ���ɣ��ٴ�����PLUS����ʱ��ջ����NUM���ţ������﷨expr(A) ::= NUM(B)�Ƚ����ԼΪexpr(A)����Ϊ��ǰ����Ĳ��ǽ�����������program ::= expr(A)���ܹ�Լ�����������ڵ�һ�У���ʾ��ǰ�﷨ģ���Ѿ���ȫ������ϣ�expr���ܽ�һ����Լ��PLUS��ջ����ʱջ����expr��PLUS������������NUM(2)��û��Ҫ��Լ�ģ���NUM��ջ����ʱջ����expr��PLUS��NUM�����������������Ȱ�NUM��ԼΪexpr����ʱջ����expr��PLUS��expr����expr(A) ::= expr(B) PLUS expr( C).�﷨������ԼΪexpr���������������ǽ�����������expr��󻹿��Թ�ԼΪprogram����ʱ�������ȫ�����������yy_accept������

��Ȼ��Ҫ��͸��������������̻�Ҫ���.out�ļ���״̬ת��ͼ�� ParseTrace(stdout, "");������ƽ��͹�Լ��Ϣ��������ParseTrace������Ϣ
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

��Ҫ�Խ���������һЩ�޸����������Ҫ����Ҫ��ע�����⼸������

```c
yy_find_shift_action()//ȷ���ƽ��͹�Լ��ת��״̬
yy_shift()//ִ���ƽ�����
yy_reduce()//ִ�й�Լ����
yy_find_reduce_action()//ȷ���Ƿ���Խ�һ����Լ
```

�����⼸��������LEMON�﷨����������(LALR 1����)Դ�����龰��������359ҳ�н��⣬��Ȼֱ�ӿ��ǲ������ģ���Ҫ���lemonԴ���LALR(1)�ķ�������������⡣
