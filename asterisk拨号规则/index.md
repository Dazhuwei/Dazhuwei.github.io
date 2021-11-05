# Asterisk拨号规则



# Asterisk拨号规则

##  一、前言

  本文档以asterisk-1.4.32为基础写作而成，可能和其他版本有些区别。其中参考了一些别的书籍和文章。因为写的比较仓促，而且基本都是晚上写的，里面的内容逻辑性和语句没有仔细斟酌，就是想到什么写什么，难免有什么遗漏和错误的地方，大家发现请及时通知我修改。另外这是我第一次写技术性的文章还很嫩涩，算是一个开始，希望大家多多支持。

## 二、Asterisk dialplan 基本结构

  Asterisk dialplan 的语法可以分为四个关键点，也就是语法结构的四个组成部分，四个部分分别context ，extensionnum ，priority 和 action。由这四个组成部分dialplan的结构为：

```shell
 [context]
  exten => extensionnum,priority,action
```

###   1、context

  context是指dialplan的流程块，整个dialplan就是由每个context的内容组成，他们协作完成整个asterisk命令逻辑的运转。context的名字必须放在中括号之中，比如PSTN外线打进系统所执行的流程我们都习惯叫from-pstn，在语法里面就写做"[from-pstn]"。所有属于这个流程的内容都写在这个下面。每一个命令都由换行符来隔开，也就是说每一行就是一个命令，每一行命令都必须由"exten => "（这个里面的空格可以没有）开头。流程的结尾就是遇到到下一个流程标识截止。

###   2、extensionnum
```
  extensionnum是指流程块里面的流程匹配标识（也就是asterisk里面说的extension），这个匹配标识其实通常就是我们要拨的号码（当然这个匹配标识不光是数字也可以是字母或者一些特殊字符）。比如你拨分机101，而你设置的拨分机的流程块是dial-ext，那么asterisk就会在dial-ext流程块里面寻找能匹配101的流程，找到了就会执行。说到匹配大家就会想到通配符吧，哈哈，asterisk里面也有类似的通配符，下面我就介绍一下asterisk里面关于extension的通配符。
   X和x表示单个0-9的数字
   N和n表示单个1-9的数字
   Z和z表示单个2-9的数字
   .表示单个的任何字符和数字
   []中括号里面可以是你想任意的匹配的数字或者字母，比如你想匹配1、3或者6，那么你就可以这样[136]或者[1,3,6]，在中括号里面还支持这样[1-8]是指匹配1到8的任意一个数字。
   当你的extensionnum中含有任何通配符的时候你就要用一个短的下划线"_"来作为extensionnum的开头除了这些以外asterisk还有一些特殊意义的匹配字符，
   s ：是指Start extension，也就是当没有extension的时候就会执行这个流程（例如在模拟外线进线没有收到callerID的情况下就会转到这个extension来执行），另外在zapata.conf的channels段里面如果设定了immediate=yes程序就会自动找到s这流程来执行。
   t ：是指timeout extension，也就是说如果等待用户输入超时后就会转到t这个流程来执行，在这里你可以设置一些提示音来告诉客户超时了。
   i ：是指invalid extension，也就是说如果客户输入无效的时候会转到i这个流程来执行
   fax ：是指fax calls，也就是说如果asterisk检测到传真信号的时候就会自动转到这个流程里面来执行。
   h ：是指hangup extension，就是说呼叫终止后执行的流程，在这里通道已经终止，放音、发送DTMF等命令都不可用了，只能做一些呼叫结束后处理的一些工作。
   需要注意的是，不管你设置的流程是什么都是属于某个流程块的，在不相互包含的情况下，流程块与流程块之间是相互独立的，流程或者变量是不会冲突的。
```

###   3、priority

  priority是指流程里面的命令的执行优先级，除了跳转的情况，都是按照priority值的之从小到大执行的。在流程里面你会经常看见"n"这个priority，它是指next也就是上一个priority+1.我们通常还会给某个priority取个名字，来方便我们流程跳转，也方便流程的阅读。语法是这样的 exten => extensionnum,priority(name),action ，name就是这个priority的名字。

```shell
  exten => _123409XX,1,GotoIf($[${EXTEN}=12340910]?ivr1:normal) 
   exten => _123409XX,n(normal),Dial(SIP/101)
   exten => _123409XX,n,Goto(end)
   exten => _123409XX,n(ivr1),BackGround(MyVox/Greeting)
   exten => _123409XX,n(end),Hangup
```

  上面这段代码就是当匹配的号码是12340901时就跳转到ivr1这个priority上否则就跳转到normal这个priority上。上面这段流程也可以写成：

```shell
  exten => _123409XX,1,GotoIf($[${EXTEN}=12340910]?4:2) 
   exten => _123409XX,2,Dial(SIP/101)
   exten => _123409XX,3,Goto(5)
   exten => _123409XX,4,BackGround(MyVox/Greeting)
   exten => _123409XX,5,Hangup
```



###   4、action

  action就相对好理解多了就是流程里面你要执行的命令，asterisk支持的命令很多，我就介绍几个比较常用的。所有命令的规则里中括号里面是可选的部分，没有中括号的必须填加的。参数之间的分隔符是"|",其实","来分隔也可以，但是1.6版本的asterisk不行，1.6的只能用"|"。还应该注意的是action的命令名有大小写之分，写错将会出现错误，当命令没有参数时可以不用写小括号。

####   1)、Answer

  语法：

```shell
Answer([delay])
```

  应答一个振铃的Channel。如果呼叫没有被应答此方法将应答这个呼叫，如果delay这个参数指定那么应答成功后将等待delay秒后再返回执行下条命令，应答不成功就会直接返回错误返回值而不执行delay延迟。这个命令用于呼入的情况，呼出的时候不会真正执行answer操作但是会执行delay延迟。

####   2)、Playback

  语法：

```shell
Playback(filename[&filename2...][|option])


  给当前Channel播放语音文件。第一个参数是语音文件，语音文件的格式asterisk默认的是gsm，其他wav、sln、vox、pcm等也都支持，但是最好用gsm格式的文件，如果没有gsm格式的可以用sox这个工具把格式变为gsm的，一般变成gsm格式的语音播放起来就不会有什么问题了。
  播放的语音文件也可以是多个，每个之间用"&"来分隔，语音文件的扩展名不用写，语音文件中不能有空格，最好就是字母和数字的组合，为了语义的需要可以用下划线来分隔。
  当一个语音文件出现问题时命令就会停止，后面的语音文件就不会再被播放了。
  语音文件是相对地址也可以是绝对地址，相对地址的默认路径是/var/lib/asterisk/sounds/。
   第二个参数是一些控制变量，这个参数是可选的：
   skip ：如果这个变量设置，而且当前通道没有UP，命令就会立刻结束，不播放任何语音。
   noanswer ：如果设置这个变量，在当前通道没有UP的时候，就不会执行anwser来UP这个通道。
   say ：如果设置这个变量，在播放语音之前会把要播放的语音文件名都出来。
   j ：如果设置这个变量，当语音文件播放出现问题的时候，流程就会跳转到priority+101这个priority继续执行。
   通道变量PLAYBACKSTATUS，通道变量的值通常都是一个字符串，当语音播放成功后PLAYBACKSTATUS为SUCCESS，否则为FAILED。当播放多个文件时只要有一个文件播放失败PLAYBACKSTATUS即为FAILED。
```

####    3)、BackGround

   语法：

```shell
Background(filename1[&filename2...][|options[|langoverride][|context]])
```

   给当前Channel播放语音文件，并等待客户输入，来执行相应的extension。第一个参数是语音文件，这个参数的用法跟Playbcak的第一参数用法一样。
   第二个参数是一些控制变量，这个参数是可选的：
   s ：如果这个变量设置，而且当前通道没有UP，命令就会立刻结束，不播放任何语音。
   n ：播放语音前不用answer这个通道
   m ：只有当用户输入的能和参数context中的流程匹配才结束。
   p ：只播放不接受用户输入。
   第三个参数是指播放声音的语言，这个参数是可选的。
   第四个参数是指当用户输入后要去寻找匹配并执行的context，在多层的IVR中这个参数是关键，这个参数是可选的。

####   4)、Dial

  语法：

```shell
Dial(Technology/resource[&Tech2/resource2...][|timeout][|options][|URL])


  这个命令会是当前通道呼叫一个或多个Channel，其中有一个Channel应答，当前通道就会和这个Channel桥接在一起，其他Channel就会挂断。
  第一个参数是要呼叫的通道可以是多个每个之间用"&"来分隔。
  第二个参数是超时时间，单位为秒，如果不设置超时时间，呼叫就会一直等到对方应答为止。
  第三个参数是一些控制变量：
   A(x) ：当被叫方应答的时候给被叫方播放一段语音，x为要播放的语音文件
   C ：重新设置CDR
   d ：允许主叫方在等待被叫方应答的时候，按一个数字键跳转到这个数字所能匹配的流程中，新的流程是指定在EXITCONTEXT变量中设置的流程，如果EXITCONTEXT没有被指定那么就在当前context中寻找。
   D([called][:calling]) ：发送DTMF到主叫方或者被叫方，当被叫应答但是通道还没有桥接的时候。
   f ：强制为被叫方Channel设置CallerID，用当前的extension
   g ：当对方挂机的后，接着当前的context执行
   G(context^exten^pri) ：当呼叫被应答之后，将主叫方跳转到指定的priority中执行，被叫跳转到指定的priority+1中执行，指定的priority由G的参数指定。
   h ：允许被叫方按"*"结束会话
   H ：允许主叫方按"*"结束会话
   i ：忽略任何forwarding请求
   j ：当所有呼叫请求都忙的时候跳转到当前priority+101处
   k ：允许被叫使用parking功能
   K ：允许主叫使用parking功能
   L(x[:y][:z]) ：限定呼叫'x'ms，当剩下'y'ms时播放一个警告，重复这个警告每隔'z'ms。下面这些变量是用于这个操作：
      LIMIT_PLAYAUDIO_CALLER  yes|no (default yes) 对主叫播放语音
      LIMIT_PLAYAUDIO_CALLEE  yes|no   对被叫播放语音
      LIMIT_TIMEOUT_FILE   时间到的时候播放的语音
      LIMIT_CONNECT_FILE   呼叫开始时播放的语音
      LIMIT_WARNING_FILE   y定义的那个警告的语音，一般都是播放还剩多少时间
   m([class]) ：为主叫提供hold music在Channel应答之前。
   M(x[^arg]) ：为被叫Channel执行指定的宏，在还未和主叫桥接之前。被指定的参数可以用"^"来分隔。宏执行完后后会返回一个变量MACRO_RESULT来指示接下来要执行的命令：
      ABORT  通话两端都挂断
      CONGESTION 当线路催挂的时候执行，也就是设置完CONGESTION状态，然后继续执行流程
      BUSY  当线路忙的时候执行，如果j这个参数被设置则，跳转到priority+101处执行
      CONTINUE 挂断被叫，主叫继续执行流程
      GOTO:<context>^<exten>^<priority>  跳转到指定的流程处继续执行
     注意：TIMEOUT()函数不能用在宏中
   n([x])和N ：修改screen/privacy模式. ；screen/privacy就是在被叫应答后还没有桥接之前给被叫播放一段IVR来让它做一些操作，其中就有选择是否愿意接受这个呼叫
   p和P([x]) ：设置screen/privacy模式.
   o ：指定主叫Channel的callerID为被叫Channel的CallerID。
   O([x]) ：设置Operator Services模式，只对zaptel和dahdi通道有效
   r ：主叫等待应答是为主叫播放回铃音
   S(x) ：应答后x秒挂断通话
   t ：允许被叫发送DTMF实现transfer主叫 详细信息在features.conf中设置
   T ：允许主叫发送DTMF实现transfer被叫 详细信息在features.conf中设置
   w ：允许被叫发送DTMF为通话录音  详细信息在features.conf中设置
   W ：允许主叫发送DTMF为通话录音  详细信息在features.conf中设置
  第四个参数是一个url地址，如果通道支持这个url将发送给被叫
  通道变量：
  DIALEDTIME    这个变量是指从呼叫开始到会话结束的时间
  ANSWEREDTIME  这个变量是指应答开始到会话结束的时间
  DIALSTATUS   这个变量显示的是呼叫的结果状态有以下一些值CHANUNAVAIL | CONGESTION | NOANSWER | BUSY | ANSWER | CANCEL | DONTCALL | TORTURE | INVALIDARGS ，在Privacy和Screening模式中，被叫选择发送主叫到'Go Away'脚本时状态变为DONTCALL；发送到'torture'脚本时状态变为TORTURE。
```

####   5)、Hangup

  语法：

```shell
Hangup([causecode])
```

  这个命令是挂断一个Channel。可选的参数是指定挂机的原因。

####   6)、Goto

  语法：

```shell
Goto([[context|]extension|]priority)
```

  这个命令是跳转到指定priority处执行，默认是当前的context和extension。当输入的context直接挂机，当输入的extension和priority错误会在当前context寻找i流程来执行，如果i不存在就寻找h流程来执行，如果连h也不存在，就挂机。当没有任何参数的时候该通道也会挂机。

####   7)、GotoIf

  语法：

```shell
GotoIf(condition?[labeliftrue]:[labeliffalse])
```

  这个命令是有条件的跳转，跟c语言中的？：语句有点类似，当condition的条件为1时跳转到labeliftrue处，否则跳转到labeliffalse处。label的格式为[[context|]extension|]priority。其他就跟Goto命令一样。
  就先介绍这几个常用的，别的一些会在下一篇中介绍。

## 三、Dialplan中的变量和逻辑表达

###   a、Dialplan中的变量通过${变量名}来引用和标识，变量名不一定要大些，但是大些可有助于阅读。

####   1)、全局变量

   全局变量适用于所有的Context所有的extension。全局变量可以在[globals]段中定义，也可以通过SetGlobalVar()来定义。下面举个例子。

```shell
	 [globals]
   OUTLINE=Zap/g1
   或
   [from-ext]
   exten => _9XXXXXXX.,1,SetGlobalVar(OUTLINE=Zap/g1)
```



####   2)、通道变量

  通道变量是特定的呼叫相关的变量，与全局变量不同，通道变量只能在当前呼叫存在期间定义，并只能用于参与该呼叫的通道。有很多预先定义的通道变来那个可以用于Dialplan，在asterisk源程序中的doc子目录下channelvariables.txt文件中有详细的介绍。通道变量可以通过Set()来设置。例如：

```shell
 exten => _123409XX,1,Set(INCOMING=${EXTEN})
```

  EXTEN就是一个asterisk已经预先定义的变量，他表示的就是当前的extension。既然提到了EXTEN变量在这里顺便就说一下变量的截取，语法是这样的：

```shell
{variable_name[:offset[:length]]}。举个例子俩说明一下：
  exten => 12340900,1,Set(INCOMING=${EXTEN})
  这个里面${EXTEN}就是12340900，如果我只想要后四位那怎么办那？${EXTEN:4}就表示0900了，也就是去掉了${EXTEN}的前四位；那要是想要前四位怎么办那？很简单${EXTEN:0:4}就表示1234了。offset和length也可以是负数，offset是负数表示要的是从后面数|offset|个数，length是负数表示要的是除了从后面数的|length|个数。哈哈，说的不太明白举个例子吧。${EXTEN:-4}也表示0900，${EXTEN:-2}就表示00，${EXTEN:2:4}表示3409，${EXTEN:2:-2}也表示3409，${EXTEN:-6:-2}也表示3409.
```

3)、环境变量

   环境变量是一种在asterisk里面访问操作系统环境变量的一种方法。这些变量以\${ENV(var)}的形式引用，其中var就是要引用的操作系统环境变量。

###   b、逻辑表达式.

你应该没有忘记\${var}表示变量，逻辑表达式和这个差不多\$[expression].\$[1 + 2]就表示1+2的结果3.中括号中运算符和变量之间最好用空格分开，否则可能会出现错误的结果。当然中括号里面可以是任意运算符。这些预算符包括：

####   1)、逻辑运算符

```
 expr1 | expr2   当expr1为真，赋值为expr1的值，否则为expr2的值
  expr1 & expr2  当expr1和expr2的值都为真时，赋值为expr1的值，否则赋值为0
  expr1 {=, >, >=, <, <=, !=} expr2  如果自变量都是整数，将得到一个整数的比较结果；否则他们将得到字符串的结果，如果给定的关系是正确的，这个结果是1，否则就是0.
  ! expr1   取反，当expr1是NULL、0、一个空字符串、或者一个字符串"0"，返回1；否则返回0.
```



####   2)、数学运算符

```
  expr1 {+, -, *, /, %} expr2  加减乘除余数
  \- expr1       取负
```



####   3)、正则表达式运算符

  expr1 : expr2  这个运算符expr2匹配到expr1，expr2必须是个表达式，如果匹配成功，被匹配的表达式包括了至少一个正则表达式的字表达式，整个表达式对应返回(1 or \1);另外，匹配操作符返回字符匹配上的数量。如果匹配失败，返回空值。其他情况返回0.
  expr1 =~ expr2  这个运算符和：很像，唯一区别就是这个可以不从字符串的开始匹配。

####   4)、三态运算符

  expr1 ? expr2 : expr3  这个用法就不用说了，说说expr1的否的条件就行了，数字的话是0，字符串时""(空串)。

###  c、运算符优先级

```
  1)、(,)
  2)、!,-
  3)、:,=~
  4)、*,/,%
  5)、+,-
  6)、=,!=,<,>,<=,>=
  7)、|,&
  8)、?:
```



## 四、拨号方案函数

 拨号方案函数可以增加更多功能到你的表达式中，你可以想象他们运行起来和操作符类似，但是更高级一些。
 语法：FUNCTION_NAME(argument)
 非常像变量，你可以引用函数名，但是你如果要引用函数的值，就要用美元"\$"放在前面，用花括号"{}"括起函数表达式。就像这样：

```shell
${FUNCTION_NAME(argument)}
```

 函数也可以嵌套封装其他的函数，如下：

```  shell
  ${FUNCTION_NAME(${FUNCTION_NAME(argument)})}
举个例子吧：
  exten => 123,1,Set(TEST=example)
  exten => 123,2,SayNumber(${LEN(${TEST})})



  上面这个例子能算出字符串example有7个字符，将字符串的数量赋值给变量长度，然后将数量送给SayNumber().
```

## 五、Include
 Asterisk允许在一个context中使用另一个context，通过include指令来实现。这用来授予访问权给不同的拨号方案段。介绍一下语法：
 语法：include => context
 在当前context包含另外的context时，必须注意包含顺序。Asterisk首先试图在当前context中匹配extension。如果不成功，会试图尝试第一个包含进来的context，然后按照包含顺序再去尝试其他的context。就跟把包含的contex拷贝到当前context的后面有点类似。另外还有一种include的应用就是一个dialplan文件包含另一个dialplan文件，意思和上面说的include用法差不多，只不过这回是文件。
 语法：#include FILENAME
 这个在elastrix中应用的比较多，差不多每个关键的配置文件都包含另一个附加的配置文件。举个在elastrix里面的例子，在extensions.conf
 文件中你会发现这样的语句：

```shell
 #include extensions_additional.conf
 #include extensions_custom.conf
```

## 六、宏macro

 你如果熟悉计算机编程，那么你对宏就肯定不会陌生，dialplan也支持。你可以定义一个宏指令，它包含多步的指令列表，然后让电话extension指向这个宏指令。

```
语法：[macro-MACRONAME]
   exten => s,1,action
   exten => s,n,action
   exten => s,n,action
 调用语法：exten => Macro(macroname,arg1,arg2...)
```

 宏指令的名字必须以macro-作为开始。这是他们与常规的context的区别。Dialplan中的宏指令的命令同其他任何命令很相似。唯一的限制因素是宏指令只能用"s"extension。我们先定义一个宏指令：

```
 [macro-dial]
 exten => s,1,Dial(SIP/101)
 exten => s,n,VoiceMail(101)
 接下来我们看看怎样调用宏指令：
  exten => _123409XX,1,GotoIf(\$[\${EXTEN}=12340910]?4:2) 
  exten => _123409XX,2,Macro(dial)
  exten => _123409XX,3,Goto(5)
  exten => _123409XX,4,BackGround(MyVox/Greeting)
  exten => _123409XX,5,Hangup
```

 此外Macro()程序也定义了几种特别的变量来为我们使用。它们包括:

```shell
 ${MACRO_CONTEXT}    这个被调用宏中，初始的context
  ${MACRO_EXTEN}   这个被调用宏中，初始的extension
  ${MACRO_PRIORITY}  这个被调用宏中，初始的priority
  ${MACRO_OFFSET}   宏返回后从${MACRO_OFFSET}+n+1的priority处执行
  ${ARGn}     传递到宏指令的第n个变量。例如第一个自变量是${ARG1},第二个是${ARG2}
```

为了举个使用变量的例子，我们把上面例子的宏修改一下：

```
  [macro-dial]
  exten => s,1,Dial(SIP/1${MACRO_EXTEN:6})
  exten => s,n,VoiceMail(1${MACRO_EXTEN:6})
```

 这样当拨的是12340901时宏就执行Dial(SIP/101),并给并留言到101语音信箱里面。

## 七、结束语
  终于结束了(有遗漏的以后再加吧)。其实这篇文章最初想法是写给杨兄的，后来想想既然写了为什么不写成个类似文章样式的，放到自己的空间，给一些希望了解asterisk的人一些帮助。
  中文说asterisk的数很少，据我所和好像只有《asterisk未来电话之路》这本书汉化了，要想让技术发展就得有自由和宽松的风气，就像春秋战国时一样大家勇于发表自己的见解、勇于讨论。我知道国内有很多asterisk的高手,我搞asterisk也才只有两年，只能算个刚入门的新生，所以我希望能有更多的人，能把自己学到的对于asterisk技术见解和技术思路，都写出来大家分享。
   哪怕就像我一样只是汇集翻译个文档。最后希望只要能帮助到大家就好，哈哈，记住转载一定要保留我的名字和出处，谢谢了。(附上作者连接在下方)





原文档连接：<https://www.cnblogs.com/einyboy/archive/2012/10/17/2727792.html>

