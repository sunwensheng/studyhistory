# Notes #
### Wensheng Sun ###
### Fight for the world and first try ###
#### This is for Sun’s feature at 2016/8/22 21:27####
***
##Content##
### 1 View的事件体系 ###
####1.1 View的基础知识####
	x=left+translationX
	y=top+translationY
其中，left和top这两个坐标都是相对于View的父容器而言的，因此它是一种相对的坐标。x和y是`View`的左上角的坐标，`translationX`和`translationY`是`View`的左上角相对于父容器的偏移量。这几个参数都是相对于父容器的坐标。**在`View`的平移过程中，top和left表示的是原始左上角的位置信息，其值不会改变，此时改变的是x，y，`translationX`和`translationY`这四个参数。**

**MotionEvent**是手指触摸屏幕的行为触发一系列的点击事件。通过MotionEvent对象我们可以得到点击事件发生的x和y的坐标。`getX/getY`返回的是相对于当前View左上角的x和y坐标，而`getRawX/getRawY`返回的是相对于手机屏幕左上角的x和y坐标。

**TouchSlop**是系统所能识别出来的呗认为是滑动的最小距离：滑动的距离太短，系统不认为它是滑动。**这是一个常量，和设备有关。**通过`ViewConfiguration.get(getContext()).getScaledTouchSlop`可获得这个常量。**当我们处理滑动时，可以利用这个常量进行一些过滤，未达到此值时，可认为不是滑动**

**VelocityTracker**:用于追踪手指在滑动中的速度，包括数值和水平上的速度。若要获取当前的速度，需要在当前View的**onTouchEvent**中追踪当前的点击事件的速度。

	VelocityTracker.velocityTracker=VelocityTracker.obtain();
	velocityTracker.addMoveMent(event);

当我们知道当前的速度，可以如下方式获得当前的速度：

	velocityTracker.computeCurrentVelocity(1000);//单位时间是可以变的
	int xVelocity=(int)velocityTracker.getXVelocity();
	int yVelocity=(int)velocityTracker.getYVelocity();
速度=（ 终点距离 - 起点距离 ）/ 时间段

这一步中需注意：获取速度之前需要先计算速度，即`computeCurrentVelocity`。其中速度的值可正可负。当不需要使用的时候，需要条用`clear`方法来重置并回收内存：

	velocityTracker.clear();
	velocityTracker.recycle();


**GestureDetector**：手势检测，用于辅助检测用户的单击，滑动，长按，双击等行为。实际的开发过程中，可以不适用`GestureDetector`，完全可以在`View`的`onTouchEvent`方法中实现所需要的监听。建议：如果是滑动相关的，可以再onTouchEvent中自己实现，如果是监听双击这种行为的话，那么可以使用`GestureDetector`.

**Scroller**:弹性滑动对象，用于实现`View`的弹性滑动。`scrollTo/scrollBy`方法进行滑动时，是瞬间完成的。`Scroller`是在时间间隔内完成的，实现有过渡效果的滑动。`Scroll`本身没有办法让View弹性滑动，它需要和`View`的`computeScroll`方法配合使用才能共同完成这个功能。
####1.2 View的滑动####
实现滑动的三种方式：

* `scrollTo/scrollBy`实现滑动
* 通过动画给`View`施加平移效果来实现滑动
* 通过改变`View`的`LayoutParams`使得`View`重新布局从而实现滑动

**scrollTo/scrollBy** `scrollBy`实际上也是调用了`scrollTo`方法。前者实现了相对的滑动，后者实现了基于传递参数的绝对滑动。我们需要了解`mScrollX`和`mScrollY`的改变规则，这两个属性可以通过`getScrollX`和`getScrollY`获得。`mScrollX`表示`View`的左边缘和`View`内容左边缘在水平方向的距离。且当`View`左边缘在View内容左边缘的右边时，mScroll为正值，反之为负值。mScrollY同理可推。**scrollTo和scrollBy只能改变View内容的位置，而不能改变View在布局中的位置。**

**动画滑动**传统的View动画并不能真正改变View的位置，只是对View的影响做了操作，它并没有改变View的位置参数。如果希望动画后的状态得以保留，必须将fillAfter属性设置为true，否则动画完成后其动画结果就会消失。如果有点击事件，动画后的相关位置的点击是没有点击事件的，点击原始的相关位置测试可以的。当然，这可以通过灵活的解决方式进行处理。以上的问题，在属性动画里都可以进行很好的解决。但是，属性动画对安卓3.0以下是不支持的，需要用到动画兼容库nineoldandroids来处理，本质上还是传统的动画。

**改变布局参数**
####1.3 弹性滑动####
####1.4 View事件的分发机制####
点击事件的传递规则中，其中分析的对象就是MotionEvent，即点击事件。所谓的点击事件的事件分发，就是对MotionEvent事件的分发过程。点击事件的分发由三个重要的方法来共同完成：`dispatchTouchEvent`，`onInterceptTouchEvent`和`onTouchEvent`。

    public boolean dispatchTouchEvent(MotionEvent ev) 
用来进行事件的分发，如果事件能够传递给当前的View，那么此方法一定会被调用。返回的结果受当前的View的`onTouchEvent`和下级View的`dispatchTouchEvent`方法的影响，表示是否消耗当前事件。

	public boolean onInterceptTouchEvent(MotionEvent event)
在上述方法内进行调用，用来判断是否拦截某个事件，如果当前View拦截了某个事件，那么在同一事件序列中，此方法不会被再次调用，返回结果表示是否拦截当前事件。

	public boolean onTouchEvent(MotionEvent event)
在`dispatchTouchEvent`中调用，用来处理点击事件，返回结果表示是否拦截当前事件，如果不消耗，在同一个事件序列中，当前View无法再次接收到事件。

上述的三种方法之间的关系可用如下伪代码表示：

	public boolean dispatchTouchEvent(MotionEvent ev){
		boolean consume = false;
		if (onInterceptTouchEvent(ev)){
			consume = onTouchEvent(ev);
		} else {
			consume = child.dispatchTouchEvent(ev);
		}
		return consume;
	}
当一个View需要处理事件时，如果它设置了`OnTouchListener`，那么其中的`onTouch`方法就会被调用，这时事件如何处理还要看`onTouch`的返回值。如果返回false，则当前View的`onTouchEvent`方法会被调用；如果返回true，那么`onTouchEvent`将不会被调用。说明，`OnTouchListener`比`onTouchEvent`优先级高，`OnClickListener`的优先级最低，处于事件传递的尾端。
如果View的`onTouchEvent`返回false，那么它的父容器的`onTouchEvent`就会被调用。

事件传递机制，给出结论如下所示

1.   同一事件序列是指手指接触屏幕的那一刻起到手指离开屏幕的那一刻结束。即`down --> n*move -->up `
2.   每个View一旦决定拦截，那么这一个事件序列都只能由它来处理，并且它的`onInterceptTouchEvent`不会再被调用。
3.   某个VIew一旦开始处理事件，如果不消耗ACTION_DOWN事件（`onTouchEvent`返回false），那么同一事件中剩下的事件就不再交给它来处理，那么它的父元素的`onTouchEvent`就会被调用。
4.   如果View消耗除ACTION_DOWN之外的其他事件，那么这个点击事件就会消失，此时父元素的`onTouchEvent`并不会被调用，并且当前的View可以持续收到后续的事件，最终这些消失的点击事件会传给Acticity处理。
5.   ViewGroup默认不拦截任何事件，ViewGroup的`onInterceptTouchEvent`返回false。
6.   View没有`onInterceptTouchEvent`方法，一旦有点击事件传给它，它的onTouchEvent方法就会被调用。
7.   View的onTouchEvent默认都会消耗事件（返回true），除非它是不可点击的（clickable和longClickable都为false）。View的longClickable默认都为false，clickable分情况，Button默认为true，TextView默认为false。
8.   View的enable不会影响onTouchEvent的默认返回值。只要clickable或者longClickable有一个为true，那么onTouchEvent就返回true；
9.   onClick会发生的前提是当前View是可点击的，并且收到了down和up的事件。
10.   事件的传递过程是由外向内的，事件总是先传给父元素，谈后由父元素传给子View，通过`requestDisallowInterceptTouchEvent`方法可以在子元素中干预父元素的事件分发过程，但是`ACTION_DOWN`事件除外。

**Activity对事件的分发过程**点击事件用MotionEvent来表示，当一个点击事件发生时，事件最先传给当前的额Activity，由Activity的dispatchTouchEvent来进行事件的分发，具体的工作是由Window来完成的。Window会将事件传递给decor view，它就是当前界面的底容器（即setContentView设置的View的父容器），通过`Activity.getWindow.getDecorView()`可以获得，它继承自FrameLayout。通过
`((ViewGroup)getWindow.getDecorView().findViewById(android.R.id.content)).getChildAt(0)`这种方式就可以获得Activity所设置的View，这时，事件就传递到顶级View，即在Activity中通过`setContentView`所设置的View。另外，顶级View也叫根View，一般来说都是ViewGroup。

**顶级View对点击事件的分发过程**

在ViewGroup中，判断是否要拦截当前事件的条件为：`actionMasked==ACTION_DOWN||mFirstTouchTarget！=null`其中，当事件由ViewGroup的子元素成功处理时，mFirstTouchTarget就会被赋值并指向子元素。当点击事件的ACTION_DOWN传递来的时候，由判断条件可知，需进行是否拦截的判断，调用ViewGroup的`onInterceptTouchEvent`进行判断。当`actionMasked==ACTION_DOWN||mFirstTouchTarget！=null`这个条件为false时，ViewGroup的`onInterceptTouchEvent`就不会进行调用，同一序列中的其他事件都会交给ViewGroup处理。

有一种特殊的情况，就是`FLAG_DISALLOW_INTERCEPT`标记位，这个标记为是通过`requestDisallowInterceptTouchEvent`方法来设置的，一般用于子View中。`FLAG_DISALLOW_INTERCEPT一`旦设置，ViewGroup将会无法拦截除了ACTION_DOWN之外的点击事件。这是因为ViewGroup在分发事件时，如果是`FLAG_DISALLOW_INTERCEPT`就会重置这个标记位，导致子View的这个标记位无效。

总结一下：`onInterceptTouchEvent`不是每次事件都会被调用，如果想提前处理所有的点击事件，要选择`dispatchTouchEvent`方法，只有这个方法能确保每次都会调用，前提是事件能够传递到当前的ViewGroup。另外，`FLAG_DISALLOW_INTERCEPT`标记位的可以用在解决滑动冲突的问题。

当ViewGroup不拦截事件的时候，它会首先会for循环遍历ViewGroup的所有子元素，然后判断是否能够接收到点击事件。能否获得点击事件由两点进行判断：子元素是否在播放动画和点击事件的坐标是否落在子元素的区域内。如果每个子元素满足这两个条件，那么事件就会传递给它处理。 古国子元素的`dispatchTouchEvent`返回true，那么mFirstTouchTarget就会被赋值并跳出for循环。如果`dispatchTouchEvent`返回的是false，ViewGruop将会把事件分发给下一个子元素，如果有的话。

如果遍历所有的子元素都没有被合适的处理，这包含两种情况，1 ViewGroup没有子元素。2 子元素处理了点击事件，但是在`dispatchTouchEvent`中反悔了false，这一般是因为子元素在`onTouchEvent`中返回了false。

**View对点击事件的处理过程**
View对点击事件的处理，首先判断是否设置了`OnTouchListener，`如果`OnTouchListener`中的`onTouch`方法返回true，那么`onTouchEvent`就不会被调用。**`OnTouchListener`的优先级比`onTouchEvent`的高，这样的好处就是方便在外界处理点击事件。**
####1.5 View的滑动冲突####
在界面中只要内外两层同时可以滑动，这个时候就会产生滑动冲突。常见的滑动场景如下：

1. 外部滑动方向和内部滑动方向不一致
2. 外部滑动方向和内部滑动方向一致
3. 上面两种情况的结合

对于1的场景，可以通过水平和竖直的距离进行判断，或者角度进行判断。对于2，3场景不能直接用以上的方法进行解决，都是从业务的需求上得出相关的处理规则。

处理的两种方式：

1. 外部拦截法
所谓外部拦截法是指点击事件都经过父容器的拦截处理，如果父容器需要此事件就进行拦截，不需要就不拦截，这种方法比较符合点击事件的分发机制。外部拦截法需要重写父容器的`onInterceptTouchEvent`方法。首先是`ACTION_DOWN`这个事件，必须返回false，即不拦截此事件。因为父容器一旦拦截`ACTION_DOWN`事件，后续的`ACTION_MOVE`和`ACTION_UP`都会直接由父容器处理。`ACTION_MOVE`根据实际情况决定是否拦截。`ACTION_DOWN`必须返回false。（如果父容器在ACTION_UP是返回了true，就会导致子元素无法接收到此事件，这个时候子元素的onClick事件就无法触发。）

2. 内部拦截法

内部拦截发是指父容器不拦截任何事件，所有的事件都传递给子元素，如果子元素需要此事件就直接消耗掉，否则就交由父容器进行处理。这个Android中的分发机制不同，需要配合`requestDisallowInterceptTouchEvent`方法才能正常工作，比外部拦截法稍显复杂。此时父容器的`onInterceptTouchEvent`也得重写。

推荐使用外部拦截法，滑动冲突的解决是有规则的，对于不同的场景，只需要更改相关滑动规则的逻辑即可。

8/24/2016 4:24:05 PM 

### 2 View的工作原理 ###
####1.1 初识ViewRoot和DecorView####
ViewRoot对应于ViewRootImpl类，它是连接WindowManager和DecorView的纽带，VIew的三大绘制流程均是通过ViewRoot来完成，它经过measure，layout和draw三个过程才最终将一个View绘制出来。其中measure是测量View的宽和高，layout用来确定View在父容器中的放置位置，而draw是用来将View绘制在屏幕上。

measure过程决定了View的宽和高，完成之后，可以通过getMeasuredWidth和getMeasuredHeight方法来获得测量过之后的宽和高，基本上是等于View的最终高度，除特殊情况外。Layout过程决定了View的四个顶点的坐标和实际的宽和高，完成之后，可以通过getTop，getBottom，getLeft和getRight来获得View的四个顶点的位置，并可以通过getWidth和getHeight方法获得最终的宽和高。Draw过程则决定了View的显示，只有draw方法完成之后View的内容才能呈现在屏幕上。

DecorView作为顶级View，一般会包含一个竖直方向上的LinearLayout，在该LinearLayout中分上下两部分，上面是标题栏，下面是内容栏。Activity中通过setContentView设置的布局就是添加到其中，id是content。可通过以下方式获得：ViewGroup content=findViewById（R.id.content),可通过content.getChild(0);其实DecorView是一个FramLayout，View层的事件都先经过DecorView，再传给我们的View。
####1.2 MeasureSpec####
Measurespec在很大程度上决定了一个View的尺寸规格，这个过程还受父容器的影响，因为父容器影响View的MeasureSpec的创建过程。在测量的过程中，系统会将View的LayoutParams根据父容器所施加的规则转换成对应的MeasureSpec，然后根据这个MeasureSpec来测量出View的宽和高，这是测量的值，并非最终的值。

### 3 自定义控件 ###

### 代码整洁之道 ###
Tips:在代码的处理过程中，尽量的对数据进行封装。分模块，分功能。小而精。整洁的边界，和适配器模式有共同之处。

不要在生产代码中实验新的东西，而是编写测试来遍览和理解第三方代码，这种方式可理解为**学习性测试**。学习性测试确保第三方程序包按照我们想要的方式工作。伴随着程序包的修正和添加新功能，如果第三方的修改与测试不兼容，就可马上发现。在调用第三方API时，对其打包是个良好的实践手段，当你对一个第三方的API进行打包，就降低了对它的依赖。在以后的，可以较为轻松的更换其他的代码库。在测试自己的代码时，打包有助于模拟第三方的调用。拥有一个整洁的**代码边界**，对于调用第三方程序包有很大的好处。


**特例模式**：创建一个类或配置一个对象，用来处理特例，使返回的数据都是同一类型。你来处理特例，客户代码就不用应付异常的行为了。异常行为封装在特例对象中，比如当为null这种特例。

**别返回null**：如果你在方法中准备返回null，不如抛出异常或者返回特例对象。如果你在调用第三方的API时可能返回null，可以考虑使用新方法打包这个方法，在新方法中抛出异常或返回特例对象。在许多情况下，特例对象都是极好的。不返回null，可以在以后的使用中不进行非空判断，使代码更简洁。**在传递给方法参数时，尽量避免传递null值。**

边界上的代码需要清晰的分割和定义期望的测试。应该避免我们的代码过多的了解第三方代码中的特定信息。依靠我们自己能够控制的东西，好过依赖你控制不了的东西，免得然后受它控制。可以通过adapter模式将我们的接口转换为第三方提供的接口。

需要保持变量和工具函数的私有性，单并不执着于此。我们首先会想办法保持私有，放松封装总是下策。




1. 枝节性代码游离在功能性代码之外，它下是函数的目的，这是对OO是一种破坏
2. 枝节性代码会造成功能性代码对其它类的依赖，加深类之间的耦合，而这是OO系统所竭 力避免的
3. 枝节性代码带来的耦合度会造成功能性代码移植困难，可重用性降低
4. 从法理上说，枝节性代码应该监视着功能性代码，然后采取行动，而不是功能性代码通知枝节性代码采取行动，这好比吟游诗人应该是主动记录骑士的功绩而不是骑士主 动要求诗人记录自己的功绩

紧耦合的代码难以编写测试。使用依赖注入，接口和抽象尽可能减少耦合。

提升内聚性，切分关注面，模块化系统性关注面，缩小函数和类的尺寸，选用更好的名称，如此等等。这也是应用简单设计后三条规则的地方：消除重复，保证表达力，尽可能减少类和方法的数量。

**面向对象的3个基本要素：封装、继承、多态**

**面向对象的5个基本设计原则：**

SRP原则（Single Responsibility Principle）
就一个类而言，应该只专注于一件事和仅有一个引起它变化的原因

单一职责原则（Single-Resposibility Principle）

其核心思想为：一个类，最好只做一件事，只有一个引起它的变化。单一职责原则可以看做是低耦合、高内聚在面向对象原则上的引申，将职责定义为引起变化的原因，以提高内聚性来减少引起变化的原因。职责过多，可能引起它变化的原因就越多，这将导致职责依赖，相互之间就产生影响，从而大大损伤其内聚性和耦合度。通常意义下的单一职责，就是指只有一种单一功能，不要为类实现过多的功能点，以保证实体只有一个引起它变化的原因。专注，是一个人优良的品质；同样的，单一也是一个类的优良设计。交杂不清的职责将使得代码看起来特别别扭牵一发而动全身，有失美感和必然导致丑陋的系统错误风险。

开放封闭原则（Open-Closed principle）

其核心思想是：软件实体应该是可扩展的，而不可修改的。也就是，对扩展开放，对修改封闭的。开放封闭原则主要体现在两个方面1、对扩展开放，意味着有新的需求或变化时，可以对现有代码进行扩展，以适应新的情况。2、对修改封闭，意味着类一旦设计完成，就可以独立完成其作，而不要对其进行任何尝试的修改。实现开开放封闭原则的核心思想就是对抽象编程，而不对具体编程，因为抽象相对稳定。让类依赖于固定的抽象，所以修改就是封闭的；而通过面向对象的继承和多态机制，又可以实现对抽象类的继承，通过覆写其方法来改变固有行为，实现新的拓展方法，所以就是开放的。
“需求总是变化”没有不变的软件，所以就需要用封闭开放原则来封闭变化满足需求，同时还能保持软件内部的封装体系稳定，不被需求的变化影响。

Liskov替换原则（Liskov-Substituion Principle）

其核心思想是：子类必须能够替换其基类。这一思想体现为对继承机制的约束规范，只有子类能够替换基类时，才能保证系统在运行期内识别子类，这是保证继承复用的基础。在父类和子类的具体行为中，必须严格把握继承层次中的关系和特征，将基类替换为子类，程序的行为不会发生任何变化。同时，这一约束反过来则是不成立的，子类可以替换基类，但是基类不一定能替换子类。Liskov替换原则，主要着眼于对抽象和多态建立在继承的基础上，因此只有遵循了Liskov替换原则，才能保证继承复用是可靠地。实现的方法是面向接口编程：将公共部分抽象为基类接口或抽象类，通过Extract Abstract Class，在子类中通过覆写父类的方法实现新的方式支持同样的职责。Liskov替换原则是关于继承机制的设计原则，违反了Liskov替换原则就必然导致违反开放封闭原则。Liskov替换原则能够保证系统具有良好的拓展性，同时实现基于多态的抽象机制，能够减少代码冗余，避免运行期的类型判别。

依赖倒置原则（Dependecy-Inversion Principle）

其核心思想是：依赖于抽象。具体而言就是高层模块不依赖于底层模块，二者都同依赖于抽象；抽象不依赖于具体，具体依赖于抽象。我们知道，依赖一定会存在于类与类、模块与模块之间。当两个模块之间存在紧密的耦合关系时，最好的方法就是分离接口和实现：在依赖之间定义一个抽象的接口使得高层模块调用接口，而底层模块实现接口的定义，以此来有效控制耦合关系，达到依赖于抽象的设计目标。抽象的稳定性决定了系统的稳定性，因为抽象是不变的，依赖于抽象是面向对象设计的精髓，也是依赖倒置原则的核心。依赖于抽象是一个通用的原则，而某些时候依赖于细节则是在所难免的，必须权衡在抽象和具体之间的取舍，方法不是一层不变的。依赖于抽象，就是对接口编程，不要对实现编程。

接口隔离原则（Interface-Segregation Principle）

其核心思想是：使用多个小的专门的接口，而不要使用一个大的总接口。具体而言，接口隔离原则体现在：接口应该是内聚的，应该避免“胖”接口。一个类对另外一个类的依赖应该建立在最小的接口上，不要强迫依赖不用的方法，这是一种接口污染。接口有效地将细节和抽象隔离，体现了对抽象编程的一切好处，接口隔离强调接口的单一性。而胖接口存在明显的弊端，会导致实现的类型必须完全实现接口的所有方法、属性等；而某些时候，实现类型并非需要所有的接口定义，在设计上这是“浪费”，而且在实施上这会带来潜在的问题，对胖接口的修改将导致一连串的客户端程序需要修改，有时候这是一种灾难。在这种情况下，将胖接口分解为多个特点的定制化方法，使得客户端仅仅依赖于它们的实际调用的方法，从而解除了客户端不会依赖于它们不用的方法。分离的手段主要有以下两种：1、委托分离，通过增加一个新的类型来委托客户的请求，隔离客户和接口的直接依赖，但是会增加系统的开销。2、多重继承分离，通过接口多继承来实现客户的需求，这种方式是较好的。
