# zzl-task1
#here is a completed code to achieve building huffman tree.

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>

//查找结点i的父结点,通过递归得到结点到根的长度
int findParent(int i,int huffman[][4],int n);
//根据初始权重构建哈夫曼树
void huffmanTree(int w[],int huffman[][4],int n);
//寻找权重最小的两个结点
void findMin(int ii[],int huffman[][4],int n);
//对每个叶结点进行哈夫曼编码
void HuffmanCode(int i,int huffman[][4],int n);
//交换两个char型数据
void inplace_swap(char *x,char *y);

int main()
{
    //对输入的字符串进行记录，对字符的总个数count以及各个字符的出现次数arr_small[]进行统计
    char str[100];
    printf("Please Enter the string\n");
    scanf("%s", str);
    int arr_big[27] = {0};  //A~Z各个字符的出现次数
    int count = 0;  //字符的总个数
    for(int i = 0; i < strlen(str); i++)    //遍历输入字符串中的所有字符
    {
        for(int j = 0; j < 27; j++) //遍历26个大写字母,从A开始
        {
            if(str[i] == (char)(0x41 + j)) //从A开始比对
            {
                arr_big[j] += 1;
                if(arr_big[j] == 1)
                {
                    count += 1;		//记录一共出现了几个字母
                }
                break;
            }
        }
    }
    printf("the letter count is %d\n",count);   //打印输入字符串出现了几个字母

    //通过arr_big对输入字符串中出现的字母次数进行统计，放入数组arr_small中
    int count2 = 0;     //字符串中出现的字符总个数
    int arr_small[count] ;  //统计输入字符串中字母的出现次数
    for(int i = 0; i < 27; i++)
    {
        if(arr_big[i]!=0)
        {
            arr_small[count2++] = arr_big[i];	//将不同字母的个数从A开始放入数组arr_small中
        }
    }

    //定义int型二维数组，数组长度[*]为哈夫曼树的结点个数
    //c[*][0]存放的是该结点的[父结点的位序]，c[*][1]为该结点的[左子树结点的位序]
    //c[*][2]存放该结点的[右子树结点的位序]，c[*][3]为该结点的权值
    int huffman[2 * count - 1][4];  //前n个为叶结点，后n-1个为中间结点

    //根据初始权重数组arr_small和字符个数构建huffman树
    huffmanTree(arr_small,huffman,count);

    //计算Huffman生成树的总长度WPL
    int sum = 0;
    for(int i = 0;i < count;i++)
    {
        int length = 0;
        if(huffman[i][1] == -1 && huffman[i][2] == -1)  //判断是否是叶子节点，是则计算路径长度
        {
            length = findParent(i,huffman,count);   //得到各个叶结点的带权路径长度
            sum += length*huffman[i][3] ; 	//各个叶结点带权路径长度累加得到总长度WPL
        }
    }
    printf("the tree's WPL  is  %d\n",sum);

    //Huffman编码
    for(int i = 0;i < count;i++)
    {
        HuffmanCode(i,huffman,count);
    }

    return 0;
}

//子函数——构建哈夫曼树
void huffmanTree(int w[],int huffman[][4],int n)
{
    //结点初始化
    for(int i = 0; i < 2 * n - 1; i++)
    {
        huffman[i][0] = -1;
        huffman[i][1] = -1;
        huffman[i][2] = -1;
        huffman[i][3] = -1;
    }
    //将各个结点的权值输入到构建二叉树的函数中
    for(int i = 0; i < n; i++)
    {
        huffman[i][3] = w[i];
    }
    //每次抽出两个权重最小的结点进行合并，直到最终产生根结点
    for(int i = n; i < 2 * n - 1; i++)
    {
        int i1,i2;  //权重最小的两个结点
        int ii[2]; 
        //找出两个权重最小的结点
        findMin(ii,huffman,n);
        i1=ii[0];
        i2=ii[1];
        //合并i1、i2结点,更新结点信息（新生成结点的左右子结点，子结点对应的父结点，新生成结点的权重）
        huffman[i][1] = i1;
        huffman[i][2] = i2;
        huffman[i][3] = huffman[i1][3] + huffman[i2][3];
        //huffman[i1][3] = -1;
        //huffman[i2][3] = -1;
        huffman[i1][0] = i;
        huffman[i2][0] = i;
    }
}

//子函数——找出两个权重最小的结点
void findMin(int ii[],int huffman[][4],int n)
{
    //找出第一个权重最小的结点
    int min = 9999999;
    for(int i = 0; i < 2 * n - 1; i++)
    {
        if(huffman[i][3] == -1 && huffman[i][0] == -1)  //忽略未赋值的结点
        {
            break;
        }
        if(huffman[i][3] < min && huffman[i][0] == -1)  //比较每个结点的权值大小
        {
            min = huffman[i][3];
            ii[0] = i;
        }
    }

    //找出第二个权重最小的结点
    int min_2 = 9999999;
    for(int j = 0; j < 2 * n - 1; j++)
    {
        if(huffman[j][3] == -1 && huffman[j][0] == -1)  //忽略未赋值的结点
        {
            break;
        }
        if(huffman[j][3] < min_2 && huffman[j][0] == -1 && huffman[j][3] != min)  //比较每个结点的权值大小
        {
            min_2 = huffman[j][3];
            ii[1] = j;
        }
    }   
}

//子函数——查找结点i的父结点,得到结点到根的长度
int findParent(int i,int huffman[][4],int n)
{
    int length = 0;
    if(huffman[i][0] == -1)     //检查结点是否为根结点（是则结束递归）
    {
        return 0;
    }
    length += (findParent(huffman[i][0],huffman,n) + 1);    //通过递归得到结点到根的路径长度
    return length;
}

//子函数——对每个叶结点进行哈夫曼编码并进行打印
void HuffmanCode(int i,int huffman[][4],int n)
{
    char code[30];  //char数组填充编码
    int current=i;  //定义当前访问的结点
    int father = huffman[i][0]; //定义当前结点的父结点
    int start=0;    //每次编码的位置，初始为编码倒数位置
    int first,last;     //char数组的头部和尾部

    while(father != -1)
    {
        if(huffman[father][1] == current)		//判断当前结点的父结点左子树是否为当前结点
        {
            code[start] = '0';                         //子结点是父结点的左子树，编码为0
        }else{
            code[start] = '1';                         //子结点是父结点的右子树，编码为1
        }		
        		

        current = father;           		//往上朔源，更新当前结点
        father = huffman[current][0];		//同理（当前结点更新后），更新当前结点的父亲结点
        start += 1;		//更新编码位置
    }
    code[start]='\0';   //编码结束符

    //将char数组中的元素头尾两端进行对调
    for(first = 0, last = start-1; first < last; first++,last--)
    {
        //对调数组内部元素
        inplace_swap(&code[first], &code[last]);	/*该函数可自己重写*/
    }

    printf("%c Huffman code:  %s\n",'A'+i,code);    //打印字符的huffman编码
}

//子函数——交换两个char型数据（使用了布尔运算），可自己另外用可读性较好的方法重新实现改函数
void inplace_swap(char *x,char *y)
{
    *y = *x ^ *y;
    *x = *x ^ *y;
    *y = *x ^ *y;
}
```

***

### 运行截图：
对字符串进行Huffman编码，得到WPL和各字符编码：

（1）DEAEDDEDBCEEECECDEEDDBEEECEEDEE
![alt text](image-3.png)

（2）EEBEACCCCAADACCBCCCCECCCEEECCCE
![alt text](image-4.png)

---

### 思考题

1. 当Huffman树的叶结点为n时，整个生成树的结点总数为何是2n-1？
结点总数等于度为0、1、2的结点数的和，其中度为1的结点数为0，度为2的结点数等于度为0的结点数减一，叶结点即度为0的结点，则n=2n-1。

2. 该代码可以对大写字母的字符串进行huffman编码，如果要读小写字母字符串进行编码，应该如何修改？如果想要对同时具有大写和小写字母的字符串都进行编码呢（写出思路即可）？
读小写：判断str[i] == (char)(0x41 + j)的时候中0x41改为0x61，printf("%c Huffman code:  %s\n",'A'+i,code)中'A'改成'a';
同时具有大小写：将 arr_big 数组的大小从 27 改为 53，判断条件变为str[i] == (char)(0x41 + j)||str[i] == (char)(0x41 + 7 + j)，同时j的范围变成0~51，打印哈夫曼编码时根据 i 的值是否小于26决定是打印大写字母还是小写字母，使用 'A' + i 或 'a' + (i - 26) 来打印相应字符。

3. 使用画图工具画出上述两个Huffman生成树的生成过程
![alt text](image-1.png)
![alt text](image-2.png)
