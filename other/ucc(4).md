
ucc�������������ܽ�(4)���ʽ�������

# 1.�������

֮ǰ���﷨����ʱ�����˳����﷨����������ʱΪÿ�����Ž���������ϵͳ����������Ҫ�������ʽ������ÿ����㣬����Щ���������Ͱ��ڶ�Ӧ�Ľ�㣬����Ӧ�Ľ�������淶�ļ�顣

ÿ���������ļ�鶼��һ��������Ϊ��λ�����������ַ�Ϊ�ֲ����������ļ���ִ�����ļ��

```c
AstStatement CheckCompoundStatement(AstStatement stmt)
{
    AstCompoundStatement compStmt = AsComp(stmt);
    AstNode p;

    compStmt->ilocals = CreateVector(1);
    p = compStmt->decls;
    while (p)
    {
        CheckLocalDeclaration((AstDeclaration) p, compStmt->ilocals);
        p = p->next;
    }
    p = compStmt->stmts;
    while (p)
    {
        CheckStatement((AstStatement) p);
        p = p->next;
    }

    return stmt;
}
```
CheckStatement�и��ݱ��ʽ������stmt->kind��ȷ�������ͨ���ʽ���Ǵ��йؼ��ֵ����

```c
static AstStatement (*StmtCheckers[])( AstStatement) =
{
    CheckExpressionStatement, // the index is (NK_ExpressionStatement - NK_ExpressionStatement), 0
        CheckLabelStatement,// the index is ( NK_LabelStatement - NK_ExpressionStatement), 1
        CheckCaseStatement,// 2
        CheckDefaultStatement,// 3
        CheckIfStatement,// 4
        CheckSwitchStatement,// 5
        CheckLoopStatement,
        CheckLoopStatement,
        CheckForStatement,
        CheckGotoStatement,
        CheckBreakStatement,
        CheckContinueStatement,
        CheckReturnStatement,
        CheckLocalCompound
};

static AstStatement CheckStatement(AstStatement stmt)
{
    return (*StmtCheckers[stmt->kind - NK_ExpressionStatement])(stmt);
}
```

CheckExpressionStatement���ֻ����CheckExpression�����������еı��ʽ��鶼Ҫ��������룬���ݱ��ʽ���������ȷ��������ͣ��Ǹ�ֵ���㻹��˫Ŀ������ʽ���ǵ�Ŀ������ʽ

```c
AstExpression CheckExpression(AstExpression expr)
{
    return (* ExprCheckers[expr->op])(expr);
}
```

�����˫Ŀ������ʽ����ô�ݹ������������߱��ʽ���ұ߱��ʽ������������������ٽ�һ��ȷ���������

```c
static AstExpression (*BinaryOPCheckers[])( AstExpression) =
{
    CheckLogicalOP, // "||"
        CheckLogicalOP,// "&&"
        CheckBitwiseOP,// |
        CheckBitwiseOP,// ^
        CheckBitwiseOP,// &
        CheckEqualityOP,// ==
        CheckEqualityOP,// !=
        CheckRelationalOP,// >
        CheckRelationalOP,// <
        CheckRelationalOP,// >=
        CheckRelationalOP,// <=
        CheckShiftOP,//  <<
        CheckShiftOP,// >>
        CheckAddOP,// +
        CheckSubOP,// -
        CheckMultiplicativeOP,// *
        CheckMultiplicativeOP,//  /
        CheckMultiplicativeOP// %
};

static AstExpression CheckBinaryExpression(AstExpression expr)
{

    expr->kids[0] = Adjust(CheckExpression(expr->kids[0]), 1);
    expr->kids[1] = Adjust(CheckExpression(expr->kids[1]), 1);
    return (*BinaryOPCheckers[expr->op - OP_OR])(expr);
}
```

�������͵ı��ʽ���Ҳ�����ƣ����Ƕ�expr->kids[0]��expr->kids[1]���еݹ飬��󵽴�ֻ��һ�����ŵ�ԭ�ӱ��ʽ��Ȼ�����CheckPrimaryExpression�����ű�ȷ���ñ��ʽ���͡�

����if��while�ȹؼ�����䣬���ȼ��С�����ڶ�Ӧ���жϱ��ʽ��Ȼ���ٶԽ�������ִ���������еݹ��飬���´����Ƕ�while���ļ��

```c
static AstStatement ParseWhileStatement(void)
{
    AstLoopStatement whileStmt;

    CREATE_AST_NODE(whileStmt, WhileStatement);

    NEXT_TOKEN;
    Expect(TK_LPAREN);
    whileStmt->expr = ParseExpression();
    Expect(TK_RPAREN);
    whileStmt->stmt = ParseStatement();

    return (AstStatement) whileStmt;
}
```
# 2.������
���ڽ�����������̫ϸ���ˣ�������һ������ֻ��һЩ���͵����Ӿ��������
## 2.1 ����δ����
��CheckPrimaryExpression()�����м����ű�ʱ���鵽

```c
    p = LookupID(expr->val.p);
	if (p == NULL)
	{
        Error(&expr->coord, "Undeclared identifier: %s", expr->val.p);
	}
```
## 2.2 �ṹ���Ա��ƫ�Ƶ�ַ

```c
typedef struct aa{
    char a;
    int b;
}AA;
int a[sizeof(AA)+(int)(&((AA*)0)->b)] = {0};
```
���������Ĵ���ucc��������ͨ�����ģ��ᱨ�����鳤�Ȳ��ǳ����Ĵ���

```c
if (arrDec->expr)
{
    if ((arrDec->expr = CheckConstantExpression(arrDec->expr)) == NULL)
    {
        Error(&arrDec->coord, "The size of the array must be integer constant.");
    }
}
```
��ʵ�����ǿ���ͨ������ģ�(int)(&((AA*)0)->b)��һ��������ucc��û�ж�����㣬��΢����һ�¾Ϳ����ˣ���ָ����������ʱ�����ж����Ӧ�Ľṹ���Ƿ���0����������Ǿͽ��������ʽתΪ�ó�Աƫ�Ƶ�ַ�ĳ���

```c
if( expr->kids[0]->op == OP_CONST &&
        expr->kids[0]->val.i[0]==0 &&
        expr->op==OP_PTR_MEMBER )
{
    expr->val.i[0] = fld->offset;
    expr->op = OP_CONST;
}
else
{
    expr->val.p = fld;
}
```
ͬʱ��CheckUnaryExpression()�����м��ȡ��ַ���ʽʱ������ӱ��ʽ�ǳ������ñ��ʽҲҪ��ȡ��ַ���ʽת��Ϊ�������ʽ

```c
if( expr->kids[0]->op == OP_CONST )
{
    expr->op = OP_CONST;
    expr->val.i[0] = expr->kids[0]->val.i[0];
}
```

## 2.3 ��ά����

������int a[5][3],����ʱa[4][2] = 1Ϊ����һ��ʼ��������ʱ��õ����¶�����     
![���������ͼƬ����](../pic/ucc4.1.PNG)     
��������������������һ��tyDrvList�����̶���������a�����ͣ�
![���������ͼƬ����](../pic/ucc4.2.PNG)      
a[4][2] ���ڱ��ʽ�н������﷨������     
![���������ͼƬ����](../pic/ucc4.3.PNG)      
�ڱ��ʽ�����ͼ���У�����CheckExpression()�ݹ飬�ᵽa�������������һ����ά��������ͣ����ͳ���Ϊ60������鵽kid[1]�������ǣ���a[4]�����Ϊa���͵������ͼ�`expr->ty = expr->kids[0]->ty->bty;` �������ƣ�a[4][2]��������һ��int����

����������Adjust����

```c
AstExpression Adjust(AstExpression expr, int rvalue)
{
    int qual = 0;
    if (rvalue)
    {
        qual = expr->ty->qual;
        expr->ty = Unqual(expr->ty);
        expr->lvalue = 0;
    }

if (expr->ty->categ == FUNCTION)
{
    ... ...
}
else if (expr->ty->categ == ARRAY)
{
    expr->ty = PointerTo(Qualify(qual,expr->ty->bty));
    expr->lvalue = 0;
    expr->isarray = 1;
}
```
�����������������:

```c
case OP_INDEX:
    expr->kids[0] = Adjust(CheckExpression(expr->kids[0]), 1);
    expr->kids[1] = Adjust(CheckExpression(expr->kids[1]), 1);
```
�ڸ�ֵ����ʱ��������

```c
	expr->kids[0] = Adjust(CheckExpression(expr->kids[0]), 0);
	expr->kids[1] = Adjust(CheckExpression(expr->kids[1]), 1);

	if (! CanModify(expr->kids[0]))
	{
        Error(&expr->coord, "The left operand cannot be modified");
	}
```
�������������������ô��ֵ���ʱ��Adjust()�����л����ֵ��0����ͼ������a��a[4]�����������������������ֵ����CanModify�б������Ƿ�����a[4][2]�ڼ���õ�����a[4]�������͵������ͣ�Ϊint���ͣ����Կ�����Ϊ��ֵ����ֵ����������������������޶���const����ôconst�ᱻ��ӵ�������������У���ʱ�Ͳ�����Ϊ��ֵ�ˣ���Ϊ CanModify���������

```c
static int CanModify(AstExpression expr)
{	
	return (expr->lvalue && ! (expr->ty->qual & CONST) && 
	        (IsRecordType(expr->ty) ? ! ((RecordType)expr->ty)->hasConstFld : 1));
}
```
## 2.4 ָ������

ָ�������ж������ͣ����䱾���϶���ȡ�ӱ��ʽ�������ͣ��������£�

```c
ty = expr->kids[0]->ty;//�ӱ��ʽ������
expr->ty = ty->bty;//ȡ�ӱ��ʽ��������
```
ȫ���������£�

```c
static AstExpression CheckUnaryExpression(AstExpression expr)
{
    Type ty;

    switch (expr->op)
    {
    //...
    case OP_DEREF:  // *
        //PRINT_I_AM_HERE();
        expr->kids[0] = Adjust(CheckExpression(expr->kids[0]), 1);
        ty = expr->kids[0]->ty;
        if (expr->kids[0]->op == OP_ADDRESS)
        {
            //*&a ---------------->  a
            //�ӱ��ʽ��&a���ӱ��ʽ����������a
            expr->kids[0]->kids[0]->ty = ty->bty;
            return expr->kids[0]->kids[0];
        }
        else if (expr->kids[0]->op == OP_ADD
                && (ty->bty->categ == ARRAY || expr->kids[0]->kids[0]->isarray))

        {
            //*(arr+3)    --------->  arr[3]
            //�ӱ��ʽ��arr+3����һ�����飬�ӱ��ʽ��������������Ԫ��arr[3]������
            expr->kids[0]->op = OP_INDEX;
            expr->kids[0]->ty = ty->bty;
            expr->kids[0]->lvalue = 1;
            return expr->kids[0];
        }
        if (IsPtrType(ty))
        {
            expr->ty = ty->bty;
            //...
            expr->lvalue = 1;
            return expr;
        }
        break;
    }
}
```
������΢��һ��С���⣬�����´���ļ����

```c
    int *p;
    *(*(p+2)+3) = 1;
```
*(p+2)�õ�����int���ͣ�*(p+2)+3Ҳ��int���ͣ���û���ӱ��ʽ����ʱ*(*(p+2)+3)�Ĵ����л����ty->bty->categ�����ֱ�ӵ����ڴ��������������Ӧ���ȼ��ԭ�����͡�

ȡ��ַ�����&Ҳ�����Ƶģ������������һ��ָ�����ͣ����ָ������ָ���ӱ��ʽ�����͡�