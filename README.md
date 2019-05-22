# Report-of-Compling


## 实验内容1
* 说明文件
	* 类型说明
		  * TOKENVAL:当前(lookahead)属性值;
          
      * TERMINAL:当前(lookahead)记号名;
    
      * EXPVALUE:变量值,只有无符号整形与字符型;
    
      * EXPVAL:疑似多余?
    
      * IDTABLE:存储变量信息的链表(栈);
  * 词法分析DFA转换表
  
    ![](https://github.com/444749308/Report-of-Compling/blob/master/Picture/%E8%AF%8D%E6%B3%95%E5%88%86%E6%9E%90DFA%E8%BD%AC%E6%8D%A2%E8%A1%A8.PNG)
    
    * 10*表示接收状态，但多读了一个有用符号，需要缓存
    * 20*表示接收状态，没有多读有用符号
  * 词法分析中部分函数,变量说明
	
  	* static char ReadAChar();//从文件中读入下一个字符或从缓存(prebuf)中读入字符;
		
	* static int FoundRELOOP();//关系符判断;

  	* static int FoundKeyword();//关键字判断; 

  	* static char prebuf=0;	//缓存多读的一个字符;
    
  	* static char tokenStr[MAXTOKENLEN];	//目前读取的词法单元;
* 语法分析

  ![](https://github.com/444749308/Report-of-Compling/blob/master/Picture/%E8%AF%AD%E6%B3%95.PNG)
