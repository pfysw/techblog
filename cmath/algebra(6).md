抽象代数的代码实现（6）自同构、正规子群、商群   

到目前为止，这里面的复杂性已经超出了我的想象，我觉得最关键的还是表达力太欠缺了，怎么用代码来表示各种数学的表达式如多元函数等，数学上的各种变换和各种代换怎么表示，还有各种表达式间的混合运算，真的感觉这些已经不在我的认知范围内了，就好像一个初中生去做微积分一样困难。

本章内容涉及到了抽象代数的一些关键概念，包括自同构、正规扩张、正规子群、商群和伽罗瓦对应，目前还是通过代码举例说明这些概念为主，其实这些概念之间在逻辑上有很强的制约关系，所以对应存在很多精彩的证明，如上所述，由于现在表达力太欠缺了，暂时没有办法用代码把这些证明表达出来，文字证明见E.Artin的《伽罗华理论》中文版。

# 自同构
对于域K到K'的一一映射σ，如果σ(a)σ(b)=σ(ab),那么就把σ叫做同构，特别的如果K’就是K本身那么把σ叫做K的自同构，接下来我们研究的主要是自同构。同构的意思就是映射后乘法仍然是成立的，自同构的意思相当于把自身元素进行交换，保持乘法成立。显然，K到K的恒等映射是一个自同构。

现在来通过具体的例子说明，比如x ^4 - 2 = 0,这个函数有4个根，分别是     
![在这里插入图片描述](../pic/5.5.PNG)     
现在设K是有理数域，E是包含上面4个根的最小域，可以看到E主要是由2^(1/4)和i构成的,现在找出所有使K保持不变的E的自同构，通过前面的介绍我们已经知道扩域的基本流程，通过添加一个新元使其作为多项式的根，域的结构是由多项式决定，根的选择不影响域的结构，也就是说只要多项式相同，那么通过不同的根扩出来的域其实是同构的，这样通过交换根来映射就得到以下8组自同构。    
![在这里插入图片描述](../pic/5.6.PNG)    
说明一下，1是恒等变换，σ保持i不变，把2^(1/4)变成i*2^(1/4)，τ保持2^(1/4)不变，把i变成-i，其中σ^4=1,τ^2=1,τστ^-1 = σ^-1
# 正规扩张
容易验证上面8个自同构组成一个群，现在先引入命题14，证明见E.Artin的伽罗瓦理论：

**如果σ1，σ2，...,σn做成E的自同构群，并且K是这些同构的不动点体（即映射后保持不变），则E在K上的扩张次数(E/K)=n**

这样就可以定义正规扩张的概念了，设E为K的扩域，G为E的自同构群，G使K中的元素保持不变，那么称E为K的正规扩张，扩张次数就是群G的元素个数。

于是上节中定义的8个自同构组成一个E的自同构群，G使有理数域K保持不变，所以(E/K)=8，该域的向量空间由以下8个元素生成：   
![在这里插入图片描述](../pic/5.7.PNG)     
# 代码实现
在介绍接下来的概念之前，我觉得有必要先把上面的自同构实现以下。因为2^(1/4)是x^4 - 2 = 0的根，首先要在有理数上把2^(1/4)添加进来扩域，这是一个4次扩域K(2^(1/4))/K=4, 由于这个域并没有虚数i，但是根里面出现了i，那么还要在K(2^(1/4))的基础上把i添加进来得到扩域E，其中i是x^2+1=0的根，(E/K(2^(1/4)）)=2,(E/K)=2*4=8

代码中首先在把有理数域扩充成K(2^(1/4))，第一个参数是新域，第2个参数是有理数域，第3个参数是x^4-2的最高次数

```c
SetVecField(&pAlgebra->pVecField,pAlgebra->pField,4);
//设置多项式参数
    aCoef[0] = -4;
    aCoef[1] = 0;
    aCoef[2] = 0;
    aCoef[3] = 0;
    aCoef[4] = 1;
```
然后在K(2^(1/4))的基础上通过多项式x ^2+1再继续扩充

```c
FieldSys *ExtendField(FieldSys *pField)
{
    ... ...
    int aCoef[10] = {1,0,1};

    pExtend = (FieldSys *)malloc(sizeof(FieldSys));
    //新建有理数多项式x^2+1
    pTemp = NewPolyVec(pSub,aCoef,2);
    //改写成新域下的多项式
    pPoly = GetNewFieldPoly(pField,pTemp);
    FreeVector(pTemp);
    PrintVec(pField,pPoly);
    //通过多项式x^2+1来扩域
    pExtend->pGroup1 = PolyPlusObj(pField,pPoly->nEle-1,pPoly);
    pExtend->pGroup2 = PolyMultObj(pField,pPoly->nEle-1,pPoly);
    pField->pParent = pExtend;
    loga("Extend %d",pPoly->nEle);
    IsField(pExtend);

    return pExtend;
}
```
这样就得到了包含x^4-2=0所有根的域E，现在在来实现E中使K不变的8个自同构群，首先要定义自同构群的结构体

```c
typedef struct MapEle MapEle;
struct MapEle
{
    //σ和τ两种基本变换，所有自同构都由这2种基本变换生成
    void (*xSigma)(VectorEle* pSrc);
    void (*xTau)(VectorEle* pSrc);
    u8 nSigma; //σ变换的次数
    u8 nTau;//τ变换的次数
};
```
定义后好，我们就可以实现群中需要的基本接口，生成函数，乘法，求逆，等号判断，这样就得到了一个自同构群

```c
OperateSys *AutomorphismsObj(void)
{
    OperateSys *pGroup;
    MapEle *pBaseItem;

    pGroup = (OperateSys *)malloc(sizeof(OperateSys));
    memset(pGroup,0,sizeof(OperateSys));
    //群中的单位元为恒等变换
    pBaseItem = (MapEle *)malloc(sizeof(MapEle));
    memset(pBaseItem,0,sizeof(MapEle));
    pBaseItem->xSigma = SigmaMap;
    pBaseItem->xTau = TauMap;
    pBaseItem->nSigma = 0;
    pBaseItem->nTau = 0;
    
    pGroup->nPara = 1;
    pGroup->pBaseEle = pBaseItem;
    pGroup->xGen = (void*)AutomorphismsGen;
    pGroup->xOperat = (void*)AutomorphismsMult;
    pGroup->xInvEle = (void*)AutomorphismsInv;
    pGroup->xIsEqual = (void*)AutomorphismsEqual;

    return pGroup;
}
```
把这个自同构群放进群证明器中可以验证其为群

```c
OperateSys *pMap = AutomorphismsObj();
IsGroup(pMap);
```
现在我们来定义这个群对E中元素所做的变换，比如群中元素是σ^2τ,那么pMap->nSigma就是2，pMap->nTau是1

```c
void FieldMapping(MapEle* pMap, VectorEle* pVec)
{
    int i;
    for(i=0;i<pMap->nSigma;i++)
    {
        pMap->xSigma(pVec);
    }
    for(i=0;i<pMap->nTau;i++)
    {
        pMap->xTau(pVec);
    }
}
```
那么怎么能说明上面定义的对自身的映射是自同构呢，可以通过以下代码说明：

```c
int isAutomorphisms(FieldSys *pField,OperateSys *pMap)
{
    int rc = 0;
    int i,j,k;
    VectorEle* pT[3];
    VectorEle* pV[4];
    OperateSys *pOpSys = pField->pGroup2;
    MapEle *pEle;

    for(i=0; i<8; i++)
    {
        loga("i %d",i);
        k = FakeRand(i);
        SetGenPara(pOpSys,k);
        pEle = pMap->xGen(pMap,i);
        //生成元素a，b和ab
        pT[0] = pOpSys->xGen(pOpSys,k);
        pT[1] = pOpSys->xGen(pOpSys,k+2);
        pT[2] = pOpSys->xOperat(pT[0],pT[1]);

        for(j=0;j<3;j++)
        {
            pV[j] = VecCpy(pT[j]);
        }
        //生成变换后的元素σ(a),σ(b),σ(ab),
        for(j=0;j<3;j++)
        {
            FieldMapping(pEle,pV[j]);
        }
        //计算σ(a)*σ(b)
        pV[3] = pOpSys->xOperat(pV[0],pV[1]);
        //验证σ(a)*σ(b)=σ(ab)
        rc = pOpSys->xIsEqual(pV[2],pV[3]);
		assert(rc);
    }

    return rc;
}
```
# 例子
现在把新扩张的域E写成如下形式，是由8个元素生成的向量空间    
![在这里插入图片描述](../pic/5.8.PNG)    
再写成向量的形式，第一行没有2^(1/4)其他行有，第一列没i，第2列有i   
![在这里插入图片描述](../pic/5.9.PNG)    
现在以一个元素为例，来说明这8个自同构        
79.00  &nbsp;  -80.00  
-50.00  &nbsp; 3.00    
73.00  &nbsp;  -74.00  
-12.00 &nbsp;  49.00     
![在这里插入图片描述](../pic/5.1.PNG)      
向量的第一行是a+bi组成的中间域B，我们看到1，σ，σ ^2, σ ^3并不改变该域的元素，所以这是使域B保持不变的自同构，并且组成一个群，所以E是B上的正规扩张，1，τ是B上的自同构群，并且使有理数域K保持不变，所以B是K上的正规扩张。现在以这8个自同构组成的群为基础，定义一些新的概念，并且不加证明的给出一些关键结论。

# 伽罗瓦基本定理
如上所述，定义有理数域为K，a+bi为中间域B，x^4 - 2 = 0的根组成的域为E，其中扩张关系如下图所示       
![在这里插入图片描述](../pic/5.2.PNG)     
设E的8个自同构为G，其子集{1，σ，σ ^2, σ ^3}为U，U是一个自同构群，在G中的元素有些使B发生改变而又些使B保持不变，而U中的元素都使B保持不变。

现在知道E是K的正规扩张，E也是B的正规扩张，那么B是K的正规扩张吗？B是K中通过添加x^2 + 1 = 0的根后扩张的，(B/K)=2,现在引入命题13：

**如果σ1，σ2，...,σn为两两不同的E到E’的同构，并且K是这些同构的不动点体，则E在K上的扩张次数(E/K)>=n**

现在把上述命题改为B到E的同构映射，且(B/K)=2，那么也就是说B到E最多有2个同构，把G的这8个同构作用在B上最多只能产生2种不同的映射，假设B中的元素为β，那么G中的元素x作用在β上时，x(β)可能还在B里，也可能跳出B到E里，如果B是正规扩张，那么这2种映射必须要是B的自同构，即x(β)还在B里面。因为x(β)还在B里面，U是一个使B保持不变的自同构群，所以U中的元素作用于x(β) 不变  即Ux(β)=x(β)，再两边乘以x^-1就得到了x ^-1Ux(β) = β,也就是说x ^-1Ux使β不变，所以x^-1Ux = U，通过上面的实际例子，把x取遍G中的所有元素，再结合τστ^-1 = σ^-1可以很容易验证这个式子。

在引出新概念之前，先直接给出命题16：

**设E是K上的具有群G的正规扩张，B为中间体，则E为B的正规扩张，也就是说存在自同构群使B保持不变。**

因为E已经是K的正规扩张了，上面的命题断言了存在一个群U使B保持不变，对应上面的例子就是U = {1，σ，σ ^2, σ ^3}。现在来定义一个新概念，如果G中的所有元素作用在B上不会出现B中元素跳出域B的情况，即对于G中的元素x，满足x^-1Ux = U，那么我们把U称作**正规子群**，要注意如果跳出了B就不是正规子群了，按上面的例子，B是由a+bi形式的元素组成的域，由于域中没有出现2^(1/4)，所以任何G中的同构作用在B上的元素都不会跳出B，所以U是正规子群。此时也就是说B到E的同构也就是B到B的自同构，并且这些自同构不会使K产生改变，所以B就是K的正规扩张。到这来我们发现G中的正规子群U与E中在U作用下不变的域B是一一对应的，并且B是K的正规扩张，这就是所谓的**伽罗瓦基本定理**。既然B是K的正规扩张，再来寻找B使K保持不变的自同构，由于U作用在B上保持不变，那么旁系xU的每一个元素作用在B上与x作用在B上的同构是相同的，所以xU对应于G中元作用在B上的不同同构，考虑两个旁系xU和yU的积，由于U是正规子群，所以y作用在B上并不会跳出B外面，所以xU*yU =  xyU,于是旁系xU也组成了一个自同构群，把这样的群称作**商群G/U**,即U是正规子群，那么B就是正规扩张，B的自同构群就是商群G/U。

为了更加清晰，再举一个U不是正规子群的例子，如下图，B是中间域，K添加2^(1/4)生成的4次扩域：  
![在这里插入图片描述](../pic/5.3.PNG)      
按命题16，E是B的正规扩张，U = {1,τ}使B保持不变，U是G的子群，但不是正规子群。例如B中的元素2^(1/4)在σ的作用下生成了i*2^(1/4)，该元素并不在B内，所以B也不是K的正规扩张，也就是说不存在4个B中的自同构使K保持不变，验证G的8个自同构，只有旁系1*U和σ ^2 U是B的自同构，这2个自同构组成的群使域K((2^(1/4))^2)保持不变，这个域记为B'，所以B是B'的正规扩张，通过添加x^2 - √2的根扩张而成，但不是K的正规扩张，现在把{1*U和σ ^2 U}记为U',B'是K添加(2^(1/4))^2的2次扩域，其中U'使B'保持不变，而且U '是商群G/U的正规子群，G/U的任何元素作用在B'上都不能使B'中的元素跳出B'，这样B'就是K的正规扩张，商群(G/U)/U'是B '的自同构，这里商群(G/U)有4个元素，而商群(G/U)/U'只有2个元素，一个是恒等变换，一个把(2^(1/4))^2变成-(2^(1/4))^2，这个元素是σU*U '。最后在这种场景下，正确的扩张顺序是K先添加x^2 - 2=0的根正规扩张成B'，再添加x^2 - √2=0根正规扩张成B，但是B不是K的正规扩张，最后添加x ^2+1=0的根正规扩张成E，扩张顺序如下图所示     
![在这里插入图片描述](../pic/5.4.PNG)      
# 参考代码
https://github.com/pfysw/CMath