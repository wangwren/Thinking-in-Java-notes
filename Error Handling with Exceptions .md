# 通过异常处理错误

## 基本异常

**异常情形**是阻止当前方法或作用域继续执行的问题。把异常情形与普通问题相区分很重要。所谓普通问题是指，在当前环境下能得到足够的信息，总能处理这个错误。而对于异常情形，就不能继续下去了，因为在当前环境下无法获得必要的信息来解决问题。你所能做的就是从当前环境跳出，并且把问题提交给上一级环境。这就是抛出异常时所发生的事情。

当抛出异常后，有几件事会随之发生。首先，同Java中其他对象的创建一样，将使用 new 在堆上创建异常对象。然后，当前的执行路径被终止，并且从当前环境中弹出对异常对象的引用。此时，异常处理机制接管程序，并开始寻找一个恰当的地方来继续执行程序。这个恰当的地方就是**异常处理程序**，它的任务是将程序从错误状态中恢复，以使程序要么换一种方式运行，要么继续运行下去。

举一个抛出异常的简单例子。对于对象引用 t ，传给你的时候可能尚未被初始化。所以在使用这个对象引用调用其方法之前，会先对引用进行检查。可以创建一个代表错误信息的对象，并且将它从当前环境中“抛出”，这样就把错误信息传播到了“更大”的环境中。这被称为**抛出一个异常**，看起来像这样：

```java
if(t == null)
    throw new NullPointerException();
```

这就抛出了异常，于是在当前环境下就不必再为这个问题操心了。

### 异常参数

与使用Java中的其他参数对象一样，我们总是用 new 在堆上创建异常对象，这也伴随着存储空间的分配和构造器的调用。所有标准异常类都有两个构造器：一个是默认构造器；另一个是接受字符串作为参数，以便能把相关信息放入异常对象的构造器：

```java
throw new NullPointException("t = null");
```

要把这个字符串的内容取出来可以有多重不同的方法。

关键字 throw 将产生许多有趣的结果。在使用 new 创建了异常对象之后，此对象的引用将传给throw。尽管返回的异常对象其类型通常与方法设计的返回类型不同，但从效果上看，它就像是从方法“返回”的。可以简单地把异常处理看成一种不同的返回机制，当然若过分强调这种类比的话，就会有麻烦了。**另外还能用抛出异常的方式从当前的作用域退出**。在这两种情况下，将会返回一个异常对象，然后退出方法或作用域。

此外，**能够抛出任意类型的 Throwable 对象，它是异常类型的根类。通常，对于不同类型的错误，要抛出相应的异常**。错误信息可以保存在异常对象内部或者用异常类的名称来暗示，上一层环境通过这些信息来决定如何处理异常。（通常，异常对象中仅有的信息就是异常类型，除此之外不包含任何有意义的内容。）

## 捕获异常
要明白异常是如何被捕获的，必须首先理解**监控区域**(guarded region)的概念。它是一段可能产生异常的代码，并且后面跟着处理这些异常的代码。

### try块
如果在方法内部抛出了异常（或者在方法内部调用的其他方法抛出了异常），这个方法将在抛出异常的过程中结束。要是不希望方法就此结束，可以在方法内设置一个特殊的块来捕获异常。因为在这个块里“尝试”各种（可能产生异常的）方法调用，所以称为 try 块。它是跟在try关键字之后的普通程序块：
```java
try{
    // Code that might generate exceptions
}
```
### 异常处理程序
当然，抛出的异常必须在某处得到处理。这个“地点”就是**异常处理程序**，而且针对每个要捕获的异常，得准备相应的处理程序。异常处理紧跟在 try 块之后。以关键字 catch 表示：

```java
try{
    //Code that might generate exception
}catch(Type1 id1){
    //Handle exceptions of Type1
}catch(Type2 id2){
    //Handle exceptions of Type2
}catch(Type3 id3){
    //Handle exceptions of Type3
}
// etc...
```

每个 catch 子句（异常处理程序）看起来就像是接收一个且仅接收一个特殊类型的参数的方法。可以在处理程序的内部使用标识符，因为异常的类型已经给了你足够的信息来对异常进行处理，但标识符并不可以省略。

异常处理程序必须紧跟在 try 块之后。当异常被抛出时，异常处理机制将负责搜寻参数与异常类型相匹配的第一个处理程序。然后进入 catch 子句执行。此时认为异常得到了处理。一旦 catch 子句结束，则处理程序的查找过程结束。注意，只有匹配的catch子句才能得到执行；这与 switch 语句不同，switch 语句需要在每一个 case 后面跟一个break，以避免执行后续的case子句。

注意在 try 块内部，许多不同的方法调用可能会产生类型相同的异常，而你只需要提供一个针对此类型的异常处理程序。
#### 终止与恢复

异常处理理论上有两种基本模型。Java支持**终止模型**。在这种模型中，将假设错误非常关键，以至于程序无法返回到异常发生的地方继续执行。一旦异常被抛出，就表明已无法挽回，也不能回来继续执行。

另一种称为**恢复模型**。意思是处理程序的工作是修正错误，然后重新尝试调用出问题的方法，并认为第二次能成功。对于恢复模型，通常希望异常被处理之后能继续执行程序。如果想要用Java实现类似恢复的行为，那么在遇见错误时就不能抛出异常，而是调用方法来修正该错误。或者，把 try 块放在 while 循环里，这样就不断地进行 try 块，直到得到满意的结果。

长久以来，尽管程序员们使用的操作系统支持恢复模型的异常处理，但他们最终还是转向使用类似“终止模型”的代码，并且忽略恢复行为。所以虽然恢复模型开始显得很吸引人，但不是很实用。其中的主要原因可能是它导致的耦合：**恢复性的处理程序需要了解异常抛出的地点，这势必要包含依赖于抛出位置的非通用性代码。这增加了代码编写和维护的困难，对于异常可能会从许多地方抛出的大型程序来说，更是如此**。

## 创建自定义异常

要自己定义异常类，必须从已有的异常类继承，最好是选择意思相近的异常类继承（不过这样的异常并不容易找）。建立新的异常类型最简单的方式就是**让编译器为你产生默认构造器**，所以这几乎不用写多少代码。

```java
package exceptions;

class SimpleException extends Exception{}

public class InherintingExceptions {
	public void f() throws SimpleException {
		System.out.println("Throw SimpleException from f()");
		throw new SimpleException();
	}
	public static void main(String[] args) {
		InherintingExceptions sed = new InherintingExceptions();
		try {
			sed.f();
		}catch (SimpleException e) {
			System.out.println("Caught it!");
		}
	}
}
/**
Throw SimpleException from f()
Caught it!
*/
```

**编译器创建了默认构造器，它将自动调用基类的默认构造器。**本例中不会得到像`SimpleException(String)`这样的构造器，这种构造器也不实用。你将看到，对异常来说，**最重要的部分就是类名**，所以本例中建立的异常类在大多数情况下已经够用了。

通过写入`System.err`而将错误发送给**标准错误流**。通常这比把错误信息输出到`System.out`要好，因为`System.out`也许会被重定向。如果把结果送到`System.err`，它就不会随`System.out`，一起被重定向，这样更容易被用户注意。

也可以为异常类定义一个接受字符串参数的构造器：

```java
package exceptions;

class MyException extends Exception{
	public MyException() {
		
	}
	
	public MyException(String msg) {
		super(msg);
	}
}

public class FullConstructors {
	public static void f() throws MyException {
		System.out.println("Throw MyException from f()");
		throw new MyException();
	}
	public static void g() throws MyException {
		System.out.println("Throw MyException from g()");
		throw new MyException("Originated in g()");
	}
	public static void main(String[] args) {
		try {
			f();
		}catch (MyException e) {
			e.printStackTrace(System.out);
		}
		
		try {
			g();
		}catch (MyException e) {
			e.printStackTrace(System.out);
		}
	}
}
/**
Throw MyException from f()
exceptions.MyException
	at exceptions.FullConstructors.f(FullConstructors.java:16)
	at exceptions.FullConstructors.main(FullConstructors.java:24)
Throw MyException from g()
exceptions.MyException: Originated in g()
	at exceptions.FullConstructors.g(FullConstructors.java:20)
	at exceptions.FullConstructors.main(FullConstructors.java:30)
*/
```

新增的代码不长：两个构造器定义了MyException类型对象的创建方式。对于第二个构造器，使用super关键字明确调用了其基类构造器，它接受一个字符串作为参数。

在异常处理程序中，调用了在Throwable类声明(Exception即从此类继承)的`printStackTrace()`方法。就像从输出中看到的，它将打印“从方法调用处”的方法调用序列。这里，信息被发送到了System.out，并自动地被捕获和显示在输出中。但是，如果调用默认版本：`e.printStackTrace();`则信息将被输出到标准错误流。

### 异常与记录日志

```java
package exceptions;

import java.io.PrintWriter;
import java.io.StringWriter;
import java.util.logging.Logger;

class LoggingException extends Exception{
	private static Logger logger = Logger.getLogger("LoggingException");
	public LoggingException() {
		StringWriter trace = new StringWriter();
		printStackTrace(new PrintWriter(trace));
		logger.severe(trace.toString());
	}
}

public class LoggingExceptions {

	public static void main(String[] args) {
		try {
			throw new LoggingException();
		}catch (LoggingException e) {
			System.err.println("Caught" + e);
		}
		try {
			throw new LoggingException();
		}catch (LoggingException e) {
			System.err.println("Caught" + e);
		}
	}
}
/**
九月 18, 2018 11:44:51 下午 exceptions.LoggingException <init>
严重: exceptions.LoggingException
	at exceptions.LoggingExceptions.main(LoggingExceptions.java:20)

Caughtexceptions.LoggingException
九月 18, 2018 11:44:51 下午 exceptions.LoggingException <init>
严重: exceptions.LoggingException
	at exceptions.LoggingExceptions.main(LoggingExceptions.java:25)

Caughtexceptions.LoggingException
*/
```

静态的`Logger.getLogger()`方法创建了一个String参数相关联的 Logger 对象（通常与错误相关的包名或类名），这个Logger对象会将其输出发送到`System.err`。向Logger写入的最简单方式就是直接调用与日志记录消息的级别相关联的方法，这里使用的是`server()`。为了产生日志记录消息，我们想要获取异常抛出处的栈轨迹，但是`printStackTrace()`不会默认地产生字符串。为了获取字符串，我们需要使用重载的`printStackTrace()`方法，它接受一个`java.io.PrintWriter`对象作为参数。

尽管由于LoggerException将所有记录日志的基础设施都构建在异常自身中，使得它所使用的方式非常方便，并因此不需要客户端程序员的干预就可以自动运行，但是更常见的情形是我们需要捕获和记录其他人编写的异常，因此我们必须在异常处理程序中生成日志消息：

```java
package exceptions;

import java.io.PrintWriter;
import java.io.StringWriter;
import java.util.logging.Logger;

public class LoggingException2 {

	private static Logger logger = Logger.getLogger("LoggingException2");
	static void logException(Exception e) {
		StringWriter trace = new StringWriter();
		e.printStackTrace(new PrintWriter(trace));
		logger.severe(trace.toString());
	}
	public static void main(String[] args) {
		try {
			throw new NullPointerException();
		}catch (NullPointerException e) {
			logException(e);
		}
	}
}
/**
九月 19, 2018 12:01:33 上午 exceptions.LoggingException2 logException
严重: java.lang.NullPointerException
	at exceptions.LoggingException2.main(LoggingException2.java:17)
*/
```

还可以更进一步自定义异常，比如加入额外的构造器和成员：

```java
package exceptions;


class MyException2 extends Exception{
	private int x;
	public MyException2() {
		
	}
	public MyException2(String msg) {
		super(msg);
	}
	public MyException2(String msg,int x) {
		super(msg);
		this.x = x;
	}
	
	public int val() {
		return x;
	}
	
	/**
	 * 重写父类Throwable的方法
	 */
	@Override
	public String getMessage() {
		return "Detail Message : " + x + " " + super.getMessage();
	}
	
}

public class ExtraFeatures {

	public static void f() throws MyException2 {
		System.out.println("Throwing MyException2 from f()");
		throw new MyException2();
	}
	
	public static void g() throws MyException2 {
		System.out.println("Throwing MyException2 from g()");
		throw new MyException2("Originated in g()");
	}
	
	public static void h() throws MyException2 {
		System.out.println("Throwing MyException2 from h()");
		throw new MyException2("Originated in g()", 47);
	}
	
	public static void main(String[] args) {
		try {
			f();
		}catch (MyException2 e) {
			e.printStackTrace(System.out);
		}
		try {
			g();
		}catch (MyException2 e) {
			e.printStackTrace(System.out);
		}
		try {
			h();
		}catch (MyException2 e) {
			e.printStackTrace(System.out);
			System.out.println("e.val() = " + e.val());
		}
	}
}
```

新的异常添加了字段 x 以及设定 x 值的构造器和读取数据的方法。此外，还覆盖了`Throwable.getMessgae()`方法，以产生更详细的信息。对于 异常类来说，getMessage()方法有点类似于toString()方法。

既然异常也是对象的一种，所以可以继续修改这个异常类，以得到更强的功能。但要记住，使用程序包的客户端程序员可能仅仅只是查看一下抛出的异常类型，其他的就不管了（大多数Java库里的异常都是这么用的），所以对异常所添加的其他功能也许根本用不上。

## 异常说明

Java鼓励人们把方法可能会抛出的异常告知使用此方法的客户端程序员。这是种优雅的做法，它使得调用者能确切知道写什么样的代码可以捕获所有潜在的异常。当然，如果提供了源代码，客户端程序员可以在源代码中查找 throw 语句来获知相关信息，然而程序库通常并不与源代码一起发布。为了预防这样的问题，Java提供了相应的语法（并强制使用这个语法），使你能以礼貌的方式告知客户端程序员某个方法可能抛出的异常类型，然后客户端程序员就可以进行相应的处理。这就是**异常说明**，它属于方法声明的一部分，紧跟在形式参数列表之后。

异常说明使用了附加的关键字 throws ，后面接一个所有潜在异常类型的列表，所以方法定义可能看起来像这样：

```java
void f() throws TooBig,TooSmall,DivZero{//...}
```

但是，要是这样写，就表示此方法不会抛出任何异常（除了从RuntimeException继承的异常，它们可以在没有异常说明的情况下被抛出。

```java
void f(){//...}
```

代码必须与异常说明保持一致。如果方法里的代码产生了异常却没有进行处理，编译器会发现这个问题并提醒你：**要么处理这个异常，要么就在异常说明中表明此方法将产生异常**。通过这种自顶向下强制执行的异常说明机制，Java在编译时就可以保证一定水平的异常正确性。

不过还是有个能“作弊”的地方：可以声明方法将抛出异常，实际上却不抛出。编译器相信了这个声明，并强制此方法的用户像真的抛出异常那样使用这个方法。这样做的好处是，为异常先占一个位置，以后就可以抛出这种异常而不修改已有的代码。在定义抽象基类和接口时这种能力很重要，这样派生类或接口实现能够抛出这些预先声明的异常。

这种在编译时被强制检查的异常称为**被检查的异常**。

## 捕获所有异常

可以只写一个异常处理程序来捕获所有类型的异常。通过捕获异常类型的基类**Exception**，就可以做到这一点（事实上还有其他的基类，但Exception是同编程活动相关的基类）：

```java
catch(Exception e){
    System.out.println("Caught an exception");
}
```

这将捕获所有异常，所以最好把它放在处理程序列表的末尾，以防它抢在其他处理程序之前先把异常捕获了。

因为Exception是与编程有关的所有异常类的基类，所以它不会含有太多具体的信息，不过可以调用它从其基类Throwable继承的方法：

```java
String getMessage()
String getLocalizedMessage()
```

用来获取详细信息，或用本地语言表示的详细信息。

```java
String toString()
```

返回对Throwable的简单描述，要是有详细信息的话，也会把它包含在内。

```java
void printStackTrace()
void printStackTrace(PrintStream)
void printStackTrace(java.io.PrintWriter)
```

打印Throwable和Throwable的调用栈轨迹。调用栈显示了“把你带到异常抛出地点”的方法调用序列。其中第一个版本输出到标准错误，后两个版本允许选择要输出的流。

```java
Throwable fillInStackTrace()
```

用于在Throwable对象的内部记录栈帧的当前状态。这在程序重新抛出错误或异常时很有用。

此外，也可以使用Throwable从其基类Object（也是所有类的基类）继承的方法。对于异常来说，getClass()也许是个很好用的方法，它将返回一个表示此对象类型的对象。然后可以使用getName()方法查询这个Class对象包含包信息的名称，或者使用只产生类名称的getSimpleName()方法。

下面的例子演示了如何使用Exception类型的方法：

```java
package exceptions;

public class ExceptionMethods {

	public static void main(String[] args) {
		try {
			throw new Exception("My Exception");
		}catch (Exception e) {
			System.out.println("Caught Exception");
			System.out.println("getMessage():" + e.getMessage());
			System.out.println("getLocalizedMessage():" + e.getLocalizedMessage());
			System.out.println("toString():" + e);
			System.out.println("printStackTrace():");
			e.printStackTrace(System.out);
		}
	}
}
/**
Caught Exception
getMessage():My Exception
getLocalizedMessage():My Exception
toString():java.lang.Exception: My Exception
printStackTrace():
java.lang.Exception: My Exception
	at exceptions.ExceptionMethods.main(ExceptionMethods.java:7)
*/
```

可以发现每个方法都比前一个提供了更多的信息---实际上它们每一个都是前一个的超集。

### 栈轨迹

printStackTrace()方法所提供的信息可以通过getStackTrace()方法来直接访问，这个方法将返回一个由栈轨迹中的元素所构成的数组，其中每一个元素都表示栈中的一帧。元素 0 是栈顶元素，并且是调用序列中的最后一个方法调用（这个Throwable被创建和抛出之处）。数组中的最后一个元素和栈底是调用序列中的第一个方法调用。下面的程序是一个简单的演示示例：

```java
package exceptions;

public class WhoCalled {

	static void f() {
		//Generate an exception to fill in the stack trace
		try {
			throw new Exception();
		}catch (Exception e) {
			for(StackTraceElement ste : e.getStackTrace()) {
				System.out.println(ste.getMethodName());
			}
		}
	}
	static void g() {
		f();
	}
	static void h() {
		g();
	}
	public static void main(String[] args) {
		f();
		System.out.println("-----------------------");
		g();
		System.out.println("-----------------------");
		h();
	}
}
/**
f
main
-----------------------
f
g
main
-----------------------
f
g
h
main
*/
```

### 重新抛出异常

有时希望把刚捕获的异常重新抛出，尤其是在使用Exception捕获所有异常的时候。既然已经得到了对当前异常对象的引用，可以直接把它重新抛出：

```java
catch(Exception e){
    System.out.println("An exception was throw");
    throw e;
}
```

重新抛出异常会把异常抛给上一级环境中的异常处理程序，同一个try块的后续catch子句将被忽略。此外，异常对象的所有信息都得以保持，所以高一级环境中捕获此异常的处理程序可以从这个异常对象中得到所有信息。

如果只是把当前异常对象重新抛出，那么printStackTrace()方法显示的将是原来异常抛出点的调用栈信息，而并非重新抛出点的信息。要想要更新这个信息，可以调用fillStackTrace()方法，这将返回一个Throwable对象，它是通过把当前调用栈信息填入原来那个异常对象而建立的，就像这样：

```java
package exceptions;

public class Rethrowing {

	public static void f() throws Exception {
		System.out.println("originating the exception in f()");
		throw new Exception("throw from f()");
	}
	
	public static void g() throws Exception {
		try {
			f();
		}catch (Exception e) {
			System.out.println("Inside g(),e.printStackTrace()");
			e.printStackTrace(System.out);
			throw e;
		}
	}
	
	public static void h() throws Exception {
		try {
			f();
		}catch (Exception e) {
			System.out.println("Inside h(),e,printStackTrace()");
			e.printStackTrace(System.out);
			throw (Exception)e.fillInStackTrace();	//在这
		}
	}
	
	public static void main(String[] args) {
		try {
			g();
		}catch (Exception e) {
			System.out.println("main:printStackTrace()");
			e.printStackTrace(System.out);
		}
		
		try {
			h();
		}catch (Exception e) {
			System.out.println("main:printStackTrace()");
			e.printStackTrace(System.out);
		}
	}
}
/**
originating the exception in f()
Inside g(),e.printStackTrace()
java.lang.Exception: throw from f()
	at exceptions.Rethrowing.f(Rethrowing.java:7)
	at exceptions.Rethrowing.g(Rethrowing.java:12)
	at exceptions.Rethrowing.main(Rethrowing.java:32)
main:printStackTrace()
java.lang.Exception: throw from f()
	at exceptions.Rethrowing.f(Rethrowing.java:7)
	at exceptions.Rethrowing.g(Rethrowing.java:12)
	at exceptions.Rethrowing.main(Rethrowing.java:32)
originating the exception in f()
Inside h(),e,printStackTrace()
java.lang.Exception: throw from f()
	at exceptions.Rethrowing.f(Rethrowing.java:7)
	at exceptions.Rethrowing.h(Rethrowing.java:22)
	at exceptions.Rethrowing.main(Rethrowing.java:39)
main:printStackTrace()
java.lang.Exception: throw from f()
	at exceptions.Rethrowing.h(Rethrowing.java:26)
	at exceptions.Rethrowing.main(Rethrowing.java:39)
*/
```

调用fillStackTrace()的那一行就成了异常的新发生地了。

永远不必为清理前一个异常对象而担心，或者说为异常对象的清理而担心。它们都是用new在堆上创建的对象，所以垃圾回收器会自动把它们清理掉。

### 异常链
常常会想要在捕获一个异常后抛出另一个异常，并且希望把原始异常的信息保存下来，这被称为**异常链**。在JDK1.4以前，程序员必须自己编写代码来保存原始异常的信息。现在所有Throwable的子类在构造器中都可以接受一个 cause 对象作为参数。这个 cause 就用来表示原始异常，这样通过把原始异常传递给新的异常，使得即使在当前位置创建并抛出了新的异常，也能通过这个异常链追踪到异常最初发生的位置。

有趣的是，在Throwable的子类中，只有三种基本的异常类提供了带 cause 参数的构造器。它们是 Error (用于Java虚拟机报告系统错误)、Exception以及 RuntimeException。如果要把其他类型的异常链接起来，应该使用initCause()方法而不是构造器。

## Java标准异常

Throwable这个Java类被用来表示任何可以作为异常被抛出的类。Throwable对象可分为两种类型（指从Throwable继承得到的类型）：

- Error用来表示编译时和系统错误（除特殊情况外，一般不用关心）；
- Exception是可以被抛出的基本类型，在Java类库，用户方法以及运行时故障中都可能抛出Exception型异常。所以Java程序员关心的基类型通常是Exception。

属于运行时异常的类型有很多，它们会自动被Java虚拟机抛出，所以不必在异常说明中把它们列出来。这些异常都是从RuntimeException类继承而来，所以既体现了继承的优点，使用起来也很方便。这构成了一组具有相同特征和行为的异常类型。并且，也不需要在异常说明中声明方法将抛出RuntimeException类型的异常(或者任何从RuntimeException继承的异常)，它们也被称为**不受检查异常**。**这种异常属于错误，将自动捕获**。

如果不捕获这种类型的异常，因为编译器没有在这个问题上对异常说明进行强制检查，RuntimeException类型的异常也许会穿越所有的执行路径直达main()方法，而不会被捕获。

```java
package exceptions;

public class NeverCaught {

	static void f() {
		throw new RuntimeException("From f()");
	}
	
	static void g() {
		f();
	}
	
	public static void main(String[] args) {
		g();
	}
}
/**
Exception in thread "main" java.lang.RuntimeException: From f()
	at exceptions.NeverCaught.f(NeverCaught.java:6)
	at exceptions.NeverCaught.g(NeverCaught.java:10)
	at exceptions.NeverCaught.main(NeverCaught.java:14)
*/
```

RuntimeException(或任何从继承的异常)是一个特例。对于这种异常类型，编译器不需要异常说明，其输出被报告给了System.err。

所以答案是：如果RuntimeException没有被捕获而直达main()，那么在程序退出前将调用异常printStackTrace()方法、

请务必记住：只能在代码中忽略RuntimeException（及其子类）类型的异常，其他类型异常的处理都是由编译器强制实施的。究其原因，RuntimeException代表的是编程错误：

- 无法预料的错误。比如从你控制范围之外传递进来的null引用。
- 作为程序员，应该在代码中进行检查的错误。（比如对于ArrayIndexOutOfBoundsException，就得注意一下数组的大小了。）在一个地方发生的异常，常常会在另一个地方导致错误。

## 使用finally进行清理

对于一些代码，可能会希望无论 try 块中的异常是否抛出，它们都能得到执行。这通常适用于内存回收之外的情况（因为回收由垃圾回收器完成）。为了达到这个效果，可以在异常处理程序后面加上 finally 子句。完整的异常处理程序看起来像这样：

```java
try{
    //The guarded region:Dangerous activities
    //that might throw A,B,or C
}catch(A a1){
    //Handler for situation A
}catch(B b1){
    //Handler for situation B
}catch(C c1){
    //Handler for situation C
}finally{
    //Activities that happen every time
}
```

为了证明finally子句总能运行，可以试试下面这个程序：

```java
package exceptions;

class ThreeException extends Exception{}

public class FianllyWorks {

	static int count = 0;
	public static void main(String[] args) {
		while(true) {
			try {
				if(count++ == 0) {
					throw new ThreeException();
				}
			}catch (ThreeException e) {
				System.out.println("ThreeException");
			}finally {
				System.out.println("In finally clause");
				if(count == 2) {
					break;
				}
			}
		}
	}
}
/**
ThreeException
In finally clause
In finally clause
*/
```

可以从输出中发现，无论异常是否被抛出，finally子句总能被执行。

这个程序也给了我们一些思路，当Java中的异常不允许我们呢回到异常抛出的地点时，那么该如何应对呢？如果把try块放在循环里，就建立了一个“程序继续执行之前必须要达到”的条件。还可以加入一个static类型的计数器或者别的装置，使循环在放弃以前能尝试一定的次数。这将使程序的健壮性更上一个台阶。

### finally用来做什么

对于没有垃圾回收和析构函数自动调用的语言来说，finally非常重要。它能使程序员保证：无论try块里发生了什么，内存总能得到释放。但Java有垃圾回收机制，所以内存释放不再是问题。而且，Java也没有析构函数可供调用。

当要把除内存之外的资源恢复到它们的初始状态时，就要用到finally子句。这种需要清理的资源包括：已经打开的文件或网络连接，在屏幕上画的图形，甚至可以是外部世界的某个开关。

甚至在异常没有被当前的异常处理程序捕获的情况下，异常处理机制也会在跳到更高一层的异常处理程序之前，执行finally子句。

```java
package exceptions;

class FourException extends Exception{}

public class AlwaysFinally {

	public static void main(String[] args) {
		System.out.println("Entering first try block");
		try {
			System.out.println("Entering second try block");
			try {
				throw new FourException();
			}finally {
				System.out.println("finally in 2nd try block");
			}
		}catch(FourException e) {
			System.out.println("Caught FourException in 1st try block");
		}finally {
			System.out.println("finally in 1st try block");
		}
	}
}
/**
ThreeException
In finally clause
In finally clause
*/
```

当涉及break和continue语句的时候，finally子句也会得到执行。请注意，如果把finally子句和带标签的break及continue配合使用，在Java里就没必要使用goto语句中了。

### 在return中使用finally

因为finally子句总是会执行的，所以在一个方法中，可以从多个点返回，并且可以保证重要的清理工作仍旧会执行：

```java
package exceptions;
public class MultipleReturns {

	public static void f(int i) {
		System.out.println("Initialization that requires cleanup");
		try {
			System.out.println("Point 1");
			if(i == 1) {
				return;
			}
			System.out.println("Point 2");
			if(i == 2) {
				return;
			}
			System.out.println("Point 3");
			if(i == 3) {
				return;
			}
			System.out.println("End");
			return;
		}finally {
			System.out.println("Performing cleanup");
		}
	}
	public static void main(String[] args) {
		for(int i = 1;i <= 4;i++) {
			f(i);
		}
	}
}
/**
Initialization that requires cleanup
Point 1
Performing cleanup
Initialization that requires cleanup
Point 1
Point 2
Performing cleanup
Initialization that requires cleanup
Point 1
Point 2
Point 3
Performing cleanup
Initialization that requires cleanup
Point 1
Point 2
Point 3
End
Performing cleanup
*/
```

从输出中可以看出，在finally类内部，从何处返回无关紧要。

### 遗憾：异常缺失

遗憾的是，Java的异常实现也有瑕疵。异常作为程序出错的标志，决不应该被忽略，但它还是有可能被轻易地忽略。用某些特殊的方式使用finally子句，就会发生这种情况：

```java
package exceptions;

class VeryImportantException extends Exception{
	public String toString() {
		return "A very important exception";
	}
}

class HoHumException extends Exception{
	public String toString() {
		return "A trivial exception";
	}
}

public class LostMessage {
	
	void f() throws VeryImportantException {
		throw new VeryImportantException();
	}
	
	void dispose() throws HoHumException {
		throw new HoHumException();
	}
	public static void main(String[] args) {
		try {
			LostMessage lm = new LostMessage();
			try {
				lm.f();
			}finally {
				lm.dispose();
			}
		}catch (Exception e) {
			System.out.println(e);
		}
	}
}
/**
A trivial exception
*/
```

从输出中可以看到，VeryImportantException不见了，它被 finally 子句里的 HollumException所取代。这是相当严重的缺陷，因为异常可能会以一种比前面例子所示更微妙和难以察觉的方式完全丢失。

## 异常的限制

当覆盖方法的时候，**只能抛出在基类方法的异常说明里列出的那些异常**。这个限制很有用，因为这意味着，当基类使用的代码应用到其派生类对象的时候，一样能够工作（当然，这是面向对象的基本概念），异常也不例外。

下面例子演示了这种（在编译时）施加在异常上面的限制：

```java
package exceptions;

class BaseballException extends Exception{}
class Foul extends BaseballException{}
class Strike extends BaseballException{}

abstract class Inning{
	public Inning() throws BaseballException{}
	public void event() throws BaseballException {
		//Dosen't actually have to throw anything
	}
	public abstract void atBat() throws Strike,Foul;
	public void walk(){}
}

class StormException extends Exception{}
class RainedOut extends StormException{}
class PopFoul extends Foul{}

interface Storm{
	public void event() throws RainedOut;
	public void rainHard() throws RainedOut;
}

public class StormyInning extends Inning implements Storm{

	public StormyInning() throws RainedOut,BaseballException {
		
	}
	
	public StormyInning(String s) throws Foul,BaseballException{
		
	}
	
	//实现的接口中有该方法，抽象类中也有该方法，可以选择不抛出异常
	public void event() {}
	
	
	@Override
	public void rainHard() throws RainedOut {}

	@Override
	public void atBat() throws PopFoul{}
	
	public static void main(String[] args) {
		try {
			StormyInning si = new StormyInning();
			si.atBat();
		}catch (PopFoul e) {
			System.out.println("Pop foul");
		}catch (RainedOut e) {
			System.out.println("Rained out");
		}catch (BaseballException e) {
			System.out.println("Generic baseball exception");
		}
		
		try {
			Inning i = new StormyInning();
			i.atBat();
		}catch (Strike e) {
			System.out.println("Strike");
		}catch (Foul e) {
			System.out.println("Foul");
		}catch (RainedOut e) {
			System.out.println("Rained out");
		}catch (BaseballException e) {
			System.out.println("Generic baseball exception");
		}
	}
}
```

在 Inning 类中，可以看到构造器和 event()方法都声明将抛出异常，但实际上没有抛出。这种方式使你能强制用户去捕获可能在覆盖后的event()版本中增加的异常，所以它很合理，这对于抽象方法同样成立，比如atBat()。

接口 Storm 值得注意，因为它包含了一个在 Inning 中定义的方法 event() 和一个不再 Inning中定义的方法 rainHard()。这两个方法都抛出新的异常RainedOut。如果StormyInning类在扩展 Inning类的同时又实现了 Storm接口，那么 Storm 里的 event() 方法就不能改变在 Inning中的event()方法的异常接口。否则的话，在使用基类的时候就不能判断是否捕获了正确的异常，所以这也很合理。当然，如果接口里定义的方法不是来自于基类，比如rainHard()，那么此方法抛出什么样的异常都没有问题。

异常限制对构造器不起作用。你会发现StormyInning的构造器可以抛出任何异常，而不必理会基类构造器所抛出的异常。然而，因为基类构造器必须以这样或那样的方式被调用（这里默认构造器将自动被调用），派生类构造器的异常说明必须包含基类构造器的异常说明。

**子类不能抛出比父类更多的异常**

尽管在继承过程中，编译器会对异常说明做强制要求，但异常说明本身并不属于方法类型的一部分，方法类型是由方法的名字与参数的类型组成的。因此，不能基于异常说明来重载方法。此外，一个出现在基类方法的异常说明中的异常，不一定会出现在派生类方法的异常说明里。这点同继承的规则明显不同，在继承中，基类的方法必须出现在派生类里，换句话说，在继承和覆盖的过程中，**某个特定方法的“异常说明接口”不是变大了而是变小了---这恰好和类接口在继承时的情形相反**。

## 构造器

有一点很重要，即“如果异常发生了，所有东西能被正确的清理吗？”尽管大多数情况下是非常安全的，但涉及构造器时，问题就出现了。构造器会把对象设置成安全的初始状态，但还会有别的动作，比如打开一个文件，这样的动作只有在对象使用完毕并且用户调用了特殊的清理方法之后才能得以清理。如果在构造器内抛出了异常，这些清理行为也许就不能正常工作了。这意味着在编写构造器时要格外细心。

读者也许会认为使用 finally 就可以解决问题。但问题并非如此简单，因为 finally 会每次都执行清理代码。如果构造器在其执行过程中半途而废，也许该对象的某些部分还没有被成功创建，而这些部分在finally子句中却要被清理的。

在下面的例子中，建立了一个 InputFile 类，它能打开一个文件并且每次读取其中的一行。这里使用了 Java标准输入/输出库中的 FileReader和 BufferedReader类。

```java
package exceptions;

import java.io.BufferedReader;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;

public class InputFile {

	private BufferedReader in;
	public InputFile(String fname) throws Exception {
		try {
			in = new BufferedReader(new FileReader(fname));
		}catch(FileNotFoundException e){
			System.out.println("Could not open " + fname);
			throw e;
		}catch (Exception e) {
			//All other exceptions must close it
			try {
				in.close();
			}catch (IOException e2) {
				System.out.println("in.close() unsuccessful");
			}
			throw e;
		}finally {
			//Don't close it here
		}
	}
	public String getLine() {
		String s;
		try {
			s = in.readLine();
		}catch (IOException e) {
			throw new RuntimeException("readLine() failed");
		}
		return s;
	}
	public void dispose() {
		try {
			in.close();
			System.out.println("dispose() successful");
		}catch (IOException e) {
			throw new RuntimeException("in.close() failed");
		}
	}
}
```

InputFile的构造器接受字符串作为参数，该字符串表示所要打开的文件名。在 try 块中，会使用此文件名建立了 FileReader对象。FileReader对象本身用处并不大，但可以用它来建立BufferedReader对象。注意，使用InputFile的好处就能把两步操作合成一步。

如果FileReader的构造器失败了，将抛出FileNotFoundException异常。对于这个异常，并不需要关闭文件，因为这个文件还没有被打开。而任何其他捕获异常的catch子句必须关闭文件，因为在他们捕获到异常之时，文件已经打开了。close()方法也可能会抛出异常，所以尽管它已经在另一个catch子句块里了，还是要再用一层try-catch----对Java编译器而言，这只不过是又多了一对花括号。在本地做完处理之后，异常被重新抛出，对于构造器而言这么做是很合适的，因为你总不希望去误导调用方，让他认为“这个对象已经创建完毕，可以使用了”。

在本例中，由于finally会在每次完成构造器之后都执行一遍，因此它实在不该是调用close()关闭文件的地方。我们希望文件在InputFile对象的整个生命周期内部处于打开状态。

对于在构造阶段可能会抛出异常，并且要求清理的类，最安全的使用方式是使用嵌套的try子句。

## 异常匹配

抛出异常的时候，异常处理系统会按照代码的书写顺序找出“最近”的处理程序。找到匹配的处理程序之后，它就认为以后吃那个将得到处理，然后就不再继续查找。

查找的时候并不要求抛出的异常同处理程序所声明的异常完全匹配。派生类的对象也可以匹配其基类的处理程序，就像这样：

```java
package exceptions;

class Annoyance extends Exception{}
class Sneeze extends Annoyance{}

public class Human {

	public static void main(String[] args) {
		try {
			throw new Sneeze();
		}catch (Sneeze s) {
			System.out.println("Caught Sneeze");
		}catch (Annoyance a) {
			System.out.println("Caught Annoyance");
		}
		
		try {
			throw new Sneeze();
		}catch (Annoyance a) {
			System.out.println("Caught Annoyance");
		}
	}
}
/**
Caught Sneeze
Caught Annoyance
*/
```

Sneeze异常会被第一个匹配的catch子句捕获，也就是程序里的第一个。然而如果将这个catch子句删掉，只留下Annoyance的catch子句，该程序仍然能运行，因为这次捕获的是Sneeze的基类。换句话说，catch(Annoyance e)会捕获Annoyance以及所有从它派生的异常。这一点非常有用，因为如果决定在方法里加上更多派生异常的话，只要客户程序员捕获的是基类异常，那么它们的代码就无需更改。

如果把捕获基类的catch子句放在最前面，以此把派生类的异常全给“屏蔽掉”，就像这样：

```java
try{
    throw new Sneeze();
}catch(Annoyance a){
    //...
}catch(Sneeze s){
    //....
}
```

这样编译器就会发现Sneeze的catch子句永远也得不到执行，因为它会向你报告错误。

## 其他可选方式

**异常处理的一个重要原则是：只有在你知道如何处理的情况下才捕获异常。**

## 异常使用指南

应该在下列情况下使用异常：

- 在恰当的级别处理问题。（在知道该如何处理的情况下才能捕获异常）
- 解决问题并且重新调用产生异常的方法。
- 进行少许修补，然后绕过异常发生的地方继续执行。
- 用别的数据进行计算，以代替方法预计会返回的值。
- 把当前运行环境下能做的事情尽量做完，然后把相同的异常重新抛到更高层。
- 把当前运行环境下能做的事情尽量做完，然后把不同的异常抛到更高层。
- 终止程序
- 进行简化。（如果你的异常模式使问题变得太复杂，那用起来会非常痛苦也很烦人）
- 让类库和程序更安全。（这既是在为调试做短期投资，也是在为程序的健壮性做长期投资。）

[练习题](https://github.com/wangwren/Thinking-in-Java-notes/tree/master/Chapter12Exercise/src/exceptions)
