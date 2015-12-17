# 【Android知识总结】（III）

----

> 作者： 王远涵  
时间： 2015-11-30  
关键词： `数据存储`　`持久化技术` `跨程序共享`

----

## 持久化技术
数据持久化就是指将内存中的瞬时数据保存到存储设备中，保证手机或电脑即使在关机的情况下，这些数据仍然不会丢失。保存在内存中的数据是瞬时的，而保存在存储设备中的数据是出于持久状态的，持久化技术，则是提供了一种机制可以让数据在瞬时状态和持久状态之间进行转换
Android系统中提供了三种方式用于简单地实现数据持久化功能： **文件存储**、**SharedPreference存储** 以及 **数据库存储**。除此之外，也可以将数据保存在手机的SD卡中，不过后者要相对复杂一些，且数据的安全性也无法得到保证

----

### 文件存储
#### 简述
不对存储的内容进行任何格式化处理，将所有数据都原封不动地保存到文件当中
#### 适用
存储一些简单的文本数据或二进制数据
#### 实现
- **存储** 利用`Context`类中提供的`openFileOutput`方法将数据存储到指定的文件中，文件名自定义，默认存储到`/data/data/<packge name>files/`目录下
- **读取** 类似的，`Context`类还提供了`openFileInput`方法用于从文件中读取数据，根据接收的文件名系统自动到`/data/data/<packge name>files/`目录下加载该文件

#### 扩展
如果需要利用文件存储的方式保存一些较为复杂的文本数据，就需要定义一套自己的格式规范，方便之后将数据从文件中重新解析出来

----

### SharedPreferences存储
#### 简述
SharedPreferences利用键值对的方式来存储数据，即每保存一条数据时，就需要给这条数据提供一个对应的键，这样在读取数据的时候就可以通过这个键把相应的值取出来
#### 适用
支持多种不同的数据类型存储，且存储及读取的数据类型保持不变
#### 实现
- **存储**
	1. 获取`SharedPreference`对象Android中主要提供了三种方法用于获取`SharedPreference`对象
		- `Context`类中的`getSharedPreferences()`方法
		- `Activity`类中的`getPreference()`方法
		- `PreferenceManager`类中的`getDefaultSharedPreference()`方法
	2. 调用`SharedPreference`对象的`edit()`方法来获取一个`SharedPreference.Editor`对象
	3. 向`SharedPreference.Editor`对象中添加数据，比如添加一个`Boolean`类型的数据就使用`putBoolean()`方法，添加一个字符串则使用`putString()`方法等
	4. 调用`commit()`方法提交添加的数据
- **读取**
	1. 同样需要通过上述方法首先获取`SharedPreference`对象
	2. 调用该`SharedPreference`对象相应的`get()`方法来获取相应数据类型的数据

----

### SQLite数据库存储
这部分内容在（II）中有详述

----

## 跨程序共享
上述使用持久化技术所保存的数据都只能够在当前应用程序中访问。尽管文件存储及SharedPreferences存储都提供了两个不同的操作模式用于供给其他应用程序访问当前应用的数据，但Android官方已不再推荐使用这种方式来实现跨程序数据共享的功能，因此均已在Android4.2中被废除

### Content Provider内容提供器
#### 简述
提供了一套完整的机制，允许一个程序访问另一个程序中的数据，同时还能通过选择只对某一部分数据进行共享保证了被访问数据的安全性
#### 适用
`Content Provider`的用法一般有两种，一是使用现有的`Content Provider`来读取和操作相应程序中的数据，另一种是创建自己的`Content Provider`给程序的数据提供外部访问接口
#### 实现
- **访问其他程序数据** 利用`Context`中的`getContentResolver()`方法获取`ContentResolver`类，该类中提供了一系列方法用于对数据进行`CRUD`操作。其中`insert()`方法用于添加数据，`update()`方法用于更新数据，`delete()`方法用于删除数据，`query()`方法用于查询数据，这一点类似于`SQLiteDatabase`
- **为程序数据的访问提供外部接口** 创建一个自己的`MyContentProvider`类继承`ContentProvider`类并重写其`onCreate()`、`insert()`、`update()`、`delete()`、`query()`、`getType()`等方法
