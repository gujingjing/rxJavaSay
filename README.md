####一、什么是rxJava
一个词:异步
rxJava在github主页上的介绍是
 "a library for composing asynchronous and event-based programs using observable sequences for the Java VM"
大概的意思就是一个在 Java VM 上使用可观测的序列来组成异步的、基于事件的程序的库。

其实rxJava的本质就是一个词，异步，它就是一个异步操作的库。


####二、rxJava的好处
简洁
异步操作的比较关键的一点就是程序的简洁，在调用复杂的异步操作的时候，代码回显得很复杂，不仅难写也很难懂。虽然android 创造的asynTask和handler 都是为了让代码更加简洁。
rxJava的优势在于，随着程序逻辑越来越复杂，代码依然很清晰


####三、rxJava的基本原理

rxJava实现异步，是通过扩展观察者模式来实现的。

首先，讲述下，观察者模式
观察者模式即是，a对象对b对象的某一个动作特别关注，做着密切的观察，当a对象做出了这个动作的时候，b对象立刻做出相应的处理。就好比android中的点击事件(onClickListener),


![onClickListener.jpg](http://upload-images.jianshu.io/upload_images/1387450-8e247ee2b817d1a5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


当button按钮被点击的时候，观察者对这个点击事件做出自己的反应

转变为通用的观察者模式如下:

![rxJava.jpg](http://upload-images.jianshu.io/upload_images/1387450-f1c623f733a45af5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

observable被观察者，在做出某一事件的时候，通知observer观察者做出处理


RxJava使用的就是通用型的观察者模式。

#####RxJava观察者模式
rxJava有四个基本概念，observable(可观察者、被观察者)、observer(观察者)、subscrib(订阅)。
observable和observer通过subscrib实现订阅的关系，在observable需要的时候，发送通知给observer。

和传统的观察者模式不同，rxJava的回调事件除了onNext事件意外（相当于onClick,Onevent事件），还定义了两个特殊的事件:onCompleted()、onError()

onCompleted():事件结束触发。rxJava不仅仅将事件单独处理，还会把他们作为一个队列，在没有onNext()事件触发的时候，通过调用omCompleted()作为结束

onError():事件队列异常触发。当事件队列发生异常的时候调研onError(),同时事件队列停止，不执行任何事件了。

在队列事件中,onCompleted()和onError()是相互对立的，两者正常只会有一个调用。


rxJava观察者模式，大致如下:

![rxJava观察者.jpg](http://upload-images.jianshu.io/upload_images/1387450-138b2383d88ec95b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#####rxJava基本实现
rxJava基本实现主要有三个要素

######(1)创建observer
observer是观察者，它觉得事件触发的时候将什么样的行为。RxJava中observer的借口实现方式如下:

		Observer<String>observer=new Observer<String>() {
					@Override
					public void onCompleted() {

					}

					@Override
					public void onError(Throwable e) {

					}

					@Override
					public void onNext(String s) {

					}
				};


除了observer借口之外，rxJava还实现了一个observer的抽象类subscriber。subscriber对observer进行了一些扩展，但是两者使用方法基本一样。

		//subscriber
				Subscriber<String>subscriber=new Subscriber<String>() {
					@Override
					public void onCompleted() {

					}

					@Override
					public void onError(Throwable e) {

					}

					@Override
					public void onNext(String s) {

					}
				};

不仅仅是使用基本一样，rxJava在subscrib建立观察的时候，也是讲observer转化为subscriber在使用的。
所以observer功能可能只是subscriber的一部分，但是也是能满足基本的使用

observer和subscriber的区别:


1.onStart():这个是有subscriber提供的方法，是在事件发生之前的调用，可以用于一些数据的初始化，例如数据的添加，删除，清空等。但是onStart()只能发生在subscrib所在的线程中，不能再独立的线程中使用，所以使用的时候需要注意，一些只能在ui线程中使用的，可能不能再onStart()中使用。


2.unSubscribe(),这个是subscriber提供的另外一个接口，通常在调用之前会先调用isUnSubscribe()判断一下是否已经取消观察了。unSubscriber()方法很重要，因为，observable在subscribe的过程中会持有observer对象，所以在合适的时候，需要将对象释放，防止内存泄漏。可以在onpause()或者onStop()的时候。


######(2.)创建observable

observable是被观察者，它觉得了什么时候触发事件，以及触发什么样的事件。

		//observable
				Observable<String>observable=Observable.create(new Observable.OnSubscribe<String>() {
					@Override
					public void call(Subscriber<? super String> subscriber) {

					}
				});

在onsubscribe中做一些事情，在某些特殊的事情的时候，可以钓鱼subscriber的onNext()、onCompleted()、onError()等事件，通知observer或者subscriber事件发生了。

######(3.)subscrib订阅
已经创建了observable和observer,下面只需要用subscrib将两者联系起来。

    observable.subscribe(observer);
    observable.subscribe(subscriber);


Observable.subscrib(subscriber)方法的核心代码如下:

		// 注意：这不是 subscribe() 的源码，而是将源码中与性能、兼容性、扩展性有关的代码剔除后的核心代码。
		// 如果需要看源码，可以去 RxJava 的 GitHub 仓库下载。
		public Subscription subscribe(Subscriber subscriber) {
			subscriber.onStart();
			onSubscribe.call(subscriber);
			return subscriber;
		}

可以看到subscrib()做了三个工作:

1. 调用onStart(),在开始之前的一些准备工作。
2. onSubscrible.call(subscriber),事件的逻辑开始运行，可以看出，observable不是在创建的时候开始发送消息，而是在subscrib订阅的时候开始。
3. 将observer对象返回，是为了方便unSubscrib()。




![事件发生流程.gif](http://upload-images.jianshu.io/upload_images/1387450-f434ec73f90d86f1.gif?imageMogr2/auto-orient/strip)


除了observable(observer)或者observable(subscriber)之外，rxJava还支持定义一些不完全的回调，如下


		Observable<String>observable=Observable.create(new Observable.OnSubscribe<String>() {
					@Override
					public void call(Subscriber<? super String> subscriber) {

					}
				});
				Action1<String>onNext=new Action1<String>() {
					@Override
					public void call(String s) {

					}
				};
				Action1<Throwable>onError=new Action1<Throwable>() {
					@Override
					public void call(Throwable s) {

					}
				};
				Action0 onCompleted=new Action0() {
					@Override
					public void call() {

					}
				};

				observable.subscribe(onNext);
				observable.subscribe(onNext,onError);
				observable.subscribe(onNext,onError,onCompleted);



######举例子

1. 打印字符串数组

		String[] array={"a","b","c"};

				Observable.from(array)
						.subscribe(new Action1<String>() {
							@Override
							public void call(String s) {
								Log.e("call==",s);
							}
						});



2.由id取得突破，并且显示

		//由id取出突破，并显示
				ImageView imageView = new ImageView(this);
				int[] drables = {R.mipmap.ic_launcher, R.mipmap.ic_launcher, R.mipmap.ic_launcher};

				Observable.create(new Observable.OnSubscribe<Drawable>() {
					@Override
					public void call(Subscriber<? super Drawable> subscriber) {
						try {
							for (int i = 0; i < drables.length; i++) {
								Drawable draw = getResources().getDrawable(drables[i]);
								subscriber.onNext(draw);
							}
							subscriber.onCompleted();
						} catch (Exception e) {
							e.printStackTrace();
							subscriber.onError(e);
						}

					}
				})
						.subscribe(new Action1<Drawable>() {
									   @Override
									   public void call(Drawable drawable) {
											imageView.setImageDrawable(drawable);
									   }
								   },
								new Action1<Throwable>() {
									@Override
									public void call(Throwable throwable) {
										Toast.makeText(UndefindSubscriberActivity.this,throwable.toString(),Toast.LENGTH_LONG);
									}
								},
								new Action0() {
									@Override
									public void call() {
										Toast.makeText(UndefindSubscriberActivity.this,"完成了",Toast.LENGTH_LONG);
									}
								});





######(4.)线程控制Scheduler 
在不指定线程的情况下，rxJava遵循线程不变的原则，即，在哪个线程调用subscrib(),就在哪个线程产生时间，在哪个线程产生时间，就在哪个线程消耗时间。
如果需要切换线程，就需要用到scheduler(调度器)。


1. scheduler(调度器),在rxJava中，scheduler相当于线程控制器，在rxJava中利用他们来表明每一段代码在什么样的线程中使用。rxJava中已经内置了几个scheduler，他们已经适合大部分的场景


* Schedulers.immediate():直接在当前线程中使用，即，不指定线程，默认就是这个值
*Schedulers.newThread():总是启用一个新的线程，并在新的线程中运行。
*Schedulers.IO():IO操作所使用的线程(读写文件、数据库操作、网络操作等)。io线程和newThread线程差不多，区别在于，io线程是一个没有上线的线程池，很好的维护了创建的线程，可以复用空闲的线程。所以正常情况下，io线程要比newThread线程要好一些。
*Scheduler.computation():计算用的线程，例如图像的计算。这个计算是CPU的密集型计算，所以很消耗CPU,使用的是固定的线程。
*AndroidSchedulers.mainThread():这是android专用的主线程。

有了这几个线程之后，就可以使用subscribOn()或者observeOn(),来指定运行在哪个线程了。subscribOn()是指
subscrib所在的线程，即observable.subscrib()的时候，调用者是observable,所以指定的是时间产生时候的线程。observeOn()指定的是observer的使用线程，即消费事件所在的线程。回忆一下，observable.subscrib()调用之后得到的是observer,所以observeOn()的调用者是observer。

    个人理解:在哪个线程中使用，主要是看调用者是谁，是observable就是事件产生的时候，是observer就是事件消费的时候。


举例说明:

		Observable.just(1,2,3,4)
						.subscribeOn(Schedulers.io())//指定 subscribe() 事件产生发生在 IO 线程
						.observeOn(AndroidSchedulers.mainThread())//指定 Subscriber 消费事件的回调发生在主线程
						.subscribe(new Action1<Integer>() {
							@Override
							public void call(Integer integer) {

							}
						});


#####(5.)变换

RxJava提供了对事件序列的变化功能，简单说就是在事件运行的过程中，对事件传递对象进行加工和变化成另外一个对象。这也是rxJava的核心功能之一。

1. API
先来看下map:先上代码,将string类型直接转化为bitmap对象

		Observable.just("/image/1.png")//这是一个图片途径
					.map(new Func1<String, Bitmap>() {
						@Override
						public Bitmap call(String path) {

							return getBitmapFromPath(path);//通过路径返回bitmap
						}
					})
				.subscribe(new Action1<Bitmap>() {
					@Override
					public void call(Bitmap bitmap) {
		//                imageView.setImageBitmap(bitmap);//得到bitmap之后的处理
					}
				});


这里有一个Func1的类，他和Action1很相似，都是rxJava的一个接口，用于包装一个带参数的方法。Func1和Action1的区别在于，Func1有返回的参数，Action1没有。

可以看到map()将参数string转化为bitmap,事件参数类型也由string转化为bitmap,这种直接变化对象参数并且返回的是很常见简单的应用。

map():事件对象的直接转换,是rxJava中常见的现象。示意图如下:


![map.jpg](http://upload-images.jianshu.io/upload_images/1387450-1d96231e251b4943.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


* flatMap():是一个很有用但是很难理解的变化。
例子:打印出所有学生的所有课程的名称

		Observable.from(students)
						.flatMap(new Func1<Student, Observable<Course>>() {
							@Override
							public Observable<Course> call(Student student) {
								//将student中的课程转化为一个Observable<Course>
								return Observable.from(student.getCourses());
							}
						})
				.subscribe(new Action1<Course>() {
					@Override
					public void call(Course course) {
						//将course名字打印
						Log.e("courseName",course.getCourseName());
					}
				});


可以看出flatMap和map有一个共同点，就是将传入的对象转化为另一个对象并返回，但是不同的是，flatMap返回的是一个observable对象，并且这个对象不是直接返回到subscriber的回调对象中。
flatMap的原理是这样的:
1. 传入事件对象创建一个observable对象
2. 并不发送这个对象，而是将这个对象激活，于是他开始发送事件
3. 每一个被创建出来的observable都会被汇入同一个observable中，而这个observable负责统一将事件交给subscriber回调。


flatMap()示意图如下:

![flatMap.jpg](http://upload-images.jianshu.io/upload_images/1387450-1379a2239e3475da.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


######线程控制scheduler(二)
在接触了map和flatMap的时候，就有些疑问了，是否可以多次切换线程呢，答案是，可以的
先上一段代码:

		Observable.just(1,2,3,4)
						.subscribeOn(Schedulers.io())//事件产生在io线程
						.observeOn(Schedulers.newThread())
						.map(new Func1<Integer, String>() {//处理线程有上面的observeOn()决定-是一个新开的线程
							@Override
							public String call(Integer integer) {
								return integer+"";
							}
						})
						.observeOn(Schedulers.io())
						.map(new Func1<String, Integer>() {//处理线程有上面的observeOn()决定-是在io线程中
							@Override
							public Integer call(String s) {
								return Integer.parseInt(s);
							}
						})
						.observeOn(AndroidSchedulers.mainThread())
						.subscribe(new Action1<Integer>() {//处理线程有上面的observeOn()决定-是在主线程中
							@Override
							public void call(Integer integer) {

							}
						});


好啦，rxJava基本到这里了，后面还有一些关于binding和rxBus的一些简单介绍。
最后提供这篇文章的[github地址](https://github.com/gujingjing/rxJavaSay.git)































