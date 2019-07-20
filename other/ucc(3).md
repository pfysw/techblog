
ucc�������������ܽ�(3) �������

��������ϵͳ�ͷ��Ź���Ļ���֪ʶ֮�󣬽������Ϳ��Է���������鲿�ֵĴ����ˡ�

Դ���뾭��Ԥ���������.i�ļ�����ʱ�������Ҫ��Ϊ��������������������֣���ʵ����ֻ������һ�������������䣬������������{}��������ݡ��������ͷǺ�������Ƿֿ����ģ�

```c
if (p->kind == NK_Function)
{
    CheckFunction((AstFunction)p);
}
else
{
    assert(p->kind == NK_Declaration);
    CheckGlobalDeclaration((AstDeclaration)p);
}
```

������������ȫ�ֶ����������飬Ҳ����CheckGlobalDeclaration()����������ݡ�

# 1 �������ͼ��
һ��ʼ����CheckDeclarationSpecifiers(decl->specs)�����������͵ļ�飬�������ͼ���⻹����һЩ���η���const��static�ȵȡ����֮��õ������ͽ��ḳֵ��specs->ty,��������δʻ���Ҫ����ϳ�һ���µ����ͣ�������������

```c
static void CheckDeclarationSpecifiers(AstSpecifiers specs)
{
    ... ...
    //storage-class-specifier:      extern  , auto, static, register, ... 
    tok = (AstToken) specs->stgClasses;
    if (tok)
    {
        ...
        specs->sclass = tok->token;
    }
    //type-qualifier:   const, volatile
    tok = (AstToken) specs->tyQuals;
    while (tok)
    {
        qual |= (tok->token == TK_CONST ? CONST : VOLATILE);
        tok = (AstToken) tok->next;
    }
    //type-specifier:   int,double, struct ..., union ..., ...
    p = specs->tySpecs;
    while (p)
    {
        //�ṹ������
        if (p->kind == NK_StructSpecifier || p->kind == NK_UnionSpecifier)
        {
            ty = CheckStructOrUnionSpecifier((AstStructSpecifier) p);
            tyCnt++;
        }
        else if (p->kind == NK_EnumSpecifier)//ö������
        {
            ty = CheckEnumSpecifier((AstEnumSpecifier) p);
            tyCnt++;
        }
        else if (p->kind == NK_TypedefName)//typedef�ض�������
        {
            ...
        }
        else
        {
            //��������
            tok = (AstToken) p;

            switch (tok->token)
            {
            case TK_SIGNED:
            case TK_UNSIGNED:
                sign = tok->token;
                signCnt++;
                break;

            case TK_SHORT:
            case TK_LONG:
                if (size == TK_LONG && sizeCnt == 1)
                {
                    size = TK_LONG + TK_LONG;
                }
                else
                {
                    size = tok->token;
                    sizeCnt++;
                }
                break;

            case TK_CHAR:
                ty = T(CHAR);
                tyCnt++;
                break;

            case TK_INT:
                ty = T(INT);
                tyCnt++;
                break;

            ...
            }
        }
        p = p->next;
    }
    ...

    //������η����Ͳ�����
    specs->ty = Qualify(qual, ty);
    return;
}
```
# 2 �ṹ�����ͼ��

�����ṹ������ʱ������CheckStructOrUnionSpecifier()�����м������봦���ṹ����4�����ͣ��ֱ���

 1.  struct Data1 // �нṹ�������ޡ������š�
 2. struct {int a; int b;} // �޽ṹ�������С������š�
3. struct Data2{int a; int b;} // �нṹ����Ҳ�С������š�
4. struct // �޽ṹ����Ҳ�ޡ������š� ���﷨����ʱ�ѱ���

����ṹ�������֣���ô���ȼ����ű����Ƿ��Ѿ������˸����֣����û�У���ô�½�һ���ṹ�����Ͳ����浽���ű��У���������

```c
tag = LookupTag(stSpec->id);
if (tag == NULL)
{
    ty = StartRecord(stSpec->id, categ);
    tag = AddTag(stSpec->id, ty,&stSpec->coord);
}
else if (tag->ty->categ != categ)
{
    Error(&stSpec->coord, "Inconsistent tag declaration.");
}
```
��������Խṹ��ĳ�Ա������һ�����������

```c
	while (stDecl)
	{
        CheckStructDeclaration(stDecl, ty);
        stDecl = (AstStructDeclaration)stDecl->next;
	}
```
����ͨ�������ļ�����ƣ��ȼ���������ͣ��ټ��������ʶ��

```c
CheckDeclarationSpecifiers(stDecl->specs);
...
/**
 struct Data{
 int c, d;           
 }
 */
while (stDec)
{
    //�ṹ�����ͣ���Ա��������Ա����
    CheckStructDeclarator(rty, stDec, stDecl->specs->ty);
    stDec = (AstStructDeclarator)stDec->next;
}
```
��Ա��ʶ���ļ���ȫ�ֱ�ʶ���ļ�����ƣ����ǵ���CheckDeclarator()��DeriveType()����ɸ������͵Ĺ�����������ں��������ÿ����Ա���͹�����Ϻ�����AddField()�ѳ�Ա���뵽�ṹ�����͵�λ�����������һƪ���µ�1.2�ڣ���ش������£�

```c
static void CheckStructDeclarator(Type rty, AstStructDeclarator stDec, Type fty)
{
    char *id = NULL;
    int bits = 0;

    // ����գ����ܳ���λ����û�����ֵ����������int :4;
    if (stDec->dec != NULL)
    {
        CheckDeclarator(stDec->dec);
        id = stDec->dec->id;
        fty = DeriveType(stDec->dec->tyDrvList, fty, &stDec->coord);
    }

    //...

    AddField(rty, id, fty, bits);
}

```
# 3 ������ʶ���ļ��

����ɵݹ����CheckDeclarator()��������ɣ��ڹ����﷨��ʱ���ܻὫ������ʶ��������ָ������͸�����һ�������������NK_NameDeclarator�����ֹ�ݹ飬��������

```c
static void CheckDeclarator(AstDeclarator dec)
{
    switch (dec->kind)
    {
    case NK_NameDeclarator:
        break;

    case NK_ArrayDeclarator:
        CheckArrayDeclarator((AstArrayDeclarator) dec);
        break;

    case NK_FunctionDeclarator:
        CheckFunctionDeclarator((AstFunctionDeclarator) dec);
        break;

    case NK_PointerDeclarator:
        CheckPointerDeclarator((AstPointerDeclarator) dec);
        break;

    default:
        assert(0);
    }
}
```
����������Ϊ�����ȵݹ����CheckDeclarator()����ӽ�㣬�����Ϻ��ټ�����鳤�ȣ����Ѹý����뵽tyDrvList�����ͷ������������

```c
//  int arr[4];
static void CheckArrayDeclarator(AstArrayDeclarator arrDec)
{
    CheckDeclarator(arrDec->dec);
    /**
     struct Data{
     ....
     int a[];    ----> legal.    when  arrDec->expr is NULL.
     }
     */
    if (arrDec->expr)
    {
        if ((arrDec->expr = CheckConstantExpression(arrDec->expr)) == NULL)
        {
            Error(&arrDec->coord,
                    "The size of the array must be integer constant.");
        }
    }

    ALLOC(arrDec->tyDrvList);
    arrDec->tyDrvList->ctor = ARRAY_OF;
    arrDec->tyDrvList->len = arrDec->expr ? arrDec->expr->val.i[0] : 0;
    arrDec->tyDrvList->next = arrDec->dec->tyDrvList;
    arrDec->id = arrDec->dec->id;
}
```
ָ����ͺ������Ҳ�����Ƶģ���ͼ˵����int *arr1[5]��int (*arr2)[5]�Ľ�������     
![���������ͼƬ����](../pic/ucc3.1.PNG)    
�ڹ�����tyDrvList����֮��DeriveType()���������tyDrvList����������������ϳ�һ���������ͣ�����ָ��������������ͼ��ʾ       
![���������ͼƬ����](../pic/ucc3.2.PNG)     
�ڹ��������ͺ󣬻�����ű�����û�иñ�ʶ�������û����ѱ�ʶ�����ƺ�����һ����ӵ����ű���

```c
/**
 Check for global variables
 */
if ((sym = LookupID(initDec->dec->id)) == NULL)
{
    sym = AddVariable(initDec->dec->id, ty, sclass,&initDec->coord);
}
```

�����������������飬��CheckFunction()ʵ�֣����Ƶ�ҲҪ��麯���������ͣ��������ͣ�����ʶ��������������ͨ��DeriveType()��ϳɸ���������ӵ��������ű��֮ǰ��CheckFunctionDeclarator()�������Ѿ��Ѻ��������ŵ���((FunctionType)ty)->sig->params�����Ȼ��ͨ��AddVariable()�Ѻ����β��б���ӵ����ű����������Ϻ������Ҫ��麯������Ƿ�Ϸ���ͨ������CheckCompoundStatement(func->stmt)����ɣ����������һƪ��������ϸ������
