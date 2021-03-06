

# Activity生命周期

问题1:onStart()和onResume(),onPause()和onStop(),有什么实质的不同.
onStart()和onStop()是从Activity是否可见这个角度来回调的,
而onResume()onPause()是从Activity是否位于前台(是否可交互)这个角度回调的,其他就没有什么实质的差别的.

问题2:假设当前Activity为A,如果这时用户打开一个新的ActivityB,那么B的onResume和A的onPause()那个先执行?
旧的Activity先onPause(),然后新的Activity再启动.

# Activity的启动模式

任务栈和taskAffinity

1. standard:

2. singleTop:   

onNewIntent()

3. singleTask:

onNewIntent():

4. singleInstance:


# IPC简介

IPC是Inter-Process Communication的缩写,含义是进程间通信或者跨进程通信,是指两个进程间进行数据交换的过程.

Window的IPC,可以通过剪切板,管道,邮槽进行进程间通讯.
Linux上可以通过命令管道,共享内存,信号量等来进行进程间通讯.
Android中使用Binder和Socket来进行进程间通讯.

同一个app内部的多进程,会出现的问题:
1. 静态成员和单例模式完全失效
2. 线程同步机制完全失效
3. SharePreferences的可靠性下降
4. Application会多创建.

这是因为,当一个组件跑在一个新的进程中的时候,由于系统要在创建新的进程同时分配独立的虚拟机,所以这个过程其实就是
启动一个应用过程.

# Binder介绍

Binder是Android的一个类,它实现了IBinder接口.从IPC角度来说,Binder是Android的一种跨进程通信方式,Binder还可以理解为一种虚拟的物理设备.
从AndroidFramework角度来说,Binder是ServiceManager链接各种Manager(ActivityManager,WindowManager)和ManagerService的桥梁.
从Android应用层来说,Binder是客户端和服务端进行通讯的媒介,当bindService的时候,服务端返回一个包含了服务端业务调用的Binder对象,

| 名称 | 优点 | 缺点 | 适用场景
| :--------: |:-------------:| :-----:| :---:|
| **Bundle** | 简单易用　| 只能Bundle支持的数据类型 | 四大组件间的进程通信
| **文件共享** | 简单易用 | 不适合高并发场景，并且无法做到进程间的即时通讯 | 无并发访问情形，交换简单的数据实时性不高的场景
| **AIDL** | 功能强大，支持一对多并发通讯，支持实时通讯 | 适用稍复杂，需要处理好线程同步 | 一对多通讯且有RPC(远程过程调用协议)需求
| **Messager**|功能一般，支持一对多串行通信，支持实时通讯|不能很好的处理高并发情形，不支持RPC，数据通过Message进行传输，因此只能传输Bundle支持的数据类型|低并发的一对多即时通讯，无RPC需求，或者无需要返回结果的RPC需求
| **ContentProvider**|在数据源访问方面功能强大，支持一对多并发数据共享，可通过Call方法扩展其他操作|可以理解为受约束的AIDL，只要提供数据源的CRUD操作|一对多进程间的数据共享
| **Socket**|功能强大，可以通过网络传输字节流，支持一对多并发实时通讯|实现细节稍微麻烦，不支持直接RPC|网络数据交换

# 消息机制

主要涉及到Handler,Message,MessageQueue,Looper

Android的消息机制主要是线程间通信使用的.平常的用途主要在于更新UI.

Looper是一个消息循环,不断在MessageQueue队列中取出消息,进行分发,Looper和MessageQueue是一一对应的,在一个线程中只能有一个Looper和MessageQueue,使用ThreadLocal来存储.

而Handler则可以有多个实例,在创建Handler对象时,会获取当前线程的Looper和MessageQueue,主要用途是用于发送消息到队列中.

Looper.looper()之后,不断在MessageQueue中取出消息,然后进行dispatchMessage(),最后在Handler的handleMessage()处理消息结果.

我们所说的主线程,在其内部也开启了一个Looper循环.在ActivityThread.main()方法中,通过Looper.prepareMainLooper()和Looper.loop()开启了消息循环.其loop()内部是一个死循环,

其中开启了一个Binder线程,用于接收系统服务的消息.

当有消息时,会从MessageQueue.next()方法中取出消息,如果没有消息,则会阻塞在next()方法上,但是这种阻塞不会造成主线程卡死,之所以不会卡死,

是因为在没有消息时,主线程释放了CPU资源,进入了休眠状态,直到下一个消息或事物发生时,通过往pipe管道写入数据来唤醒主线程工作.

这里采用epoll机制,是一种IO多路复用的机制,同时监控多个描述符,当描述符就绪,则立刻通知相应程序进行读写操作,本质是同步IO,读写是阻塞的.

所以说,主线程大多数时候是处于休眠状态的,并不会消耗大量CPU资源.

# View动画的方式

1. scrollTo/scrollBy,注意是对view内容的滑动
2. 动画
3. 改变布局参数

# 测量流程,布局流程,和绘制流程

ViewRootImpl的performTraversals开始,

ViewGroup

performMeasure -> measure -> onMeasure

performLayout -> layout -> onLayout

performDraw -> draw -> onDraw

performTraversals会依次调用performMeasure,performLayout和performDraw,这三个方法分别完成顶级View的measure,layout,draw这三大流程.

performMeasure中会调用measure方法,measure()方法又会调用onMeasure()方法,在onMeasure()方法中则会对所有子元素进行measure过程,这时measure流程就从父容器传递到
子容器中了,这样就完成了一次measure过程,接着子元素重复父元素的measure过程,如此反复就完成了整个View树的遍历.

在View测量的时候,系统会将LayoutParams在父容器的约束下转换成对应的MeasureSpec,然后再根据这个MeasureSpece来决定的,在根据这个MeasureSpec来确定View测量后的宽高.

对于DecorView其MeasureSpec由窗口的尺寸和气自身的LayoutParams来公共决定(getRootMeasureSpec()),对于普通的View由父亲的MeasureSpec和自身的LayoutParams来共同决定(getChildMeasureSpec()).

以父亲是WRAP_CONTENT为例
:如果孩子的布局大小是准确的尺寸,则模式是精准的.如果孩子的布局大小是MATCH_PARENT,则尺寸是父亲的大小,且模式是最大的.如果孩子的布局大小是WRAP_CONTENT的,则尺寸是父亲的大小,切成模式是最大的.

# android的invalidate()和postInvalidate()的区别

Android中实现View更新的又两组方法,一组是invalidate(),另一组是postInvalidate(),其中前者是在UI线程中自身中使用的,而后者是在非UI线程中使用的.

为什么这样呢?之所以不能在线程中直接使用invalidate(),因为它违背了单线程模型,android的UI线程并不是线程安全的,并且这些更新UI的操作必须的UI线程操作.

# Android的单线程模型

当第一个程序第一次启动的时候,Android会同时启动一个对应的主线程(Main Thread),主线程主要负责处理与UI相关的事件,
如:用户的按键事件,用户接触屏幕的事件,以及屏幕绘制事件,并把相关的事件分发到对应的组件进行处理.
所以主线程又通常被叫做UI线程.

在开发Android应用时必须遵守单线程模型的原则:Android UI操作并不是线程安全的并且这些操作必须在UI线程中执行.

由于UI线程负责事件的监听和绘制,因此,必须保证UI线程能够随时响应用户的需求,UI线程里的操作应该像中断操作那样短小,费时间的操作需要另开线程,否则UI线程超过5s没有响应用户请求,会弹出对话框提醒用户终止应用程序.

如果在新开的线程中需要对UI线程进行更新,就可能违反单线程模型,因此Android采用消息机制保证线程间通信.

MessageQueue Looper Handler

# AttributeSet是获取自定义属性的.context.obtain

# View中的常用方法


|Category|	Methods|	Description|
|---|---|---|
|Creation|	Constructors	|There is a form of the constructor that are called when the view is created from code and a form that is called when the view is inflated from a layout file. The second form should parse and apply any attributes defined in the layout file.|
| |onFinishInflate()	|Called after a view and all of its children has been inflated from XML.|
|Layout|	onMeasure(int, int)	|Called to determine the size requirements for this view and all of its children.|
| |onLayout(boolean, int, int, int, int)	|Called when this view should assign a size and position to all of its children.|
| |onSizeChanged(int, int, int, int)	|Called when the size of this view has changed.||
|Drawing|	onDraw(android.graphics.Canvas)	|Called when the view should render its content.
|Event processing|	onKeyDown(int, KeyEvent)|	Called when a new hardware key event occurs.|
| |onKeyUp(int, KeyEvent)	|Called when a hardware key up event occurs.|
| |onTrackballEvent(MotionEvent)	|Called when a trackball motion event occurs.
| |onTouchEvent(MotionEvent)	|Called when a touch screen motion event occurs.|
|Focus	|onFocusChanged(boolean, int, android.graphics.Rect)	|Called when the view gains or loses focus.||
| |onWindowFocusChanged(boolean)	|Called when the window containing the view gains or loses focus.
|Attaching	|onAttachedToWindow()	|Called when the view is attached to a window.|
| |onDetachedFromWindow()	|Called when the view is detached from its window.|
| |onWindowVisibilityChanged(int)	|Called when the visibility of the window containing the view has changed.|

# View事件分发机制

1. dispatchTouchEvent(MotionEvent ev)
2. onInterceptTouchEvent(MotionEvent ev)
3. onTouchEvent(MotionEvent ev);

```java
public boolean dispatchTouchEvent(MotionEvent ev){
    boolean consume = false;
    if (onInterceptTouchEvent(ev){
        consume = onTouchEvent(ev);
    } else {
        consume = child.dispatchTouchEvent(ev);
    }
    return consume;
}
```

事件的传递顺序:
Activity -> Window ->View

Activity的dispatchTouchEvent 具体由Window来完成的, Window传给DecorView

# 理解Window和WindowManager

Android 中所有的视图都是通过Window来呈现的,不管是Activity,Dialog还是Toast,他们都是附加在Window上的,因此Window是View的实际管理者.

Window是一个抽象概念,每一个Window都对应这一个View和一个ViewRootImpl,Window和View通过ViewRootImpl来建立联系. 

WindowManager添加一个View的过程:
1. WindowManagerImpl -> WindowManagerGlobal -> addView -> 创建ViewRootImpl -> setView()方法更新View -> WindowSession (IPC) -> WindowManagerService处理．

Activity的Window创建过程:
ActivityThread执行performLaunchActivity()操作,然后会回调到Activity.attach方法中,创建window(new PhoneWindow()).
在setContentView(),创建Decor视图,并将视图加入到Decor视图中,最后在makeVisible()时,将Decor视图加入到Window上.

# ANR

程序无响应,根本原因是在Ui界面进行了太多耗时操作.

android规定Activity超过5秒钟无响应会出现ANR,在Service中10秒钟无响应会出现ANR.

如何分析出处?分析traces文件.

如何解决,将所有耗时操作放入到工作线程.
1. Thread+Handler,这样开销比较大
2. AsyncTask虽然轻量,但是使用时有非常多需要注意的点,1.串行并行版本2.cancel不太好用3.内存泄露
3. RxJava


# OOM 内存溢出

使用了太多的内存,而没有释放.加载了大图片.



# 内存泄露

根本原因是无用对象无法被GC回收,也就是长声明周期的对象持有了短生命周期的对象,导致短生命周期对象无用后无法被释放.

3. 内部类引用外部类对象:在Activity内部,匿名内部类和非静态内部类都会持有Activity的引用对象,当匿名内部类和非静态内部类生命周期不确定时,就很容一产生内存泄露.以及为非静态内部类创建静态实例,也会造成内存泄露.解决方法:尽量创建静态内部类.
1. 静态变量:错误的使用静态变量,例如使用静态变量引用了Context
2. 监听器(Listener)没有被取消,由于在Activity内部的匿名内部类引用了Activity,如果监听器没有被取消,就很容易内存泄露.
5. Handler:非静态的Handler内部类引用了Acitivity对象,当发送消息时Handler和Looper以及MessageQueue关联在一起,当消息没有被处理时,就会一直持有Handler,而Handler持有Activity,也就引起了内存泄露.解决:静态内部类+软或弱引用.
4. 单例模式:很多情况下都会传入Context以方便使用,由于是单例的生命周期和应用一致,并且创建单例时传入Activity作为Context,导致单例引用了Acitivty,无法被释放,发生内存泄露.解决方法:context.getApplicationContext();
6. 资源没有被释放:Bitmap.recycler();

# 优化

布局优化/线程优化/绘制优化/内存泄露优化/OOM优化/响应速度优化/

# FragmentPagerAdapter与FragmentStatePagerAdatper

主要区别在于fragment是否销毁.
FragmentPagerAdapter对于不在需要的framgent,选择调用detach()方法,仅销毁视图,并不会销毁fragment实例.
FragmentStatePagerAdapter,会销毁不在需要的fragment.

FrragmentStatePagerAdapter更节省内存,但是销毁和创建要消耗时间.
在页面不多的情况下,使用FragmentPagerAdapter,在较多的情况下使用FragmentStatePagerAdapter
