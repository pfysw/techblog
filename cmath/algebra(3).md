# �����ռ�
## �����
��ƪ���½�����������Լ򵥵����Ϊһ�����������ϣ�����������мӷ�����ͳ˷����㣬�����Ľ���Ƿ�յģ����������������������Ǵ����������ȡ��n��Ԫ�أ���n��Ԫ�����һ����˳����Ӽ�������Ӽ���Ϊnά������

��a��b��c������������ô(a b c)��һ����ά����������������ɵļ��ϳ�ΪV����������������ΪK��V�е�Ԫ��ΪA��B��C�����������������������ô��V��ΪK�ϵ������ռ�

 - a(A+B) = aA + aB
 - (a + b)A = aA + bA
 - a(bA) = (ab)A
 - 1A = A
 
 # ����ʵ��
 
����ƪ�������Ѿ�������������������ݽṹ�����÷�������ʽ�����Ǿ�����μӼ��˳���������������Ϊ�˱�������鷳�����ǲ��ø���������ʾ

```c
struct FieldEle
{
    u32 nmrtr;//����
    u32 dnmtr;//��ĸ
    float val;
    u8 eSymb;
};
```
�����Ľṹ����Կ�������FieldEleԪ����ɵ�����

```c
struct VectorEle
{
    int nEle;  //����Ԫ�ظ���
    FieldEle **aVecEle;//����������������
};
```
����������VectorEle�����ļӷ���ֻ���aVecEleÿ����Ӧ��Ԫ����ӣ����������е�Ԫ�����������ʱ��ֻҪ�Ѹ�Ԫ�طֱ���aVecEle��ÿ��Ԫ����ˣ�����֤�����������VectorEle��һ�������ռ�

����֤��a(A+B) = aA + aB����������

```c
int VectorDist1(FieldSys *pField,int nEle)
{
    int rc = 0;
    int i,j,l;
    u32 k;
    FieldEle* pFieldEle;
    OperateSys *pMult = pField->pGroup2;
    VectorEle *pVector[2];
    VectorEle *pV[5];

    //�������10������
    for(i=0; i<10; i++)
    {
        //�������һ��������
        k = FakeRand(i+j*10);
        SetGenPara(pMult,k);
        pFieldEle = pMult->xGen(pMult,k);
        //�������2������
        for(j=0; j<2; j++)
        {
            pVector[j] = (VectorEle *)malloc(sizeof(VectorEle));
            memset(pVector[j],0,sizeof(VectorEle));
            pVector[j]->nEle = nEle;
            pVector[j]->aVecEle = malloc(nEle*sizeof(void *));
            for(l=0;l<nEle; l++)
            {
                k = FakeRand(i+j+l);
                SetGenPara(pMult,k);
                pVector[j]->aVecEle[l] = pMult->xGen(pMult,k);
            }
        }
        //����A+B
        pV[0] = VectorPlus(pField,pVector[0],pVector[1]);
         //����a(A+B)
        pV[1] = FieldMultVector(pField,pFieldEle,pV[0]);
        //����aA
        pV[2] = FieldMultVector(pField,pFieldEle,pVector[0]);
        //����aB
        pV[3] = FieldMultVector(pField,pFieldEle,pVector[1]);
        //����aA+aB
        pV[4] = VectorPlus(pField,pV[2],pV[3]);
        //��֤�Ƿ����
        rc = isVecotrEqual(pField,pV[1],pV[4]);
        FreeVector(pVector[0]);
        FreeVector(pVector[1]);
        for(j=0; j<5; j++)
        {
            FreeVector(pV[j]);
        }
        assert( rc );
    }
    loga("vector Distributive1 ok %d",rc);
    return rc;
}
```

�����������ʵ�֤�����ƣ��ӿ�����

```c
void isVectorSpace(FieldSys *pField,int nEle)
{
    //a(A+B) = aA + aB
    VectorDist1(pField,nEle);
    //֤��(a + b)A = aA + bA
    VectorDist2(pField,nEle);
    //a(bA)=(ab)A
    VectorAsso(pField,nEle);
    //1A=A
    VectorIdentity(pField,nEle);
}

```
# �������
���������ռ�ĸ������������һ���ǳ���Ҫ�ĸ���Ҳ����������ء������������A��B��C���������           
![���������ͼƬ����](../pic/3.1.PNG)        
 �Ľ�x1,x2,x3,��ȫΪ0����ô��������˵����A��B��C������أ�������ص����������3��������ÿһ������������2��������ʾ�ˣ����Ƶ�������صĸ��������ƹ㵽���ࡣ    

�����������жϸ���һ�������飬�ж��Ƿ���������أ��������һ����ά�����ռ䣬��ô�ٶ�
![���������ͼƬ����](../pic/3.2.PNG)      
��ô(*)ʽ�Ľ�Ҳ���ǣ����з�����Ľ�

![���������ͼƬ����](../pic/3.3.PNG)       
���ʱ���õ�����Ԫ�����Ȱѷ��̣�2���ͷ��̣�3����ȥ���̣�1���������ϵ��������x1���ٵݹ���ⷽ��(2)�ͷ��̣�3����ɵ�2Ԫ���̣���������

```c
int isLinearDepedent(
        FieldSys *pField,
        VectorEle **paVector,//�����������
        int n,//n������
        int iRow,//����ķ�����ʼ���
        int nRow)//����ķ��̸���
{
    int rc = 0;
    int i,j;
    OperateSys *pPlus = pField->pGroup1;
    FieldEle **paEle;
    FieldEle *pTemp;
    FieldEle *pTempEle;

    assert(n>0);
    assert(nRow>0);
    if( n==1 )
    {
        //������Ԫ�ݹ��ֻʣһ�����������ܻ��ж������
        paEle= (FieldEle **)&((paVector[0])->aVecEle[iRow]);
        for(i=0; i<nRow; i++)
        {
            //���Ա任�����һ���з���Ԫ�أ����Բ����ܴ��ڷ���⣬�����޹�
            if( !pPlus->xIsEqual(paEle[i],pPlus->pBaseEle) )
            {
                return 0;
            }
        }
        //������һ��ϵ��ȫ��0����ô�з���⣬�������
        return 1;
    }
    else if( nRow==1 )
    {
        //ֻʣһ�У�����δ֪��������1����ô�ض��з����
        return 1;
    }
    else
    {
        paEle = (FieldEle **)&((paVector[0])->aVecEle[iRow]);
        for(i=0; i<nRow; i++)
        {
            //�ҵ���1�еķ�0Ԫ�أ�������һ��
            if( !pPlus->xIsEqual(paEle[i],pPlus->pBaseEle) )
            {

                for(j=0;j<n;j++)
                {
                    paEle = (FieldEle **)&((paVector[j])->aVecEle[iRow]);
                    pTempEle = paEle[i];
                    paEle[i] = paEle[0];
                    paEle[0] = pTempEle;
                }
                break;
            }
        }
        paEle = (FieldEle **)&((paVector[0])->aVecEle[iRow]);
        //�����һ�е�Ԫ��ȫΪ0����ô������һ�У��ݹ���һ��
        //todo ��ʱ�Ƿ����ֱ���϶�Ϊ�������
        if( pPlus->xIsEqual(paEle[0],pPlus->pBaseEle) )
        {
            rc = isLinearDepedent(pField,&paVector[1],n-1,iRow,nRow);
        }
        else
        {
            for(i=1; i<nRow; i++)
            {
                paEle = (FieldEle **)&((paVector[0])->aVecEle[iRow]);
                pTemp = FiledDiv(pField,paEle[i],paEle[0]);
                //pTemp�Ǻ��淽�̵�Ԫ�����1������Ԫ�صı�ֵ
                for(j=0;j<n;j++)
                {
                    paEle = (FieldEle **)&((paVector[j])->aVecEle[iRow]);
                    //���������ֵУ԰
                    paEle[i] = EliminationUnkowns(pField,paEle[i],paEle[0],pTemp);
                }
                free(pTemp);
            }
            //��ȥ��һ�������󣬵ݹ���һ�к���һ�У�����ִ�����ϲ���
            rc = isLinearDepedent(pField,&paVector[1],n-1,iRow+1,nRow-1);
        }
    }

    return rc;
}
```
# �ο�����
https://github.com/pfysw/CMath