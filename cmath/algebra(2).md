��������Ĵ���ʵ�֣�2����

# ��
��һƪ���½���Ⱥ�ĸ������������һ����ĸ��Ⱥ��һ�����Ϻ����㣬������Ԫ�ص���������Ⱥ��4�����ʡ����Ⱥ��˵����������һ�����㣬�����м���A������{+}������{��}���������A��{+}��������һ��Ⱥ�������ӷ�Ⱥ���ӷ�ȺA�еĵ�λԪ��Ϊ��Ԫ��A��ȥ����Ԫ��ļ��϶�{��}����Ҳ����һ��Ⱥ,����+�͡���������ɣ���

 a��(b+c) = a��b+a��c
 
��ô���ǰ�A����һ����Ҫע��+�͡���һ����������֪�ļӷ��ͳ˷���Ҳ���Զ�����������㣬ֻҪ����������ʾ����ˣ�ͨ������ϰ�ߣ�{��}�����ڱ��ʱ����ʡ�ԡ�������������������ĳ������ӡ�

## ��������
�����������ǱȽ���Ϥ��һ�����ӣ������������ļ��϶Լӷ��ͳ˷�����һ���������ļ��ϲ�����һ��������3��������ڳ˷���˵��������Χ�ڲ���������Ԫ����������������Χ�ھʹ�����Ԫ1/3������������һ���򣬿��Ծ�һЩ�������ļӷ��ͳ˷�����

�ӷ����һ��Ⱥ��

   - �����     
   ���� (5/4+��2/1+4/3��=(5/4+2/1)+4/3 = 55/12
   - ��λԪ����Ԫ��     
     ����0+3/2 = 3/2       
   - ����Ԫ     
      ����3/2����Ԫ��-3/2
      
��0��˷����һ��Ⱥ��

- �����          
 (5/4����2/1��4/3��=(5/4��2/1)?4/3 = 40/12  = 10/3
 - ��λԪ��1��       
     ����1��3/2 = 3/2    
 - ����Ԫ        
      ����3/2����Ԫ��2/3,��Ϊ(3/2)��(2/3)=1
## ʣ����
��p����������pΪģ��ʣ�������һ���򣬺���Ĵ����˵��Ϊʲôp������������p=5����ʣ����{0,1,2,3,4}��һ����

�ӷ����һ��Ⱥ��

   - �����       
   ���� 0+(1+2)=(0+1)+2=0
   - ��λԪ��0��    
     ����0+2 = 2          
   - ����Ԫ      
      ����1����Ԫ��4
      
��0��˷����һ��Ⱥ��

- �����      
1��(3��4)=(1��3����4 = 12(mod 5) = 2
 - ��λԪ��1��     
     ����1��4 = 4
  - ����Ԫ      
      ����2����Ԫ��3,��Ϊ2��3=6��mod 5�� = 1
    
 #  ����ʵ��
## ��������

�ȶ�����Ľṹ�壺
```c
struct FieldSys
{
    OperateSys *pGroup1;//�ӷ�Ⱥ
    OperateSys *pGroup2;//�˷�Ⱥ
};
```
������Ԫ�Ľṹ�����£��ɷ��ӡ���ĸ������������ɣ�

```c
typedef struct FieldEle FieldEle;
struct FieldEle
{
    u32 nmrtr;//����
    u32 dnmtr;//��ĸ
    u8 eSymb;//����
};

```
��������Ҫʵ�ּӷ�Ⱥ�ͳ˷�Ⱥ����غ�����Ϊ�˼򵥵㣬�����Ȳ����������Щϸ�ڣ������ɵ���������100���ڡ������ʵ�ֶ��ǳ��򵥣�����ֻ�г��ӿڣ����������ȥ��

�ӷ�Ⱥ��

```c
//����������Ȼ����Ҳ����RationGen����������
FieldEle *NatureGen(OperateSys *pOpSys,u32 num) 
//�������ӷ�
FieldEle *RationPlusOp(FieldEle *p1,FieldEle *p2)
//�ж��������Ƿ����
int RationEqual(FieldEle *p1, FieldEle *p2)
//�������ӷ�ȡ�棬��ȡ����
FieldEle *RationPlusInv(FieldEle *p1)
```


�˷�Ⱥ��
```c
//�����������ɣ�������Ҫ���ӡ���ĸ������3������������ֻ��һ����
//����2������pOpSys�������������100���ڣ�Ҫע�ⲻ������0
FieldEle *RationGen(OperateSys *pOpSys,u32 num)
//�������˷�
FieldEle *RationMultOp(FieldEle *p1,FieldEle *p2)
//����������
FieldEle *RationMultInv(FieldEle *p1)
//�ж��������Ƿ����
int RationEqual(FieldEle *p1, FieldEle *p2)

```
## ʣ������
�ȶ���ʣ����ṹ��

```c
typedef struct ModEle ModEle;
struct ModEle
{
    u32 num;//��Ȼ��
    u32 mod;//ȡģ
};
```

����������ʵ�ּӷ�Ⱥ�ͳ˷�Ⱥ����ؽӿڣ�������ģ5��ʣ����Ϊ��

�ӷ�Ⱥ��

```c
OperateSys *ModPlusObj(void)
{
    static ModEle baseItem =
    {
        0,
        5,
    };

    static OperateSys plus;
    memset(&plus,0,sizeof(plus));

    plus.pBaseEle = &baseItem;
    plus.nPara = 1;
    plus.xIsEqual = (void*)ModEqual;
    plus.xGen = (void*)ModNumGen;
    plus.xInvEle = (void*)ModPlusInv;
    plus.xOperat =  (void*)ModPlus;

    return &plus;
}
```

�˷�Ⱥ��

```c
OperateSys *ModMultObj(void)
{
    static ModEle baseItem =
    {
        1,
        5,
    };

    static OperateSys mult;
    memset(&mult,0,sizeof(mult));

    mult.pBaseEle = &baseItem;
    mult.nPara = 1;
    mult.isMult = 1;
    mult.xIsEqual = (void*)ModEqual;
    mult.xGen = (void*)ModNumGen;
    mult.xInvEle = (void*)ModMultInv;
    mult.xOperat =  (void*)ModMult;

    return &mult;
}
```
�������ܼ򵥣�������Ҫ˵һ�³˷�����Ԫ���㷨�����ݷ��������a��p�����Լ����1,��ôa^(p-1)��1��mod p��,��a*a ^(p-2)��1��mod p��,���a����Ԫ����a ^(p-2),���Դ�������

```c
ModEle *ModMultInv(ModEle *p1)
{
    ModEle *p;
    int i;
    u32 mod = p1->mod;
    p = malloc(sizeof(ModEle));
    memset(p,0,sizeof(ModEle));

    p->mod = mod;
    p->num = 1;
    for(i=0; i<mod-2; i++)
    {
        p->num = (p->num*p1->num)%mod;
    }

    return p;
}
```
## ���֤��
ֻ��Ҫ֤���ӷ���Ⱥ���˷���Ⱥ���ӷ��ͳ˷�֮�����������

```c
void IsGroup(OperateSys *pOpSys)
{
    AssociativeLaw(pOpSys);
    HasIdentityEle(pOpSys);
    HasInvEle(pOpSys);
}

void IsField(FieldSys *pField)
{
    IsGroup(pField->pGroup1);//�ӷ���Ⱥ
    IsGroup(pField->pGroup2);//�˷���Ⱥ
    DistributiveLaw(pField);//������
}
```
������Щ֤��֮��Ϳ��Խ��һЩ��ֵ����⣬����

Ϊʲô�����ĳ˷�����ô���    
![���������ͼƬ����](../pic/2.1.PNG)   
������    
![���������ͼƬ����](../pic/2.2.PNG)          
���ǿ��԰ѷ����ĳ˷�ʵ�ָĳɵ�2����ʽ����ô��֤���˷�ȺʱHasIdentityEle(pOpSys);�����ᴥ�����ԣ���λԪ���κ�Ԫ����˶�����

```c
        pGen = pOpSys->xGen(pOpSys,k);//����һ��Ԫ��
        pEle = pOpSys->xOperat(pOpSys->pBaseEle,pGen);//��Ԫ�غ͵�λԪ��˺�Ľ��
        rc = pOpSys->xIsEqual(pGen,pEle);//�Ƿ����
        assert( rc );
```
�ֱ�������ļӷ�����֣�Ϊʲô����ô���    
![���������ͼƬ����](../pic/2.3.PNG)      
�������������ָ�����ֱ�����㷨��      
![���������ͼƬ����](../pic/2.4.PNG)      
��Ϊ������յ�2���㷨����ô�ͻᴥ����������ɵĶ���

```c
        //�������3��Ԫ��
        for(j=0; j<3; j++)
        {
            k = FakeRand(i+j*10);
            SetGenPara(pMult,k);
            pT[j] = pMult->xGen(pMult,k);
        }
        //��2��Ԫ������ٺ͵�1��Ԫ�����
        pT[3] = pLus->xOperat(pT[1],pT[2]);
        pT[4] = pMult->xOperat(pT[0],pT[3]);
        //��һ��Ԫ�طֱ��ں�2��Ԫ����������
        pT[5] = pMult->xOperat(pT[0],pT[1]);
        pT[6] = pMult->xOperat(pT[0],pT[2]);
        pT[7] = pLus->xOperat(pT[5],pT[6]);
        //�������
        rc = pMult->xIsEqual(pT[4],pT[7]);
        assert( rc );
```
���Ҫ��ģp��ʣ������һ����Ϊʲôֻ��p�������ų�����Ϊ��˵��������⣬���ǿ����ڳ˷���ʼ���а�ģ��Ϊ4

```c
    static ModEle baseItem =
    {
        1,
        4,
    };
```
���������д���� �ͻ��ں���HasInvEle�д����˷����ڿ���Ԫ�Ķ��ԣ���Ϊ����2������Ԫ��xʹ��2*x��1��mod 4��

# �ο�����
����ʵ�ֵĴ���ϸ�ڼ�    
https://github.com/pfysw/CMath/tree/master/Algebra
