---
title: Kotlin 1.0 正式版发布啦
date: 2016-02-18 23:01:14
tags: Kotlin
categories: 技术
---

 

就在昨天，Android领域的Swift--Kotlin 1.0发布了，详细的新版本发布介绍请「阅读原文」查看官方发行说明。

### 何为Kotlin？

Kotlin 是一门实用的编程语言，可用于 JVM 和 Android 程序开发，Kotlin 结合了面向对象和函数式特性，致力于互操作性，安全，简洁和工具支持。

Kotlin 是一门通用的语言，只要能用 Java 的地方就能用 Kotlin，包含：服务器程序开发，移动应用开发（Android），桌面客户端程序开发。 Kotlin 支持所有主要的开发工具以及服务，比如：

	IntelliJ IDEA，Android Studio 和 Eclipse
	Maven, Gradle 和Ant
	Spring Boot（Kotlin 支持今天正式发布！）
	GitHub，Slack，甚至 Minecraft

<!--more-->
Kotlin 的主要特点之一是Java+Kotlin 混合工程的互操作性以及无缝兼容，使引入Kotlin的过程简单容易，并达成更少的重复性代码(boilerplate code)和更佳的类型安全(type-safety)。
Kotlin 还有一个扩展标准库(extensive standard library)能让日常工作变得简单顺畅，它能帮助保持低内存占用 (bytecode footprint)。当然，Kotlin 中自然可以使用 Java 库，反之亦然。

### 为何说Kotlin非常适合于Android？

基本上，这是因为Kotlin的所有特性都非常适合于Android生态圈。Kotlin的库非常小，我们在开发过程中不会引入额外的成本。其大小 相当于support-v4库，我们在很多项目中所使用的库都比Kotlin大。除此之外，Android Studio（官方的Android IDE）是基于IntelliJ构建的。这意味着我们的IDE对该语言提供了非常棒的支持。我们可以很快就配置好项目，并且使用熟悉的IDE进行开发。我 们可以继续使用Gradle以及IDE所提供的各种运行与调试特性。这与使用Java开发应用别无二致。归功于互操作性，我们可以在Kotlin代码中使 用Android SDK而不会遇到任何问题。实际上，部分SDK使用起来会变得更加简单，这是因为互操作性是非常智能的，比如说它可以将getters与setters映 射到Kotlin属性上，我们也可以以闭包的形式编写监听器。

### 如何在Android开发中使用Kotlin？

过程非常简单，只需按照下面的步骤来即可：

>从IDE plugins中下载Kotlin插件
在模块中创建Kotlin类
使用“Configure Kotlin in Project…”
开始编码

### Kotlin的一些特性

Kotlin拥有大量非常打动人心的特性，这里无法一一进行介绍，不过我们来看一下其中最为重要的一些。

#### Null安全

如前所述，Kotlin是null安全的。如果一个类型可能为null，那么我们就需要在类型后面加上一个?。这样，每次在使用该类型的变量时，我们都需要进行null检查。比如说，如下代码将无法编译通过：

	var artist: Artist? = null?
	artist.print()

第2行会显示一个错误，因为没有对变量进行null检查。我们可以这样做：

	if (artist != null) {
	?    artist.print()?
	}

这展示了Kotlin另一个出色的特性：智能类型转换。如果检查了变量的类型，那就无需在检查作用域中对其进行类型转换。这样，我们现在就可以在 if中将artist作为Artist类型的变量了。这对于其他检查也是适用的。还有一种更简单的方式来检查null，即在调用对象的函数前使用?。甚至 还可以通过Elvis运算符?提供另外一种做法：

	val name = artist?.name ?: ""

#### 数据类
在Java中，如果想要创建数据类或是POJO类（只保存了一些状态的类），我们需要创建一个拥有大量字段、getters与setters的类，也许还要提供toString与equals方法：

	public class Artist {
		private long id;
		private String name;
		private String url;
		private String mbid;

		public long getId() {
			return id;
		}

		public void setId(long id) {
			this.id = id;
		}

		public String getName() {
			return name;
		}

		public void setName(String name) {
			this.name = name;
		}

		public String getUrl() {
			return url;
		}

		public void setUrl(String url) {
			this.url = url;
		}

		public String getMbid() {
			return mbid;
		}

		public void setMbid(String mbid) {
			this.mbid = mbid;
		}

		@Override
		public String toString() {
			return "Artist{" +
					"id=" + id +
					", name='" + name + '\'' +
					", url='" + url + '\'' +
					", mbid='" + mbid + '\'' +
					'}';
		}
	}

在Kotlin中，上述代码可以写成下面这样：

	data class Artist (?
		var id: Long,
		var name: String,
		var url: String,
		var mbid: String)

Kotlin使用属性而非字段。基本上，属性就是字段加上其getter与setter。

#### 互操作
Kotlin提供了一些非常棒的互操作特性，这对于Android开发帮助非常大。其中之一就是拥有单个方法的接口与lambda表达式之间的映射。这样，下面这个单击监听器：

	view.setOnClickListener(object : View.OnClickListener {
		override fun onClick(v: View) {
			toast("Click")?
		}
	?})

可以写成这样：

	view.setOnClickListener { toast("Click") }

此外，getters与setters都会自动映射到属性上。这并不会造成性能上的损失，因为字节码实际上只是调用原来的getters与setters。如下代码所示：

	supportActionBar.title = title
	textView.text = title
	contactsList.adapter = ContactsAdapter()

####Lambda表达式

Lambda表达式会在极大程度上精简代码，不过重要的是借助于Lambda表达式，我们可以做到之前无法实现或是实现起来非常麻烦的事情。借助于 Lambda表达式，我们可以以一种更加函数式的方式来思考问题。Lambda表达式其实就是一种指定类型，并且该类型定义了一个函数的方式。比如说，我 们可以像下面这样定义一个变量：

	val listener: (View) -> Boolean

该变量可以声明一个函数，它接收一个view并返回这个函数。我们需要通过闭包的方式来定义函数的行为：

	val listener = { view: View -> view is TextView }

上面这个函数会接收一个View，如果该view是TextView的实例，那么它就会返回true。由于编译器可以推断出类型，因此我们无需指定。还可以更加明确一些：

	val listener: (View) -> Boolean = { view -> view is TextView }

借助于Lambda表达式，我们可以抛弃回调接口的使用。只需设置希望后面会被调用的函数即可：

	fun asyncOperation(value: Int, callback: (Boolean) -> Unit) {
		...
		callback(true)?
	}

	asyncOperation(5) { result -> println("result: $result") }

还有一种更加简洁的方式，如果函数只接收一个参数，那就可以使用保留字it：

	asyncOperation(5) { println("result: $it") }

#### Anko

Anko是Kotlin团队开发的一个库，旨在简化Android开发。其主要目标在于提供一个DSL，使用Kotlin代码来声明视图：

	verticalLayout {
		val name = editText()
		button("Say Hello") {
			onClick { toast("Hello, ${name.text}!") }
		}
	}

它还提供了其他一些很有用的特性。比如说，导航到其他Activity：

	startActivity("id" to res.id, "name" to res.name)

### 总结

如你所见，Kotlin在很多方面都简化了Android的开发工作。它会提升你的生产力，并且可以通过非常不同且更加简单的方式来解决一些常见的问题，你开始学习了么？

### 参考资料：
	
    1. http://www.infoq.com/cn/news/2016/01/kotlin-android?utm_campaign=infoq_content&utm_source=infoq&utm_medium=feed&utm_term=global
    2. http://www.oschina.net/news/70734/kotlin-1-0-final
