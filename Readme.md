# 实验一 202000800156 谭福华

## 简介

词法分析器是编译器中的一个组件，也称为词法分析器或扫描器。它将输入的源代码分解成一个个标记，也称为词法单元，这些标记可以是关键字、标识符、运算符、分隔符等。词法分析器通常是编译器的第一个阶段，其输出将成为后续阶段的输入。

|    语言    |      Java(jdk1.8)       |
| --------- | ----------------------- |
| 代码地址   | http:                   |
| 主函数入口 | /Analysis/Analysis.java |

## 函数执行顺序

* nextchar() 获取字符|读取文件
* step_judge() 判断字符类型
* 根据字符类型进入stringStep()或charStep()或者noteStep()或者digitStep()或者letterStep()或者signStepOrIllegal()
* 循环直至结束

## 函数说明

### nextChar() 
> 用途:读文件并获取下一个字符
1. 判断BufferedReader是否为空，为空则没有读取文件，使用reader = new BufferedReader(fileReader)去读文件
2. 判断reader是否还有字符可读，有则返回字符
3. 没有字符可读时，设置Analyisi类的全局变量isEnd为True表示结束，并返回'`'表示结束

```
    public char nextChar() {
        if (reader == null) {
            try {
                fileReader = new FileReader("C:\\Users\\谭福华\\Desktop\\CompilePrinciple-master\\src\\Analysis\\test.txt");
            } catch (FileNotFoundException e) {
                e.printStackTrace();
            }
            reader = new BufferedReader(fileReader);
        }
        try {
            if (reader.ready()) {
                setNowChar((char) reader.read());
                return getNowChar();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        isEnd = true;
        return '`';//结束
    }
```

### step_judge(char c)
> 接收一个字符判断:如果该字符是换行,空字符,制表符则忽略取下一个字符程序总体把分析分成6种种类去分析,并由初始模块根据第一字符原则判断进入哪个类别,分别是
1. 字符串:双引号关联起来的字串
2.  单个字符:单引号关联起来的字符
3.   注释:行注释以及块注释
4.   数字:自然数以及小数
5.   文字(letter):标识符以及关键字
6.   符号:操作符以及特殊符号或非法字符
```
    public void step_judge(char c) {
        while (c == '\n' || c == ' ' || c == '\r') {
            c = nextChar();
        }
        if (isEnd) {
            System.out.println("分析结束");
            try {
                fileReader.close();
                reader.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
            return;
        }
        if (c == '\"') {
            stringStep();
        } else if (c == '\'') {
            charStep();
        } else if (c == '/') {
            noteStep();
        } else if (Character.isDigit(c)) {
            digitStep();
        } else if (Character.isLetter(c) || c == '_') {
            letterStep();
        } else {
            signStepOrIllegal();
        }
    }

```

### stringStep()

>  进入字符串步骤,过滤第一种缺陷情况就是"\"",但是不能过滤第二重转义字符出现的缺陷.

```
    private void stringStep() {
        token = new StringBuffer(String.valueOf('\"'));
        char c;
        do {
            c = nextChar();
            token.append(c);
            if (c == '\"' && !token.toString().equals("\"\\\"")) {
                break;
            }
        } while (true);
        System.out.println(token + " :字符串");
        step0(nextChar());
    }
```

### charStep()
> 进入单字符步骤,可能出现转义字符打印错误,故提供一重修复.,鉴于深层情况少见,忽略
```
    private void charStep() {
        token = new StringBuffer(String.valueOf('\''));

        char c;
        do {
            c = nextChar();
            token.append(c);
            if (c == '\'' && !token.toString().equals("'\\'")) {
                break;
            }
        } while (true);
        System.out.println(token + " :单字符");
        step0(nextChar());
    }
```

### noteStep()
>进入注释步骤:行注释以及块注释
```
    private void noteStep() {
        token = new StringBuffer(String.valueOf('/'));
        char c = nextChar();

        if (c == '/') {//行注释
            token.append('/');
            token.append(getRemainLine());
        } else if (c == '*') {//块注释
            token.append('*');
            token.append(getRemainBlock());
        } else {
            System.out.println("注释代码未知情况");
        }
        System.out.println(token + " :注释");
        step0(nextChar());
    }
```

### getRemainBlock()
>返回注释块字符串,策略是一直扫描直到扫描到*和/符号
```
    private String getRemainBlock() {
        StringBuffer buffer = new StringBuffer();
        char c;
        char c2;
        while (true) {
            c = nextChar();
            if (c == '\t')
                continue;
            buffer.append(c);
            if (c == '*') {
                c2 = nextChar();
                if (c2 == '\t')
                    continue;

                if (c2 == '/') {//如果继*后的符号是斜杠,那么就退出循环
                    buffer.append(c2);
                    break;
                }
                buffer.append(c2);
            }
        }
        return buffer.toString();
    }
```
### getRemainLine()
>返回行注释的字符串,扫描策略是直接扫描直到换行符
```
    private String getRemainLine() {
        StringBuffer buffer = new StringBuffer();
        char c;
        while (true) {
            c = nextChar();
            if (c == '\n' || c == '\r') {
                break;
            }
            buffer.append(c);
        }
        return buffer.toString();
    }
```
### digitStep()
>注入数字过程:不包括正负号,最多只能出现一个点.
```
    private void digitStep() {
        boolean dot = false;
        token = new StringBuffer(String.valueOf(getNowChar()));//把当前数字加入

        char c = nextChar();
        while (c == '.' || Character.isDigit(c)) {
            if (c == '.') {
                if (dot) {//就是点已经出现过了
                    break;
                }
                dot = true;
            }
            token.append(c);//点或者数字都加入字串
            c = nextChar();
        }
        System.out.println(token + " :数字");

        step0(getNowChar());
    }
```
### letterStep()
>进入文字过程
```
    private void letterStep() {
        token = new StringBuffer(String.valueOf(getNowChar()));//吧当前的字符串加入
        char c = nextChar();
        while (c == '_' || Character.isDigit(c) || Character.isLetter(c)) {
            token.append(c);
            c = nextChar();
        }

        if (keyWork.contains(token.toString())) {
            System.out.println(token + " :关键字");
        } else {
            System.out.println(token + " :标识符");
        }

        step0(getNowChar());
    }
```
### signStepOrIllegal()
>    进入符号阶段,在这里分别区分非法字符,操作符,特殊字符
```
    private void signStepOrIllegal() {
        token = new StringBuffer();
        char c = getNowChar();
        StringBuffer buffer = new StringBuffer(String.valueOf(getNowChar()));
        while (opera.contains(buffer.toString()) || special.contains(buffer.toString())) {
            token.append(c);
            c = nextChar();
            buffer.append(c);
        }
        if (token.length() != 0) {//非法字符
            if (special.contains(token.toString())) {
                System.out.println(token + " :界符");
            } else {
                System.out.println(token + " :操作符");
            }
        } else {
            System.out.println(getNowChar() + " :非法字符");
        }
        step0(getNowChar());
    }
```

## 测试

### 输入
```
public class Puppy{
   int puppyAge;
   public Puppy(String name){
      // 这个构造器仅有一个参数：name
      System.out.println("小狗的名字是 : " + name );
   }

   public void setAge( int age ){
       puppyAge = age;
   }

   public int getAge( ){
       System.out.println("小狗的年龄为 : " + puppyAge );
       return puppyAge;
   }

   public static void main(String []args){
      /* 创建对象 */
      Puppy myPuppy = new Puppy( "tommy" );
      /* 通过方法来设定age */
      myPuppy.setAge( 2 );
      /* 调用另一个方法获取age */
      myPuppy.getAge( );
      /*你也可以像下面这样访问成员变量 */
      System.out.println("变量值 : " + myPuppy.puppyAge );
   }
}
```

### 分析结果
```
public :关键字
class :关键字
Puppy :标识符
{ :界符
int :关键字
puppyAge :标识符
; :界符
public :关键字
Puppy :标识符
( :界符
String :标识符
name :标识符
) :界符
{ :界符
// 这个构造器仅有一个参数：name :注释
System :标识符
. :界符
out :标识符
. :界符
println :标识符
( :界符
"小狗的名字是 : " :字符串
+ :操作符
name :标识符
) :界符
; :界符
} :界符
public :关键字
void :关键字
setAge :标识符
( :界符
int :关键字
age :标识符
) :界符
{ :界符
puppyAge :标识符
= :操作符
age :标识符
; :界符
} :界符
public :关键字
int :关键字
getAge :标识符
( :界符
) :界符
{ :界符
System :标识符
. :界符
out :标识符
. :界符
println :标识符
( :界符
"小狗的年龄为 : " :字符串
+ :操作符
puppyAge :标识符
) :界符
; :界符
return :关键字
puppyAge :标识符
; :界符
} :界符
public :关键字
static :关键字
void :关键字
main :标识符
( :界符
String :标识符
[ :界符
] :界符
args :标识符
) :界符
{ :界符
/* 创建对象 */ :注释
Puppy :标识符
myPuppy :标识符
= :操作符
new :关键字
Puppy :标识符
( :界符
"tommy" :字符串
) :界符
; :界符
/* 通过方法来设定age */ :注释
myPuppy :标识符
. :界符
setAge :标识符
( :界符
2 :数字
) :界符
; :界符
/* 调用另一个方法获取age */ :注释
myPuppy :标识符
. :界符
getAge :标识符
( :界符
) :界符
; :界符
/*你也可以像下面这样访问成员变量 */ :注释
System :标识符
. :界符
out :标识符
. :界符
println :标识符
( :界符
"变量值 : " :字符串
+ :操作符
myPuppy :标识符
. :界符
puppyAge :标识符
) :界符
; :界符
} :界符
} :界符
分析结束
语法正确
```