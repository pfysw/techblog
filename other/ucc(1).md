������������ϵͳ�����ݿ��Ǽ���������е�����������������������Ӧ��������ǽ������������������֮�ϣ���Щ���򾭹�ǰ���ǲ��ϴ�ĥ�����Ѿ���÷ǳ����죬�ܶ��漰���Ĵ��뼼�����Ǿ����еľ��������԰�����3�����������Դ���ˮƽ����߻��кܶ�����������ܹ����ִ������������ܹ��и��������˽⡣

������Ҫ�о�һ���߼�����ϵͳ���������ϵͳ��C����Ϊ�����������Ȼ�漰��C���ԵĽ������⣬����C���Ա������Ǳ���Ҫ��ѧϰ�ģ�C���Ա������ǰ�C����ת���ɻ�����ԣ��Ҵ����ڱ������Ļ���������һЩ�����߼��Ķ��ԣ�Ȼ������Щ�߼�����Ϊ��������һ������ϵͳ��

ĿǰC���Ա������Ƚϳ������gcc��clang��llvm��������Щ��̫�Ӵ��ˣ��������и��ּ����Ż��ϵĿ�����������û�о���Ҳû��ʱ�����յġ�����Ҫѧϰ���Ǳ�������һЩ����˼�룬����ֻҪ��һЩС�͵Ŀ�ԴC���Ա������Ϳ����ˣ���ʵ����ͦ��ģ���lcc��ucc��tcc��8cc�ȵȡ�

�����ڴ����ucc����ѧ�����������ǹ���д�ģ��ɶ��Է������ıȽϺã����д��ж�ucc���˷ǳ���ϸ�Ľ�������
https://blog.csdn.net/sheisc/article/category/2813001
���һ�д��һ����С�c����������������������˵Ҫ�������ٶ�ѧ���������һЩ����֪ʶ��ucc�����˵��һ���ȽϺõ�ѡ��

# 1.�����ܹ�
��ʵucc������������һ�������ı������������ı���������Ԥ��������C�����������������������uccֻ�а�C���Ա���ɻ�����Ե���һ���֣�����ͼ��ʾ��     
![���������ͼƬ����](../pic/ucc1.1.PNG)     
��ucc��Դ�룬���Կ�������ucc��ucl�����ֹ��̣���Ϊucc��������ֻ����.i�ļ�����.s�ļ��Ĺ�������ucl�Ĵ�������ɣ�����Ϊ�˴�.c�ļ����ɿ�ִ���ļ���Ҫ��ucc�е���gcc��Ԥ����������������������������������������Ĺ��̡�

���ڼ���Ҫ����һ��test.c�ļ�����������������ucc -E -v test.c������Ԥ�������ļ�test.i,Ȼ��������ucl -ext:.s test.i�����Է�����������Ĺ��̡�

# 2.�ʷ�����
���ڴ�ucl.c�ļ��е�main����������������������̣���������

```c
	i = ParseCommandLine(argc, argv);

	SetupRegisters();
	SetupLexer();
	SetupTypeSystem();
	for (; i < argc; ++i)
	{
        Compile(argv[i]);
	}
```
�����ǼĴ����ʹʷ�����������س�ʼ����Ȼ��ͨ��forѭ����Compile����ÿһ��.c�ļ���

֮����ParseTranslationUnit()����ɴʷ��������﷨�������������ȶ�ȡ�����ļ�������Input��ȫ�ֱ��������һ��struct input�ṹ�����͵ı������������£�

```c
struct input
{
    //�ļ�����Ԥ�����������.h�ļ�չ����.i�ļ�
    //���� # 667 "/usr/include/stdio.h" 3 4
    //���¼ͷ�ļ����Ͷ�Ӧ����
    char *filename;
    unsigned char *base;//�����ļ�
    //��ǰ�������ĵ��ʵ���һ����ַ
    //���� int aa[4]; 
    //������ڽ�����aa����ôcursorָ��[4];
    unsigned char *cursor;
    unsigned char *lineHead;//ÿһ�������;Ϊ��λ
    int line;// ��ǰ��.i�ļ��е�����
    void* file;//�ļ�������
    void* fileMapping;
    unsigned long size;//�ļ���С
};
```
���⻹һ��TokenCoordȫ�ֱ�����Input�Ĳ��䣬������ͷ�ļ��е���������C���������Ե���Ϊ��λ�ģ���������ע������int aa[4];����һ�����ӣ��������ĵ��ʷֱ�Ϊint��aa��[��]��4��; ��ucc����NEXT_TOKENȡ��һ�����ʷ���CurrentToken�Ȼ���ٽ����﷨������
```c
#define NEXT_TOKEN  CurrentToken = GetNextToken();
```

CurrentToken��һ��ö���ͱ���������һ�����ʵ����ԣ�������ÿ�������ַ�������ÿ���ؼ��ֶ�����һ����Ӧ��ö�ٱ���������������token�Ķ��壺

```c
enum token
{
    TK_BEGIN,
#define TOKEN(k, s) k,
#include "token.h"
#undef  TOKEN
};
//keywords
TOKEN(TK_AUTO,      "auto")
TOKEN(TK_EXTERN,    "extern")
TOKEN(TK_REGISTER,  "register")
TOKEN(TK_STATIC,    "static")
TOKEN(TK_TYPEDEF,   "typedef")
... ...
```

C�����ж���ı�ʶ��Tokenͳһ��TK_ID����ʾ����ȡ��һ�����ʵ�Token�����ӵ��﷨���Ľ���

# 3.�﷨����

�ʷ�������CԴ�ļ���ȡ�õ��ʵ�token�����﷨���������Щ������ӵ��﷨�����档AstNode���﷨�����������Ľṹ�壺

```c
#define AST_NODE_COMMON   \
    int kind;             \
    struct astNode *next; \
    struct coord coord;

typedef struct astNode
{
	AST_NODE_COMMON
} *AstNode;
```
����kind����������͡�next������ͬ��ı��ʽ������һ��������;�ͣ���������䡣coord���������ļ��������Ӧ���������������͵Ľ������kind��ͬ����Ҫ����Ϣ��ͬ���ṹ����������죬��������AstNode�Ļ�������������ġ�

���﷨�����Ĺ����У����Ȼ��Ƚ���һ��NK_TranslationUnit���͵ĸ���㣬�ý�㶨������
```c
struct astTranslationUnit
{
	AST_NODE_COMMON
	AstNode extDecls;
};
```
���������һ��.c�ļ���ÿһ��ȫ���������ͺ���������һ������ͼ��ʾ����ֱ�۸��ܵ��﷨�����ص�       
![���������ͼƬ����](../pic/ucc1.2.PNG)      
�������£�

```c
AstTranslationUnit ParseTranslationUnit(char *filename)
{
    AstTranslationUnit transUnit;
    AstNode *tail;

    //...

    while (CurrentToken != TK_END)
    {
        *tail = ParseExternalDeclaration();
        tail = &(*tail)->next;
        SkipTo(FIRST_ExternalDeclaration,
                "the beginning of external declaration");
    }
}
```

ÿ�������Է�Ϊ�������ͱ��ʽ�������ͣ����ʽ���ɸ�ֵ�����������ɡ�������Ĵ������˵��

```c
extern int c��
int test(int a)
{	
    int b = 1;
    a = b+sizeof(int);
    if(a>0){
    	c++;
    }
	return 0;
}
```
�������������
>extern int c�� 
int test(int a){}  
 int b = 1��

���ʽ�����
>    a = b+sizeof(int);
    if(a>0){
    	c++;
    }
	return 0;

������2�����͵Ľ����ٽ������﷨�������ݹ�ֱ���﷨������е�Ԫ��Ϊ����Ϊ�ˡ�
## 3.1 �������
�������Ľ�������ParseCommonHeader()��������ɣ�ParseDeclarationSpecifiers()��������������ؼ�����int��static��typedef�ȣ������еı�ʶ����ParseInitDeclarator()����������������ͨ��nextָ��������һ��

```c
while (CurrentToken == TK_COMMA)
{
    NEXT_TOKEN;
    *tail = (AstNode)ParseInitDeclarator();
    tail = &(*tail)->next;
}
```
����������п��ܻ��ʼ����int a = 3���ڽ�������ٰ�=����������뵽�﷨����

```c
static AstInitDeclarator ParseInitDeclarator(void)
{
    AstInitDeclarator initDec;

    CREATE_AST_NODE(initDec, InitDeclarator);

    initDec->dec = ParseDeclarator(DEC_CONCRETE);
    if (CurrentToken == TK_ASSIGN)//����=˵��Ҫ��ʼ��
    {
        NEXT_TOKEN;
        initDec->init = ParseInitializer();
    }

    return initDec;
}
```
ParseDeclarator���������������ʶ��������Ĳ���DEC_CONCRETE˵����ʶ������ʡ�ԣ���Щ�ط���������䲻���б�ʶ������sizeof��int��,��ʱ����DEC_ABSTRACT������Щ�ط����п��ޣ�������������ʱ��int f(int,int); ,��ʱ�����ParseDeclarator(DEC_CONCRETE|DEC_ABSTRACT);

�������п��ܻ�����ָ��������*����Ϊһ���������﷨��������

```c
static AstDeclarator ParseDeclarator(int kind)
{
    if (CurrentToken == TK_MUL) //ָ��������*
    {
        AstPointerDeclarator ptrDec;

        CREATE_AST_NODE(ptrDec, PointerDeclarator);

        //...

        //�ݹ���ü�������
        ptrDec->dec = ParseDeclarator(kind);

        return (AstDeclarator) ptrDec;
    }
    //����������ָ�������µ�������ʶ��
    return ParsePostfixDeclarator(kind);
}
```
��ParsePostfixDeclarator()�����л����ParseDirectDeclarator()�������ı�ʶ�����ʣ���ʶ���������[]�����飬����()�Ǻ����βΣ�����Ҫ����������

���������������typedefʱ����ͨ��CheckTypedefName()�����������ı�ʶ�����뵽TDName�б��м�¼������Ȼ������TK_ID�ı�ʶ��ʱ����ͨ��IsTypedefName()ȷ��֮ǰ��û��ͨ��typedef����������C������ÿ��{}����һ��������ucc����Level��ǵ�ǰ��������������ÿ����һ��Level ��1����ʱ�����ڲ���������ı������ⲿtypedef�����ı�ʶ�������ˣ���Ҫ�ӵ�OverloadNames�б������ء�

���ͨ��һ��ͼ��ֱ�۸���һ���������Ľ������̣�ͼƬȡ�ԡ�C������������    
![���������ͼƬ����](../pic/ucc1.4.PNG)     

## 3.2 ���ʽ���

���ʽ���ֻ�����ں����У���ParseCommonHeader()�������������󣬻��������ı�ʶ���Ƿ��Ǻ���������Ǻ������һ��������{}�������ķ�����䣬��ParseCompoundStatement()����ɣ������ڲ�Ҳ��Ϊ�������ͱ��ʽ��䣬�ֱ���ParseDeclaration()��ParseStatement()������������ɡ�

���ʽ��������кܶ࣬�縳ֵ��䣬if��䣬while���ȵȣ����ݵ�ǰ��ͬ��token����Ӧ������

```c
static AstStatement ParseStatement(void)
{
    switch (CurrentToken)
    {
    case TK_ID:
        return ParseLabelStatement();

    case TK_IF:
        return ParseIfStatement();

    case TK_SWITCH:
        return ParseSwitchStatement();

    case TK_WHILE:
        return ParseWhileStatement();

    case TK_DO:
        return ParseDoStatement();

    case TK_FOR:
        return ParseForStatement();

    case TK_GOTO:
        return ParseGotoStatement();

        //...

    default:
        return ParseExpressionStatement();
    }
}
```
���ʽ�Ľ����struct astExpression�ṹ�嶨��

```c
struct astExpression
{
    AST_NODE_COMMON
    Type ty;
    int op :16;//���ʽ��Ӧ�Ĳ�����
    int isarray :1;
    int isfunc :1;
    int lvalue :1;
    int bitfld :1;
    int inreg :1;
    int unused :11;
    //��¼2Ԫ���ʽ��ǰ��2���ӱ��ʽ
    struct astExpression *kids[2];
    union value val;
};
```

�������ǹؼ��ֵı�ʶ�������ͨ��ParseLabelStatement()�鿴�Ƿ��Ǳ�ǩ��䣨����error���������������ôͨ��ParseExpressionStatement()���������������Ƚ�����ֵ��䣬�������ż����ݹ���ã�����һ�����ʽ������comaExpr->kids[1]��

```c
AstExpression ParseExpression(void)
{
    AstExpression expr, comaExpr;

    expr = ParseAssignmentExpression();
    while (CurrentToken == TK_COMMA)
    {
        CREATE_AST_NODE(comaExpr, Expression);

        comaExpr->op = OP_COMMA;
        comaExpr->kids[0] = expr;
        NEXT_TOKEN;
        comaExpr->kids[1] = ParseAssignmentExpression();

        expr = comaExpr;
    }

    return expr;
}
```

ucc����Ϊ���еı�ʾʽ��䶼�Ǹ�ֵ���ʽ������и�ֵ����= *= /= %= += -= <<= >>= &= ^= |=�򵱳ɸ�ֵ���ʽ��������û�и�ֵ�ŵ����������ʽ����

```c
AstExpression ParseAssignmentExpression(void)
{
    AstExpression expr;

    expr = ParseConditionalExpression();
    if (CurrentToken >= TK_ASSIGN && CurrentToken <= TK_MOD_ASSIGN)
    {
        //...
    }

    return expr;
}
```
��ֵ����ߵı��ʽ���ᵱ���������ʽ���������ʽ��2Ԫ���ʽ�ͣ���ɣ�û�У��򵱳�2Ԫ���ʽ����

```c
static AstExpression ParseConditionalExpression(void)
{
    AstExpression expr;

    expr = ParseBinaryExpression(Prec[OP_OR]);
    if (CurrentToken == TK_QUESTION)
    {
        //...
    }

    return expr;
}
```
�������ƣ�2Ԫ���ʽ��һԪ���ʽ��2Ԫ�������ɣ�һԪ���ʽ��ParseUnaryExpression()��������1Ԫ���ʽ��ǰ��һԪ�������++ -- & * + - ! ~����û��ǰ���������һԪ���ʽ��ɣ�������ParsePostfixExpression()������,��ʱû�к�׺�ı��ʽ��ParsePrimaryExpression()��������������к�׺.��->��--��++�Ȼ�Ҫ���½�һ������ǰ������ı�ʾ���Ϻ�׺���������������ŵ����������ݹ�������Ͳ�����ϸ�����ˡ���Ҫע�����++�������ǰ�úͺ���ʱ�ı�ʾ�ǲ�ͬ�ģ�ǰ����OP_PREINC��ʾ������OP_POSTINC��ʾ��

�ڽ���2Ԫ���ʽʱ����Ҫע�����ȼ����⣬����a<<5+3,�ڽ�����5+3ʱ������+�����ȼ���<<�ߣ�����5+3Ҫ��Ϊһ�����壬���ڱ��ʽa*b+3�У�+���ȼ�û��\*�ߣ�bҪ����Ϊ\*���Ҷ˿�ʼ����Ȼ���ٽ���+�š�

```c
static AstExpression ParseBinaryExpression(int prec)
{
    AstExpression binExpr;
    AstExpression expr;
    int newPrec;

    expr = ParseUnaryExpression();
    /// while the following binary operater's precedence is higher than current
    /// binary operator's precedence, parses a higer precedence expression
    while (IsBinaryOP(CurrentToken) && (newPrec = Prec[BINARY_OP]) >= prec)
    {
        CREATE_AST_NODE(binExpr, Expression);

        binExpr->op = BINARY_OP;
        binExpr->kids[0] = expr;
        NEXT_TOKEN;
        binExpr->kids[1] = ParseBinaryExpression(newPrec + 1);

        expr = binExpr;
    }

    return expr;

}
```
�����Ȼ�����﷨����ͼƬ��ֱ�۸����½����Ĺ��̣�ͼƬȡ�ԡ�C������������    
![���������ͼƬ����](../pic/ucc1.5.PNG)      
![���������ͼƬ����](../pic/ucc1.6.PNG)     

# �ڴ����

���һ���ڴ���䣬��ucc�������л���ڴ�ķ�������Խ���һ��������й����������ܹ���3���ѣ��ֱ�Ϊ��

 - ProgramHeap
 ������ʼ���Ĵ����Ĺ���ͻ���������ϵͳ��
 - FileHeap
 Ϊÿ��C�ļ������﷨���ͷ��ű��á�
 - StringHeap
 ���C�ļ����ַ����ͱ�־�������ơ�

ÿ��������һ��ͷ���ڵ���ڴ�ڵ���ɵ�����ͷ���ڵ㲻�����ڴ棬���ֱ�ָ����һ���ڵ���������һ���ڵ㣬����ṹ����ͼ��ʾ��     

![���������ͼƬ����](../pic/ucc1.3.PNG)          

�����ڴ�ʱ��HeapAllocate��������ɣ�ÿһ�������ڴ�ʱ�����½�һ���ڵ㲢������Ӧ�Ŀռ䣬�ڽ�����һ���ļ���FileHeap�ѻ����³�ʼ����������ڴ沢�����ͷţ�����FreeBlocksָ���2����㣬�ٴ������ڴ�ʱ���ȴ�FreeBlocks������û���㹻�ռ�Ľ�㡣�����ƺ������⣬���һ��ʼ����һ��Ƚϴ�Ŀռ䣬��FreeBlocks�Ƶ��˺��棬Ȼ��FreeBlocks��û�����ص�ǰ�棬����ǰ��Ŀռ䶼�˷��ˡ�
