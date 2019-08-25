
ucc�������������ܽ�(5) �м��������

# 1.�������

����������������Ѿ����˰������ź����͵������﷨��������Ҫ����������Ǽ��������﷨����һ�飬��ÿ�����ͱ��ʽ������Ӧ���м���롣

�м����������ַ�����ʽ��ʾ��������Դ��������һ��Ŀ�Ĳ�������һ���������ɡ�Ȼ��ucc�������м����ʱ�Ի�����Ϊ��λ��ÿ������������������м���룬������Ŀ�ͷ����������BB1�������ı�ǩ��

�Ӿ�̬�����������л����鰴˳��ͨ��˫������������һ�𣬵��Ӷ�̬�����������벢�����Ż�������ϵ��µ�ִ�У��������һЩ��ת�����Դ��ڿ�����ת�Ƶ�����������������һ������ߣ�һ���������ж�������߶�Ӧ���Դ����Ŀ�Ľ�㣬�������л��������������߾������һ��������ͼ����ͼ��ֱ�۵ر���˻�����ľ�̬�ṹ�Ͷ�̬�ṹ��     

![���������ͼƬ����](../pic/ucc5.1.PNG)   

����������������Ľṹ��

```c
struct bblock
{
    struct bblock *prev;
    struct bblock *next;
    Symbol sym;
    // successors
    CFGEdge succs;
    // predecessors
    CFGEdge preds;
    struct irinst insth;
    // number of instructions
    int ninst;
    // number of successors
    int nsucc;
    // number of predecessors
    int npred;
    int ref;
};
```
����perv��next�ֱ��ǻ�������Ϊ��̬�ṹʱ��˫�������ǰһ�����ͺ�һ����㡣insth�洢�����м���룬�����м����Ҳ��ͨ��˫�����������ӵġ�succs��preds�洢���ǵ�ǰ�������ڿ�����ͼ�ĺ�������ǰ����㡣������ͼ�Ľṹ�����£�

```c
typedef struct cfgedge
{
    BBlock bb;
    struct cfgedge *next;
}*CFGEdge;
```

����ǰ�����ͺ�����㲻ֹһ�������ǿ����ṹ���л���һ��next��Ա�Ѷ��ǰ�����������㴮����һ����������һ����������ߵĴ��룺

```c
static void AddSuccessor(BBlock bb, BBlock s)
{
    CFGEdge e;

    ALLOC(e);
    e->bb = s;
    e->next = bb->succs;
    bb->succs = e;
    bb->nsucc++;
}

void DrawCFGEdge(BBlock head, BBlock tail)
{
    AddSuccessor(head, tail);
    AddPredecessor(tail, head);
}
```
����Ҫ����һ������ߣ���head�������Ӻ�̽�㣬��tail��������ǰ����㡣��AddSuccessor���������½�һ������ߣ����Ѻ�̽�㸳ֵ��e->bb��Ȼ��ǰ�������ж����̽�㣬ͨ��e->next = bb->succs;���µĺ�̽����뵽����Ȼ����ĺ�̽��������ͷ�����Ѻ�̽���������һ��AddPredecessor����Ҳ�����Ƶġ�

����ʱCurrentBB����ǰ�м�������ɺ�洢�Ļ����飬ͨ��StartBBlock�������ĵ�ǰ�����顣

# 2.���ʽ���м����

�ⲿ�����ݺܶ���ҷ������Ͳ�һһϸ˵�ˣ��������д����Եĵط�˵һ�£���Ӧ�Ĵ�����tranexpr.c�ļ���

## 2.1 TranslatePrimaryExpression

������󷵻�expr->val.p������һ��Symbol���ͣ���������ĺ���CheckPrimaryExpression��֪expr->val.p�洢��ȷʵ�ǻ�������Ӧ�ķ��š�

## 2.2 λ��Ķ�д

������ReadBitField����

```c
static Symbol ReadBitField(Field fld, Symbol p)
{
    int size = 8 * fld->ty->size;
    int lbits = size - (fld->bits + fld->pos);
    int rbits = size - (fld->bits);

    p = Simplify(T(INT), LSH, p, IntConstant(lbits));
    p = Simplify(GetBitFieldType(fld), RSH, p, IntConstant(rbits));

    return p;
}
```
����Ҫ�����½ṹ�����е�λ��b

```c
typedef struct aa{
    int a:8;
    int b:19;
    int c:5;
}AA;
```
�����ṹ����ڴ��������ͼ     
![���������ͼƬ����](../pic/ucc5.2.PNG)     
b��Ӧ������ɫ����sizeΪ������ռ��bit����lbitsΪc��ռ��bit��Ϊ5��rbitsΪa��c����bit����13��Ȼ����Ҫ��ȡb�����ݣ�������lbitsλ��c������Ҫע���������Ʋ���ָͼƬ�еķ��򣩣�Ȼ��������rbits��b����ʼ��ַ�Ƶ���ͷ��Simplify���Լ򵥿�������TryAddValue���һ���м���룬ֻ��������һЩ�Ż����֣�TryAddValue���漰����ʱ���������ں��������k = sad.b;���ɵ��м�������£�

```c
	t22 : sad[0] << 5;
	t23 : t22 >> 13;
	k = t23;
```
��������дλ��WriteBitField�����Ĵ��룬����ؼ��������£�

```c

static Symbol WriteBitField(Field fld, Symbol dst, Symbol src)
{
    int fmask = (1 << fld->bits) - 1;
    int mask = fmask << fld->pos;
    Symbol p;

    //... ...
    //Դ����������8λ
    src = Simplify(T(INT), LSH, src, IntConstant(fld->pos));
    //�����з���������0������λ���Դ����������ɫ������0
    src = Simplify(T(INT), BAND, src, IntConstant(mask));
    //��Ŀ�Ĳ���������ɫ������0������һ����ʱ����
    p = Simplify(T(INT), BAND, dst, IntConstant(~mask));
    //����ʱ������Դ�����������㣬�൱�ڰ�Դ��������ֵ������ɫ����
    p = Simplify(T(INT), BOR, p, src);
    //����ʱ����p��ֵ��Ŀ�Ĳ�����
    GenerateMove(fld->ty, dst, p);

    //... ...
}
```
�����Դ���sad.b = k;Ϊ�������ɵ��м�������£�

```c
	t16 : k << 8;
	t17 : t16 & 134217472;  ������2^27-2^8
	t18 : sad[0] & -134217473;
	t19 : t18 | t17;
	sad[0] = t19;
```

## 2.3 ��ʱ����

�ڷ��ŵ�������ר�Ŷ�������ʱ���������ͣ���֮��Ӧ����ValueDef�ṹ�壬�洢�˶�Ԫ������ʽ�е�Դ��������Ŀ�Ĳ���������ʱ�������ô�����Դ������û�и���ʱ����ֱ�������ö���������һ�Σ������ⲿ�����ݵĽ��ܿ��Կ���C��������������2.5�ڡ�

���������TryAddValue�Ĵ��룬���ע��

```c
Symbol TryAddValue(Type ty, int op, Symbol src1, Symbol src2)
{
    int h = ((unsigned)src1 + (unsigned)src2 + op) & 15;
    //ͨ��hash���ҵ��洢��ʱ�������ʽ��Ͱ
    ValueDef def = FSYM->valNumTable[h];
    Symbol t;
 
    //���Ҹñ��ʽ�Ƿ���ڶ�Ӧ����ʱ����  
    while (def)
    {
        if (def->op == op && (def->src1 == src1 && def->src2 == src2))
            break;
        def = def->link;
    }
    //������ڣ��Ͳ��ؼ��㣬ֱ����def->dst
    if (def && def->ownBB == CurrentBB && def->dst != NULL)
        return def->dst;

new_temp:
    //�½���ʱ���������ɸ�ֵ���
    t = CreateTemp(ty);
    GenerateAssign(ty, t, op, src1, src2);
    //Ȼ�����ʱ������def��¼�����ű���
    def = AsVar(t)->def;
    def->link = FSYM->valNumTable[h];
    FSYM->valNumTable[h] = def;
    return t;
}
```

## 2.4 ��·����

��������c = a||b�� c = a&&b�ı��ʽ���Ƕ��ұ�����ֵ��Ȼ���ٸ�ֵ����ߣ������ȼ���a��ֵ�������ʱ�Ѿ���ȷ���������ʽ��ֵ����ôb��û�б�Ҫ�������㡣�����ʵ����TranslateBranchExpression�����Դ��������

```c
static Symbol TranslateBranchExpression(AstExpression expr)
{
    BBlock nextBB, trueBB, falseBB;
    Symbol t;

    t = CreateTemp(expr->ty);
    nextBB = CreateBBlock();
    trueBB = CreateBBlock();
    falseBB = CreateBBlock();

    TranslateBranch(expr, trueBB, falseBB);
    StartBBlock(falseBB);    
    GenerateMove(expr->ty, t, IntConstant(0));    
    GenerateJump(nextBB);
    StartBBlock(trueBB);
    GenerateMove(expr->ty, t, IntConstant(1));
    StartBBlock(nextBB);

    return t;
}

```

������k = k||2���������˵���������ɸ�ֵ���ʽTranslateAssignmentExpression�ĺ�������� k||2ʱ�����TranslateBranchExpression��������TranslateBranch�����л�������´���

```c
case OP_OR:
    rtestBB = CreateBBlock();
    //�����м���룬���k��0����������trueBB
    TranslateBranch(expr->kids[0], trueBB, rtestBB);
    //���ڽ���rtestBB������
    StartBBlock( rtestBB);
    //�����м���룬���2Ϊ��0��������trueBB
    TranslateBranch(expr->kids[1], trueBB, falseBB);
    break;
default:
    src1 = TranslateExpression(expr);
    GenerateBranch(ty, trueBB, JNZ, src1, NULL);
```
TranslateBranch���غ������falseBB�����飬��ʱ���ɵĴ�����ζ�� k||2���ʽ�������߶�Ϊ0����ʱ��0��ֵ��k����nextBB�����飬����������falseBB��nextBB֮������trueBB�Ĵ��룬������k = k||2��Ӧ���ɵ��м����

```java
	if (k) goto BB3;
BB1:    // rtestBB
 	goto BB3;
 	//���ﱾ����falseBB���Ż�֮��BB2��û����ʾ����
	k = 0;
	goto BB4;
BB3: //trueBB
 	k = 1;
BB4://nextBB
 	...
```
## 2.5 ƫ�Ƶ�ַ

�ڴ���������ش���ʱҪ����ƫ�Ƶ�ַ����Ҫ�� TranslateArrayIndex��Offset���������

```c
static Symbol TranslateArrayIndex(AstExpression expr)
{
    do
    {
        if (p->kids[1]->op == OP_CONST)
        {
            coff += p->kids[1]->val.i[0];
        }
        else if (voff == NULL)
        {
            voff = TranslateExpression(p->kids[1]);
        }
        else
        {
            voff = Simplify(voff->ty, ADD, voff, TranslateExpression(p->kids[1]));
        }
        p = p->kids[0];
        // see  case OP_DEREF in CheckUnaryExpression()
    }while (p->op == OP_INDEX);
    
    addr = TranslateExpression(p);
    dst = Offset(expr->ty, addr, voff, coff);
    return expr->isarray ? AddressOf(dst) : dst;
}
static Symbol Offset(Type ty, Symbol addr, Symbol voff, int coff)
{
    if (voff != NULL)
    {
        voff = Simplify(T(POINTER), ADD, voff, IntConstant(coff));
        addr = Simplify(T(POINTER), ADD, addr, voff);
        return Deref(ty, addr);
    }
    ... ...
}
```
��int a[5][3]; a[k][2] = 7;Ϊ�������Ȼ����������Ԫ�ص��﷨�����õ�voffΪk*12��coffΪ8��Ȼ��addr��ȡ����a�ĵ�ַ����offset������Ѱ�12*k����8�õ�t9���õ����յ�ַt10������ȡ����ַ�еĴ��������Deref��ɣ����ɵ��м��������

```c
t7 : k * 12;
t8 :&a;
t9 : t7 + 8;
t10 : t8 + t9;
*t10 = 7;
```
## 2.6 ��������

�������õ����ɴ���ΪGenerateFunctionCall() �����������м����������ΪCALL������ֵΪrecv��������faddr����������args

```c
inst->opcode = CALL;
inst->opds[0] = recv;
inst->opds[1] = faddr;
inst->opds[2] = (Symbol)args;
AppendInst(inst);
```

## 2.7  ����ת��

��ucc�а�ÿһ�ָ�ʽת�����������һ�����������enum OPCode����ʽ��ʾ������intתfloat�������ΪCVTI4F4��
OPCODE(CVTI4F4, "(float)(int)",         Cast)
���ݱ��ʽ�õ�opcode��ֵ��Ȼ������ǿ��ת��������м���룬��src��ֵdst
```c
dst = CreateTemp(ty);	
GenerateAssign(ty, dst, opcode, src, NULL);
return dst;
```
# 3.�����м����

## 3.1 if��������

��������

```c
nextBB = CreateBBlock();
trueBB = CreateBBlock();
falseBB = CreateBBlock();

//�����3������trueBBû��
TranslateBranch(Not(ifStmt->expr), falseBB, trueBB);

StartBBlock(trueBB);
TranslateStatement(ifStmt->thenStmt);
GenerateJump(nextBB);

StartBBlock(falseBB);
TranslateStatement(ifStmt->elseStmt);
StartBBlock(nextBB);
```
if�������ɻ��Ǻܺ����ģ��ڵ�ǰ����飬���ifStmt->expr����������ת��falseBB�������������ִ�У���ʱӦ��������ģ���������trueBB�Ĵ��룬ִ����������ת��nextBB�Ĵ��룬��������falseBB�Ĵ��룬�������nextBB�Ĵ��롣

## 3.2 for��������

for��䲻����⣬˵��������ע��

```c
static void TranslateForStatement(AstStatement stmt)
{
    AstForStatement forStmt = AsFor(stmt);

    forStmt->loopBB = CreateBBlock();
    forStmt->contBB = CreateBBlock();
    forStmt->testBB = CreateBBlock();
    forStmt->nextBB = CreateBBlock();

    //forѭ����ʼ��
    if (forStmt->initExpr)
    {
        TranslateExpression(forStmt->initExpr);
    }
    //����forѭ����ֹ�ж�
    GenerateJump(forStmt->testBB);
    //����ѭ�������
    StartBBlock(forStmt->loopBB);
    TranslateStatement(forStmt->stmt);
    //���ɼ���ֵ����
    StartBBlock(forStmt->contBB);
    if (forStmt->incrExpr)
    {
        TranslateExpression(forStmt->incrExpr);
    }
    //����ѭ����ֹ�жϴ���
    StartBBlock(forStmt->testBB);
    //���ѭ����ֹ��������nextBB����������loopBB
    if (forStmt->expr)
    {
        TranslateBranch(forStmt->expr, forStmt->loopBB, forStmt->nextBB);
    }
    else
    {
        GenerateJump(forStmt->loopBB);
    }

    StartBBlock(forStmt->nextBB);
}
```

�������������break���룬�ȴ�ջ��ȥ��break��Ӧ��forѭ�����﷨�����brkStmt->target��Ȼ��������ת��nextBB�Ĵ��룬GenerateJump(AsLoop(brkStmt->target)->nextBB);��������forѭ��

## 3.3 switch��������

switch���������������ɵ�����ӵģ����Ȼ��switch�жϱ��ʽ��sym��Ȼ�����ÿһ��case��䣬������뵽Ͱ�����ݽṹ����ת��case���ʹ�õ�����ת��������case����block���һ�ű�����caseֵ��ñ��ƫ�Ƶ�ַ��Ȼ��ֱ����ת���õ�ַ�����ֻ������caseֵ�ܶȽϸ�ʱ�����������case��1��3��4��200�����������ʹ����ת��ͻ��˷Ѵ����ռ䣬���Խ��ܶȽϸߵĻ�����һ��Ͱ���1��3��4���һ��Ͱ��200���һ��Ͱ��

```c
if (bucket && (bucket->ncase + 1) * 2 > (val - bucket->minVal))
```
��case������*2�������ֵ����Сֵ��˵���ܶȽϸߣ���case���뵽Ͱ������½�һ��Ͱ��case��ֵ�ǰ���С�����˳�����еģ���case����������ʱ�����ڵ�ǰͰ�����ֵ����һ��Ͱ����Сֵ����ʱ����MergeSwitchBucket����һ��Ͱ�ϲ�

���������ǵ���TranslateSwitchBuckets���ÿһ��Ͱ�����м���롣���ȴ��м���Ǹ�Ͱ��ʼ����Ͱ�������case���������dstBBs�Ȼ�����choiceҲ���Ǵ�������switch�жϱ��ʽ��symֵȷ��choice�Ƿ������Ͱ��ȡֵ��Χ�����������������ת���룬�������������ת����롣������ͨ���ݹ���������Ͱ����ת���롣

�������ٸ����ӣ����µ�switch����

```c
 switch(k+1){
 case 5:
 case 2:
    k++;
    p++;
    break;
 case 1:
    k++;
    break;
  case 201:
    k += 5;
    break;
  case 203:
    k += 7;
    break;
 defalut:
    break;
 }
```

���ɵ��м�������£�

```c
	t29 : k + 1;
	if (t29 < 1) goto BB22;
BB13:
 	if (t29 > 5) goto BB15;
BB14:
 	t30 : t29 - 1;//��һ��Ͱ
	goto (BB19,BB18,BB22,BB22,BB18,)[t30];
BB15:
 	if (t29 < 201) goto BB22;
BB16:
 	if (t29 > 203) goto BB22;
BB17:
 	t31 : t29 - 201;//�ڶ���Ͱ
	goto (BB20,BB22,BB21,)[t31];
BB18:
 	++k;
	t35 : p + 4;
	p = t35;
	goto BB22;
BB19:
 	++k;
	goto BB22;
BB20:
 	t38 : k + 5;
	k = t38;
	goto BB22;
BB21:
 	t39 : k + 7;
	k = t39;
BB22:
```
