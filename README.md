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
>> 1.在`Prod_S`中,在结束一次while循环恢复指针后,要再读一次文件

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
