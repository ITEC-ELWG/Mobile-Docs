# iOS中数据库的最佳实践
> 创建时间：2015-12-1 
> 作者：曾祥意
> 关键词：SQlite，数据库  

----------


###IOS中的数据存储

iOS中数据持久化是开发中经常遇到的问题，而数据库持久化的方法也有很多:

- **SQLitePersistentObject**: SQLitePersistentObject只支持Cocoa中基础的数据类型（如：UIImage、NSString、NSNumber、NSData等等），但是对于NSArray，NSSet，NSDictionary等相关集合类型是不支持的。使用时，定义的模型继承于SQLitePersistentObject，就可以通过它定义的私有方法对数据库进行模型的增删改查。
**注意**：SQLitePersistentObject所有的原始方法都是线程同步的
- **Core Data**：Core Data是框架提供的是对象-关系映射功能，Core Data可以将OC对象转换成数据，并保存到数据库文件中，Core Data还还可以将保存后的数据还原成OC对象。Core Data是对SQlite的封装，大大减少了Model层的代码量，还可以可视化创建数据库结构。
- **SQlite**：SQLite是一款轻型的关系型数据库，是原生API，没有面向对象接口，在使用时用纯c语言实现
- **FMDB**：FMDB以OC的方式封装了SQLite的C语言API，提供了多线程安全的数据库操作方法，有效地防止数据混乱，相比于SQlite，FMDB可以省略很多代码和相关参数。FMDB中有三个主要的类：FMDatabase，FMResultSet，FMDatabaseQueue

###SQlite优缺点

####优点
- **轻量级**：占用资源非常低，一般只需要添加一个lib，不用启动其他系统进程，在嵌入式设备中，可能只需要几百k的内存就够了，而且速度很快，处理速度比Mysql、PostgreSQL都还快。
- **配置简单**：不依赖于第三方软件，不用安装配置服务程序。
- **可移植性强**：除了可以支持主流的操作系统，SQlite还支持很多其他操作系统以及一些嵌入式系统。
- **无数据类型**：采用无数据类型，所以可以保存任何类型的数据，SQLite采用的是动态数据类型，会根据存入值自动判断。SQLite具有以下五种数据类型：NULL(空值), INTEGER(带符号的整型), REAL(浮点数字), TEXT(字符串文本), BLOB(二进制对象)。
- **独立性**：sqlite 使用标准C 语言实现，它只需要非常少的系统或外部库的支撑，这使得它非常易于移植进嵌入式设备，同样这也使得它能应用于更广泛的不同配置的软件环境。sqlite 使用一个VFS（虚拟文件系统）层完成和磁盘的交互，而在不同系统中完成这个交互层是非常简单的工作。
- **非服务式**：多数SQL 数据库是以服务的形式实现的，这要求客户程序必须通过某种中间接口来连接数据库。与此相反，SQLite 直接访问数据库文件本身，没有任何中间媒介。
-  **开放性**：源代码开放, 代码有较好的注释。

####缺点
- **小规模数据**：SQlite轻量级的特点也使得它应用目标是较小的数据量，大规模数据情况下，可以使用MYSQL或ORACLE。
- **原生SQlite**：原生API比较麻烦，并且线程不安全。封装后的FMDB操作方便、简单、代码优雅，易于维护，使用FMDatabaseQueue来保证线程安全，一个FMDatabaseQueue的对象可以在多线程中共享使用。
- **并发控制**：SQLite只支持库级锁，意味着同时只能允许一个写操作，这可能导致其他读写进程阻塞或者出错。
- **存储模型**：SQLite的文件物理上被划分成相同大小的块；逻辑上划分成一些B-Tree——每个表对应一个B-Tree。而没有像Oracle，或者InnoDB对数据块进行复杂的逻辑组织，这种按需分配数据块的做法会影响磁盘的读写性能。
- **恢复技术**：DBMS的常用恢复技术有影子分页技术与基于日志的技术，SQLite的恢复技术是影子分页技术，影子分页的缺点就是事务提交时要输出多个块，这使得提交的开销很大，而且以块为单位，很难应用到允许多个事务并发执行的情况。

----------


###数据库最佳实践

####数据库设计要点

数据库设计是整个程序的重点之一，为了支持相关程序运行，最佳的数据库设计往往不可能一蹴而就，只能反复探寻并逐步求精，这是一个复杂的过程，也是规划和结构化数据库中的数据对象以及这些数据对象之间关系的过程。下面给出了20个数据库设计最佳实践:

>1. 使用明确、统一的标明和列名，例如 School, SchoolCourse, CourceID。
>2. 数据表名使用单数而不是复数，例如 StudentCourse，而不是StudentCourses。
>3. 数据表名不要使用空格。
>4. 数据表名不要使用不必要的前缀或者后缀，例如使用School，而不是TblSchool，或者SchoolTable等等。
>5. 数据库中的密码要加密，到应用中再解密。 （其实就是散列存储、单向加密）
>6. 使用整数作为ID字段，也许现在没有这个必要，但是将来需要，例如关联表，索引等等。
>7. 使用整数字段做索引，否则会带来很大的性能问题 。
>8. 使用 bit 作为布尔字段，使用整数或者varcha是浪费。同时，这类字段应该以“Is”开头。
>9. 要经过认证才能访问数据库，不要给每一个用户管理员权限。
>10. 尽量避免使用“select *”，而使用“select [required_column_list]”以获得更好的性能。
>11. 假如程序代码比较复杂，使用ORM框架，例如hibernate，iBatis。ORM框架的性能问题可以通过详细的配置去解决。
>12. 分割不常使用的数据表到不同的物理存储以获得更好的性能。
>13. 对于关键数据库，使用安全备份系统，例如集群，同步等等。
>14. 使用外键，非空等限制来保证数据的完整性，不要把所有的东西都扔给程序。
>15. 缺乏数据库文档是致命的。你应该为你的数据库设计写文档，包括触发器、存储过程和其他脚本。
>16. 对于经常使用的查询和大型数据表，要使用索引。数据分析工具可以帮助你决定如何建立索引。
>17. 数据库服务器和网页服务器应该放在不同的机器上。这回提高安全性，并减轻CPU压力。
>18. Image和blob字段不应该定义在常用的数据表中，否则会影响性能。
>19. 范式（Normalization）要按照要求使用以提高性能。Normalization做的不够会导致数据冗余，而过度Normalization 会导致太多的join和数据表，这两种情况都会影响性能。
>20. 多花点时间在数据库设计上，否则你将来会付出加倍的时间来偿还。


>[英文原址](http://www.javacodegeeks.com/2012/02/20-database-design-best-practices.html)

####FMDB的最佳实践
- **close方法**：通常打开完操作操作后，需要调用 close 方法来关闭数据库，节省资源
- **线程安全**：如果我们的 app 需要多线程操作数据库，那么就需要使用 FMDatabaseQueue 来保证线程安全了。 切记不能在多个线程中共同一个 FMDatabase 对象并且在多个线程中同时使用，这个类本身不是线程安全的，这样使用会造成数据混乱等问题。另外，在闭包中操作数据库，而不直接参与 FMDatabase 的管理。
- **递归锁**：为避免死锁的情况，可以使用NSRecursiveLock。它可以允许同一线程多次加锁，而不会造成死锁。递归锁会跟踪它被lock的次数。每次成功的lock都必须平衡调用unlock操作。只有所有达到这种平衡，锁最后才能被释放，以供其它线程使用。
- **封装数据库管理**：在开发中，要将数据库管理和业务逻辑分开，数据库的表的创建以及队列操作可以放在一个单独的文件，在实现业务逻辑时去调用这个文件。
