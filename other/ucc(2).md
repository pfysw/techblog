
ucc�������������ܽ�(2) ����ϵͳ�ͷ��Ź���

���﷨�����Ĺ����У�ucc����C���Ե��ķ�������һ���﷨����������Ҫ�����﷨�������������飬�жϴ����Ƿ���ڱ������Ʃ����ʽ�еı�����û�ж��壬��������ĳ����Ƿ�Ϊ���������ʽ�е������Ƿ�Ϸ��ȵȡ�

�������Ϊ�����ʽ����������������֣��ڷ����������֮ǰ����Ҫ���˽�ucc�����͹���������������л�Ϊ�����ı���������һ������ϵͳ��ͬʱ�Գ��ֱ�ʶ�����й���

# 1 ����ϵͳ
## 1.1 ��������

���ȶ���һЩ����������ͣ���SetupTypeSystem()�����г�ʼ��,���ͽṹ������

```c
#define TYPE_COMMON \
    int categ : 8;  \
    int qual  : 8;  \
    int align : 16; \
    int size;       \
    struct type *bty;
	
typedef struct type
{
    TYPE_COMMON
} *Type;
```
���˻�������֮�󣬾Ϳ���ͨ�����͵ĸ�������µ����ͣ�����btyָ����������ͣ�����ͼ��ʾָ���int����ϣ������int�����     
![���������ͼƬ����](../pic/ucc2.1.PNG)   
## 1.2 �ṹ������

�ṹ��������Ҫ�Ի������ͽ�������Ӷ��ܹ���ʾ�ṹ��ĳ�Ա��λ�򣬴���������ͼ���������Ŀ����ṹ������struct recordType�ͳ�Ա������λ���Լ��������͵Ĺ�ϵ     
![���������ͼƬ����](../pic/ucc2.2.png)     
��Ա������λ��ı�﷽ʽ����ͬ�ģ�����struct field���ͣ�ֻ������Ա������posʼ��Ϊ0����Ա֮��������������һ�𣬲��Ҷ�ָ��һ�����������͡�

## 1.3 ��������

�ں��������ͽṹ�У�bty��ԱΪ����ֵ�����ͣ�����������һ��sig��Ա����¼��غ�����Ϣ������sig.hasProto��ʾ�Ƿ�ʹ����ʽ����������񣬱���int f(a,b)int a in b{}��������ʽ���sig.hasEllipsis��ʾ��������������f(...), ���sig.paramsΪһ�������б������洢�����βΣ������ֱ�۽ṹ����ͼ��  
![���������ͼƬ����](../pic/ucc2.3.PNG)    
# 2 ���Ź���

��C�����������˸��ֱ�ʶ������Ҫ����Щ���ŷ��ಢͳһ����ÿ��������һ��struct symbol�ṹ�����ͣ����涨���˷��ŵ������Ϣ��

���������ܹ�12�֣���Щ����ֱ����struct symbol�Ϳ��Լ�¼�����Ϣ������Щ������Ҫ���Ӷ���ĳ�Ա��¼������Ϣ������һ��������˵������Ҫ�������Ϣ����¼�����ӱ��ʽ������

```
c = a+b;
d =a+b;
```
�����a+b�ǹ����ӱ��ʽ��c = a+b����֮��a��b��ֵ��û�иı䣬���Բ������¼��㣬ֱ�Ӳ��Ϳ����ˣ������Ľṹ�嶨������

```c
typedef struct variableSymbol
{
    SYMBOL_COMMON
    InitData idata;
    ValueDef def;
    ValueUse uses;
    int offset;
} *VariableSymbol;
```
��������ͼ���Կ��������͹����ӱ��ʽ�Ĺ�ϵ    

![���������ͼƬ����](../pic/ucc2.4.PNG)     
�����ķ��ŽṹҲ��Ҫ�ڻ����ķ��Žṹ�Ͻ������䣬����params��¼�βη��ŵĵ�������locals��¼�ֲ������ĵ����������⻹��һ��hash���¼���ֹ����ӱ��ʽ�������ٲ�ѯ����ϣ������ͼ��ʾ     
![���������ͼƬ����](../pic/ucc2.5.PNG)       
����������������δ洢��Щ���ţ�����uccר�Ŷ�����struct table���͵�ȫ�ֱ�������ţ���������

```c
typedef struct table
{
    Symbol *buckets;
	int level;
	struct table *outer;
} *Table;

// tags in global scope, tag means struct/union, enumeration name
static struct table GlobalTags;
// normal identifiers in global scope
static struct table GlobalIDs;
// all the constants
static struct table Constants;
// tags in current scope
static Table Tags;
// normal identifiers in current scope
static Table Identifiers;
```
����Tags��ŵ��ǽṹ����ϣ�Identifiers��ŵ����������ţ���C������ÿһ��{}����һ���µ��򣬲�ͬ�����еķ������໥���ɼ��ģ�����ÿ�����ж������Ÿ��ֵķ���GlobalTags��GlobalIDs�������ȫ�ַ��š�ÿ�ν�����뿪һ���򶼻����EnterScope��ExitScope��Ȼ��ı�Identifiers->outer��Tags->outerָ�����buckets��һ��hash��keyֵ��ͬ�ķ��Ż������һ�������������´���

```c
// ��� level is 0
int a = 10;
int main(int argc, char * argv[])
{ // level is 1
    { // level is 2
        int a = 20;
        int b[10] =
        { 1, 2 };
        { // level is 3
            int a = 30;
            a = 40;
        }
        a = 50;
    }
    a = 60;
    return 0;
}
```
����a�ڲ�ͬλ���еĴ洢����    
![���������ͼƬ����](../pic/ucc2.6.PNG)     
