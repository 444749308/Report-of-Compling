# Report-of-Compling


## 实验内容1

### 说明文件

> 类型说明

```
    TOKENVAL:当前(lookahead)属性值;

    TERMINAL:当前(lookahead)记号名;

    EXPVALUE:变量值,只有无符号整形与字符型;

    EXPVAL:疑似多余?

    IDTABLE:存储变量信息的链表(栈);
```

>  词法分析DFA转换表


![](https://github.com/444749308/Report-of-Compling/blob/master/Picture/%E8%AF%8D%E6%B3%95%E5%88%86%E6%9E%90DFA%E8%BD%AC%E6%8D%A2%E8%A1%A8.PNG)
   
    
    10*表示接收状态，但多读了一个有用符号，需要缓存
    
    20*表示接收状态，没有多读有用符号
    
 >  词法分析中部分函数,变量说明
    
    static char ReadAChar();//从文件中读入下一个字符或从缓存(prebuf)中读入字符;
    
    static int FoundRELOOP();//关系符判断;
    
    static int FoundKeyword();//关键字判断; 
    
    static char prebuf=0;    //缓存多读的一个字符;
    
    static char tokenStr[MAXTOKENLEN];    //目前读取的词法单元;
    
    
>  语法分析
    
![](https://github.com/444749308/Report-of-Compling/blob/master/Picture/%E8%AF%AD%E6%B3%95.PNG)
    
> 语法分析中部分函数,变量说明
    
    Prod开头的函数都分别对应语法分析中的非终结符.
    
    static TERMINAL lookahead;//下一个读到的词法单元
    
    static int curtoken_num;//当前读到的整型常量
    
    static char curtoken_str[MAXTOKENLEN];//当前读到的变量(id)
    
    static char curtoken_ch;//当前读到的字符型常量
    
    static IDTABLE *IDTHead=NULL;//存储变量的链表头部
    
    static int run_status = 1;//0；程序不执行；1:程序正常执行；2:if跳过 3:while 跳过

    

### 部分错误修改

> LexicalAnalysis.c
>>   1.当读到由两个关系符组成的关系符时,不需要缓存后一个关系符,故修改 `FoundRELOOP`中两个符号的关系符,去除后一个符号的缓存.

```c
    else if (tokenStr[0]=='=' && tokenStr[1]=='=') { prebuf = 0; return(SYN_EQ); }
```

```c
    else if (tokenStr[0]=='!' && tokenStr[1]=='=') { prebuf = 0; return(SYN_NE); }
    else if (tokenStr[0]=='&' && tokenStr[1]=='&') { prebuf = 0; return(SYN_AND); }
    else if (tokenStr[0]=='|' && tokenStr[1]=='|') { prebuf = 0; return(SYN_OR); }
```

>>> 测试程序:

```c
main()
{ int i=0;
if(i==0)
{
  show(111);
}
}
```
>>> 源程序无结果,修改后输出 `111`

> SyntaxAnalysis.c
>> 1.在`Prod_S`中,在结束一次while循环恢复指针后,要再读一次文件,否则lookahead仍为`}`

```c
        if (run_status==1)
        {    fseek(sFile,file_index,SEEK_SET);
            renewLex();
            lookahead = nextToken();//再读一次文件
        }
```

>> 2.在`Prod_TB1`中有笔误,将bval1改为bval2


```c
        bval2=Prod_FB();//错误修改
```

>> 3.当while条件不满足时,若其循环内存在if-else语句,则会更改run_status,使其后面原本无效的代码继续执行,并且回到while起点.

>> 产生该错误的原因是run_status的状态区分不够,将原本的3个状态增添到4个,特别区分是if跳过还是while的跳过.
```c static int run_status = 1;//0；程序不执行；1:程序正常执行；2:if跳过 3:while跳过 ```
>>> 测试程序
```c
main()
{
  int i = 0;
  while(i < 10)
  {
    if (i == 5)
    {
      show(1);
    }
    else 
    {
      show(0);
    }
    i = i + 1;
  }
}
```
>>> 原结果:死循环

>>> 修改后结果
```
0000010000
```

>> 4.不支持多重循环,将`file_index`的值作为形参每次传入`Prod_S`中即可.
>>> 测试程序
```c
main()
{
  int i = 0,j = 0;
  while(i < 10)
  {
      while (j < 10 )
      {
        show (i * 10 + j);
        show(' ');
        j = j + 1;
      }
      show('\n');
      j = 0;
      i = i + 1;
  }
}
```
>>> 修改后结果
```
0 1 2 3 4 5 6 7 8 9
10 11 12 13 14 15 16 17 18 19
20 21 22 23 24 25 26 27 28 29
30 31 32 33 34 35 36 37 38 39
40 41 42 43 44 45 46 47 48 49
50 51 52 53 54 55 56 57 58 59
60 61 62 63 64 65 66 67 68 69
70 71 72 73 74 75 76 77 78 79
80 81 82 83 84 85 86 87 88 89
90 91 92 93 94 95 96 97 98 99
```



## 实验内容2

> 增添char型主要是修改词法部分,使其能正确的识别字符型,故在词法分析中加入对单引号部分的分析,其自动机如下:

![](https://github.com/444749308/Report-of-Compling/blob/master/Picture/char%E8%87%AA%E5%8A%A8%E6%9C%BA.PNG)

> 更改后的dfa分析表如实验内容1中所示.

> constvar.h

>> 1.修改`TOKENVAL`类型,增添字符型;


```c
    typedef union {
    	int number;
    	char ch;//新增char型
    	char *str;
    } TOKENVAL;
```
>> 2.对应DFA转换表,转换表增加三行两列,-1表示ERR;

```c
static int LexTable[9][10]=
   {	{   1, 201, 204,   2,   3,   4,   5, 205,   6,  -1},
	{ 101, 101, 101, 101, 101, 101, 101, 101, 101,  -1},
	{ 102, 102, 202, 203, 102, 102, 102, 102, 102,  -1},
	{ 103, 103, 103, 103, 103, 103, 103, 103, 103,  -1},
	{ 104, 104, 104, 104, 104,   4, 104, 104, 104,  -1},
	{ 105, 105, 105, 105, 105,   5,   5, 105, 105,  -1},
	{   7,   7,   7,   7,   7,   7,   7,   7, 206,   8},//添加三行
	{  -1,  -1,  -1,  -1,  -1,  -1,  -1,  -1, 206,  -1},
	{   7,   7,   7,   7,   7,   7,   7,   7,   7,   7}};
```

>> 3.新增常量,用于字符与单引号的匹配


```c
#define SYN_CH		3		// 字符型
#define SYN_QUOTA	28	 	//’
```

> LexicalAnalysis.c

>> 1.新增常量,用于单引号与转义字符的匹配

```c
#define LEX_QUOTA	8	//’
#define LEX_ESC	    	9	//转义字符
```
>> 2.新增读入字符是单引号,转义字符的处理(nextToken内)


```c
		if (feof(sFile))
			state=LexTable[state][LEX_DELIM];
		else if (c=='<' || c=='>' || c=='=' || c=='!' || c=='&' || c=='|')
			state=LexTable[state][LEX_RELOOP];
		else if (c==' ' || c=='\t' || c=='\n')
			state=LexTable[state][LEX_DELIM];
		else if (c=='*')
			state=LexTable[state][LEX_MUL];
		else if (c=='/')
			state=LexTable[state][LEX_DIV];
		else if (c=='+' || c=='-')
			state=LexTable[state][LEX_ADDMIN];
		else if (c>='0' && c<='9')
			state=LexTable[state][LEX_DIGIT];
		else if ((c>='a' && c<='z')||(c>='A' && c<='Z')||(c=='_'))
			state=LexTable[state][LEX_LETTER_];
		else if (c=='(' || c==')' || c=='{' || c=='}' || c==',' || c==';')
			state=LexTable[state][LEX_SYMBOL];
		//新增char型词法
		else if (c == '\'')
            		state = LexTable[state][LEX_QUOTA]
        	else if (c == '\\')
            		state = LexTable[state][LEX_ESC]
        	//新增char型词法完结
        	else
		{	printf("Unknown symbol: %c\n",c);
			break;
		}
		//报错
		if (state == ERR)
            		FreeExit();
		//报错
```

>>  3.对单引内部分单独处理(nextToken内)


```c
            //新增char型词法
            case 206:   token.token = SYN_CH;
                        if (tokenStr[1]=='\\')  EscTrans(tokenStr[2]);
                        else token.tokenVal.str=tokenStr[1]; 
                        break;
            //新增char型词法完结
```
>>  4.新增函数`void EscTrans(char c)`在3中调用,用于去除转义


```c
void EscTrans(char c)
{
    token.token=SYN_CH; 
    switch (c)
    {//将转义字符转变为acsii码
        case 'n':   token.tokenVal.ch = 10;
                    break;
        case 't':   token.tokenVal.ch = 9;
                    break;
        case '\\':  token.tokenVal.ch = 92;
                    break;
        case '\'':  token.tokenVal.ch = 39;
                    break;
        case '/"':  token.tokenVal.ch = 34;
                    break;
        case '\?':  token.tokenVal.ch = 63;
                    break;
        case '\0':  token.tokenVal.ch = 0;
                    break;
        default:    FreeExit();
    }
}
```

> SyntaxAnalysis.c(此处修改基本对称于整型)

>>  1.增加变量`curtoken_ch`来存储字符值,对应于其他`curtoken_`

```c
static char curtoken_ch;
```
>>  2.在匹配下一部分时存储字符(match内)


```c
	if (lookahead.token == t)
	{	if (t==SYN_NUM)
			curtoken_num=lookahead.tokenVal.number;
		else if (t==SYN_ID)
			for (p=lookahead.tokenVal.str,q=curtoken_str;(*q=*p)!='\0';p++,q++);
		//新增存储char值
        	else if (t == SYN_CH)
            		curtoken_ch = lookahead.tokenVal.ch;
		//新增存储char值完结
	}		
```

>>  3.为字符入栈准备

```c
	else if (lookahead.token==SYN_CH)
    	{
        	match(SYN_CH);
		val.type=ID_CHAR;
		val.val.charval=curtoken_ch;
    	}
```

>测试

>>测试1:

>>>源程序
```c
main()
{
  char c,s;
  c = 'a';
  s = 'b';
  if (c + 1 == s)
  {
    show (c);
    show('\n');
    show(s);
  }
}
```
>>>输出
```
a
b
```

>> 测试2
>>> 源程序
```c
main()
{
  char c;
  int i = 0; 
  c = 'a';
  while (c <= 'z')
  {
    show(c);
    show(' ');
    i = i + 1;
    if(i == 5)
    {
      i = 0;
      show('\n');
    }
    c = c + 1;
  }
}
```
>>> 输出
```
a b c d e
f g h i j
k l m n o
p q r s t
u v w x y
z
```



## 实验内容3
> 更改的语法部分已在实验内容1中用红色标识出来
    思路:由于逻辑运算的优先级低于算术运算,故将原先所有的非终结符E替换为B,然后再由B=>* E,即可以在保留原算术计算的同时实现逻辑运算.重点在与如何完美的衔接逻辑运算与算术运算.修改F<sub>B</sub>,添加<sub>B1</sub>,使得逻辑运算可以多次进行,并且与算术运算联系了起来;
    
> SyntaxAnalysis.c

>> 1.修改变量类型,把原本逻辑表达式所返回的int型改为EXPVAL,使其类型与算术表达式统一,涉及到以下函数(声明)与变量(声明):

```c
static EXPVAL Prod_B();
static EXPVAL Prod_B1(EXPVAL bval);
static EXPVAL Prod_TB();
static EXPVAL Prod_TB1(EXPVAL bval);
static EXPVAL Prod_FB();
static EXPVAL Prod_FB1(EXPVAL val1);
static EXPVAL Prod_E();
static EXPVAL Prod_E1(EXPVAL val);
static EXPVAL Prod_TE();
static EXPVAL Prod_TE1(EXPVAL val);
static EXPVAL Prod_F();
```
>> 这些函数中的变量`bval`,`bavl1`,`bval2`也都改为`EXPVAL`类型.

>> 并且把之前的逻辑值都存储在`.val.intval`中.

>> 修改后的F<sub>B</sub>:

```c
static EXPVAL Prod_FB()
{
    EXPVAL val1,val2;
    val1=Prod_E();
    val2=Prod_FB1(val1);
    return (val2);
}
```

>> 新添加的的F<sub>B1</sub>:

```c
static EXPVAL Prod_FB1(EXPVAL val1)
{
    EXPVAL bval;
    EXPVAL val2;
    int ival1,ival2;
    bval.type = ID_INT;
    bval.val.intval = 0;
    if (run_status==1) ival1=cast2int(val1);
    if (lookahead.token==SYN_LT)
    {
        #if defined(AnaTypeSyn)
        printf("SYN: FB-->E<E\n");
        #endif
        match(SYN_LT);
        val2=Prod_E();
        if (run_status==1)
        {	
	    ival2=cast2int(val2);
            bval.val.intval = (ival1<ival2 ? 1 : 0);
            bval=Prod_FB1(bval);
            return(bval);
        }
        else
            return(bval);
    }
    else if (lookahead.token==SYN_LE)
    {
        #if defined(AnaTypeSyn)
        printf("SYN: FB-->E<=E\n");
        #endif
        match(SYN_LE);
        val2=Prod_E();
        if (run_status==1)
        {	ival2=cast2int(val2);
            bval.val.intval = (ival1<=ival2 ? 1 : 0);
            bval=Prod_FB1(bval);
            return(bval);
        }
        else
            return(bval);
    }
    else if (lookahead.token==SYN_GT)
    {
        #if defined(AnaTypeSyn)
        printf("SYN: FB-->E>E\n");
        #endif
        match(SYN_GT);
        val2=Prod_E();
        if (run_status==1)
        {	
	    ival2=cast2int(val2);
            bval.val.intval = (ival1>ival2 ? 1 : 0);
            bval=Prod_FB1(bval);
            return(bval);
        }
        else
            return(bval);
    }
    else if (lookahead.token==SYN_GE)
    {
        #if defined(AnaTypeSyn)
        printf("SYN: FB-->E>=E\n");
        #endif
        match(SYN_GE);
        val2=Prod_E();
        if (run_status==1)
        {	
	    ival2=cast2int(val2);
            bval.val.intval = (ival1>=ival2 ? 1 : 0);
            bval=Prod_FB1(bval);
            return(bval);
        }
        else
            return(bval);
    }
    else if (lookahead.token==SYN_EQ)
    {
        #if defined(AnaTypeSyn)
        printf("SYN: FB-->E==E\n");
        #endif
        match(SYN_EQ);
        val2=Prod_E();
        if (run_status==1)
        {	
	    ival2=cast2int(val2);
            bval.val.intval = (ival1==ival2 ? 1 : 0);
            bval=Prod_FB1(bval);
            return(bval);
        }
        else
            return(bval);
    }
    else if (lookahead.token==SYN_NE)
    {
        #if defined(AnaTypeSyn)
        printf("SYN: FB-->E!=E\n");
        #endif
        match(SYN_NE);
        val2=Prod_E();
        if (run_status==1)
        {	
	    ival2=cast2int(val2);
            bval.val.intval = (ival1!=ival2 ? 1 : 0);
            bval=Prod_FB1(bval);
            return(bval);
        }
        else
            return(bval);
    }
    else
    {
        if (run_status==1)
        {
            return(val1);
        }

        else
            return(bval);
    }
}
```

>> 修改后的F:

```c

static EXPVAL Prod_F()
{//
	EXPVAL val,bval;
	static IDTABLE *p;
	if (lookahead.token==SYN_NUM)
	{
		#if defined(AnaTypeSyn)
		printf("SYN: F-->num\n");
		#endif
		match(SYN_NUM);
		val.type=ID_INT;
		val.val.intval=curtoken_num;
	}
	else if (lookahead.token==SYN_ID)
	{
		#if defined(AnaTypeSyn)
		printf("SYN: F-->id\n");
		#endif
		match(SYN_ID);
		p=LookupID();
		val.type=p->type;
		val.val=p->val;
	}
	else if (lookahead.token==SYN_PAREN_L)
	{
		#if defined(AnaTypeSyn)
		printf("SYN: F-->(B)\n");
		#endif
		match(SYN_PAREN_L);
		val=Prod_B();
		match(SYN_PAREN_R);
	}
	else if (lookahead.token==SYN_CH)
    	{
        match(SYN_CH);
		val.type=ID_CHAR;
		val.val.charval=curtoken_ch;
    	}
	else if (lookahead.token==SYN_NOT)
	{
		#if defined(AnaTypeSyn)
		printf("SYN: FB-->!B\n");
		#endif
		match(SYN_NOT);
		bval=Prod_B();
		bval.val.intval = (run_status==1 ? 1-bval.val.intval : 0);
		return(bval);
	}
	else if (lookahead.token==SYN_TRUE)
	{
		#if defined(AnaTypeSyn)
		printf("SYN: FB-->TRUE\n");
		#endif
		match(SYN_TRUE);
		bval.val.intval = (run_status==1 ? 1 : 0);
		return(bval);
	}
	else if (lookahead.token==SYN_FALSE)
	{
		#if defined(AnaTypeSyn)
		printf("SYN: FB-->FALSE\n");
		#endif
		match(SYN_FALSE);
		bval.val.intval = 0;
		return(bval);
	}
    else
		FreeExit();
	return(val);
}
```

>> 之前原程序中的以1表示true都改为不等于0为true

>>测试1
>>> 源程序
```c
main()
{
  show(1<2<3<4>0);
}

```
>>> 输出
```
1
```
>> 测试2 
>>> 源程序
```c
main()
{
  show((1<2>0 && 1+2*3 || 1==2)+(1<2)*4);
}

```
>>> 输出
```
5
```

##实验内容4

> break即到达该while的大括号前都将run_status置3,并在读完该大括号后将run_status在置回1;
> continue即直接把文件指针移回之前记录的while循环开始处.

> constvar.h

>> 添加状态
```c
#define SYN_CONTINUE 62     //continue
#define SYN_BREAK    63     //break
```

>> LexicalAnalysis.c

>>> 在关键字查找`FoundKeyword()`中增加对break和continue的查找

```c
	if (strcompare(tokenStr,"continue")) return(SYN_CONTINUE);
	if (strcompare(tokenStr,"break")) return(SYN_BREAK);
```

>> SyntaxAnalysis.c

>>> 在`Prod_S`中添加break的判断,即将run_status置3

```c
    else if (lookahead.token==SYN_BREAK)
    {
        match(SYN_BREAK);
        match(SYN_SEMIC);
        if (file_index != -1 )
        {
            if (run_status==1)
            {
                run_status = 3;
                Prod_S(file_index);
                return 0;
            }
        }
        else
        {
            printf("BREAK_ERROR!");
            FreeExit();
        }
    }
```

>>> 在`Prod_S`中添加continue的判断,即直接将文件指针移回while的开始处

```c
    else if (lookahead.token == SYN_CONTINUE)
    {
        match(SYN_CONTINUE);
        match(SYN_SEMIC);
        if (file_index != -1 )
        {
            if (run_status==1)
            {
                fseek(sFile,file_index,SEEK_SET);
                renewLex();
                lookahead = nextToken();//再度一次文件
                Prod_S(file_index);
                return 0;
            }
        }
        else
        {
            printf("CONTINUE_ERROR!");
            FreeExit();
        }
    }
```


>> 测试
>>> 源程序
```c
main()
{
  int i = 0,j = 0;
  while(i < 10)
  {
      while (j < 10 )
      {
        show (i * 10 + j);
        show(' ');
        j = j + 1;
        if(j == 5)
        {
          j = j + 1;
          continue;
        }
      }
      show('\n');
      j = 0;
      i = i + 1;
      if (i == 8)
      {
        break;
      }
  }
}
```
>>> 输出
```c
0 1 2 3 4 6 7 8 9
10 11 12 13 14 16 17 18 19
20 21 22 23 24 26 27 28 29
30 31 32 33 34 36 37 38 39
40 41 42 43 44 46 47 48 49
50 51 52 53 54 56 57 58 59
60 61 62 63 64 66 67 68 69
70 71 72 73 74 76 77 78 79
```
