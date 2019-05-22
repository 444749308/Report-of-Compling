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

```
  源程序无结果,修改后输出`111`
```

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


## 实验内容2
> 更改的语法部分已在实验内容1中用红色标识出来
    思路:由于逻辑运算的优先级低于算术运算,故将原先所有的非终结符E替换为B,然后再由B=>* E,即可实现逻辑运算
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
        {	ival2=cast2int(val2);
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
        {	ival2=cast2int(val2);
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
        {	ival2=cast2int(val2);
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
        {	ival2=cast2int(val2);
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
        {	ival2=cast2int(val2);
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
}```

