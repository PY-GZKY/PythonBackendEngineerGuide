# 如何优化后端服务的性能

## cpu密集型计算的场景有哪些?

CPU密集型计算的场景是指那些需要大量CPU计算资源来处理任务的应用场景。
这类计算通常涉及到复杂的数学和逻辑运算，而与磁盘I/O、网络通信或用户交互相比，CPU的运算能力是这些任务的瓶颈。

以下是一些典型的CPU密集型计算场景：

- 科学计算：物理模拟、气候模型、分子建模、天体物理学等领域需要大量的数值计算。
- 数据分析：大数据处理、数据挖掘、机器学习模型训练等需要进行复杂的数据处理和分析。
- 图形渲染：3D图形渲染和视频编码通常需要大量的CPU资源来计算光线追踪、纹理映射等。
- 游戏：现代计算机游戏中的物理引擎和AI计算通常需要大量的CPU资源。
- Web服务器：在高流量的情况下，动态内容生成（如模板渲染）可能会导致CPU资源成为瓶颈。
- 数据库服务器：在处理复杂查询或大量并发请求时，数据库服务器可能会遇到CPU瓶颈。

更具体的后端应用下，我们通常会涉及到大量的数据处理、计算和逻辑运算，这些任务可能会成为CPU密集型任务的瓶颈。

比如：

- 在处理一些的用户请求时，会涉及到大量的模板渲染和逻辑计算
- 对于服务器，在高并发的场景下，系统cpu开始繁忙的处理用户请求（上下文切换，同步和锁竞争等）
- 对于数据库，处理复杂的查询和大量的并发请求

这些操作会消耗大量的CPU资源。

## IO密集型计算的场景有哪些?

I/O密集型计算的场景是指那些需要大量I/O资源（如磁盘I/O、网络I/O）来处理任务的应用场景。

- 文件处理：大量的文件读写操作，如文件复制、文件压缩、文件解压等。
- 数据库操作：大量的数据库读写操作，如查询、插入、更新、删除等。
- 网络通信：大量的网络通信操作，如HTTP请求、TCP连接、UDP数据包等。

更具体的后端应用下，我们通常会涉及到大量的磁盘操作（如文件读写）或网络通信（如数据库访问、HTTP请求）。

这些任务的瓶颈主要是因为I/O操作的速度远远低于CPU的处理速度，导致CPU在等待I/O操作完成时出现空闲，从而影响了应用的整体性能。以下是I/O密集型任务可能遇到的一些主要瓶颈：

- 磁盘速度：磁盘的读写速度是I/O操作中常见的瓶颈之一。尤其是当使用传统的机械硬盘（HDD）时，其转速和寻道时间限制了数据的读写速度。
- 网络延迟：网络请求涉及到数据在网络中的传输，这通常受到网络带宽、延迟和丢包率的影响。网络延迟会直接影响到应用响应的速度。
- 操作系统的I/O调度：操作系统的I/O调度器对I/O请求的处理效率也会影响I/O密集型任务的性能。不同的调度算法和设置可能会对I/O性能产生显著影响。
- 数据库性能：如果应用程序频繁地与数据库交互，数据库的性能可能成为瓶颈。这可能包括数据库的查询优化、索引、并发连接数、内存及其I/O子系统的性能。
- 同步I/O操作：如果应用程序使用同步I/O，那么每个I/O操作必须在进行下一个计算之前完成，这会导致CPU等待I/O操作的完成，从而造成资源的浪费。
- 资源争用：在多任务环境中，多个进程或线程可能会同时进行I/O操作，导致资源争用，进而影响I/O性能。

那我们应该如何优化CPU/IO密集型任务的性能呢?

## 为什么多核心 CPU 会提高性能？

多核心 CPU 是指在一个 CPU 芯片上集成了多个 CPU 核心，**每个核心都可以独立运行程序**。

多核心可以提高计算机的性能，主要有以下几个原因：

- 并行处理：同时处理多个任务，提高计算机的并行处理能力。
- 负载均衡：将任务分配到不同的核心上执行，从而减少每个核心的负载，提高计算机的负载均衡能力。
- 提高响应速度：同时处理多个任务，提高计算机的响应速度。

## 如果应用是 I/O 密集型操作，如何提高响应的速度？

假如数据库查询是 I/O 密集型操作，那么使用多进程时可能会遭遇性能瓶颈，因为每个进程在等待 I/O 时不会主动释放 CPU 资源，而是阻塞在 I/O 操作上，导致进程间的资源竞争增加。

这个时候多进程并不能提高性能，我们应该选择多线程或者异步方法来处理 I/O 密集型操作。但需注意线程安全以及锁的使用。

## 如何对 FastAPI 框架进行性能优化?

### 1. 如何在合适的地方使用异步方法?

在 FastAPI 中，可以使用异步方法来提高性能。比如在处理请求时，特别是一些 I/O 密集型的操作，可以使用异步方法来处理，这样可以提高并发处理能力。

那什么时候可以使用异步方法呢？

### 2. 使用多进程部署充分利用多核 CPU

uvicorn 默认使用单进程处理请求，可以通过设置 `--workers` 参数来启动多个进程，充分利用多核 CPU。

```shell
uvicorn main:app --workers 4
```

## FastAPI 启动1个进程和4个进程有什么区别？

在 FastAPI 中，使用 uvicorn 来启动应用时，如果你选择了多进程（--workers 参数），它将启动多个独立的进程来处理请求。
这对于 I/O 密集型任务来说并不总是最优的，因为每个进程都有自己的内存空间，并且缺乏 asyncio 的异步能力。

在数据库查询、接口方法的逻辑计算并没有那么复杂时，比如只是简单的查询数据库并返回结果，设想一下：

假如你发现单进程接口的响应速度是 50ms，而使用4个进程却需要100ms。你会奇怪为什么多进程的性能反而下降了?

这其实是因为多进程的开销，每个进程都需要占用一定的内存空间，而且进程间的通信也会带来一定的开销。因此，对于一些简单的任务，多进程并不一定能提高性能。

反观单进程，对于 FastAPI这种异步web框架来说，它只需要占用一份内存空间，在线程中的任务开销也比进程间的开销要小得多，因此在一些简单的任务中，单进程可能会比多进程更快。

## 如何优化数据库性能？

### 如何查看正在连接的数据库用户？

在 MySQL 中，可以通过以下命令查看当前正在连接的数据库用户：

```shell
SHOW FULL PROCESSLIST;
```

### 如何设置  MySQL 的 wait_timeout?

`wait_timeout` 是 MySQL 中的一个系统变量，用于设置客户端连接的超时时间，即连接在空闲一段时间后会自动断开。

这个变量的默认值是 `28800` 秒（8 小时），也就是说，如果一个客户端连接在 8 小时内没有任何活动，那么 MySQL 会自动断开这个连接。

但是有时候我们不希望数据库在闲置 8 小时之后才断开连接，如果我的后台服务并发性较高，那么 8 小时的时间可能会导致数据库连接数过多，从而影响数据库的性能。

在 MySQL 中，可以通过以下方式设置 `wait_timeout` 变量：

```shell
SHOW SESSION VARIABLES LIKE 'wait_timeout';
```

然后，你需要找到启动 MySQL 的配置文件，通常是 `my.cnf` 或 `my.ini`，在这个文件中修改或添加以下配置：

```shell
[mysqld]
wait_timeout = 14400
```

这里的 `14400` 是你希望设置的超时时间，单位是秒。修改完配置文件后，重启 MySQL 服务，新的 `wait_timeout` 设置就会生效。

设置 `wait_timeout` 的目的是为了优化数据库的性能和资源利用，避免因为连接数过多而导致数据库性能下降。

设置过小的 `wait_timeout` 可能会导致一些问题，比如客户端连接频繁断开、连接池无法正常工作等，因此需要根据实际情况来合理设置这个值。

### 什么是数据库索引？

数据库索引可以提高查询的效率，减少数据库系统的负载。当我们查询数据库时，数据库系统会首先检查索引，然后根据索引的信息来定位数据，从而减少查询的时间。

常见的索引类型有单列索引、多列索引（组合索引）、唯一索引、主键索引等。

### 如何创建数据库索引？

在创建数据库表时，我们可以为表的某些列创建索引。例如，我们可以为用户表的用户名列创建索引，以提高查询用户信息的效率。

例如有一个用户表的表结构如下：

```sql
CREATE TABLE users
(
    id         INT PRIMARY KEY,
    username   VARCHAR(255),
    email      VARCHAR(255),
    password   VARCHAR(255),
    created_at TIMESTAMP
);
```

我们假设这个表有100万条记录，我们要查询用户名为`test123`的用户信息，如果没有索引，数据库系统会逐条扫描表中的记录，
直到找到用户名为`admin`的记录。这样的查询效率非常低。

#### 创建单列索引

```sql
CREATE INDEX idx_username ON users (username);
```

上面的语句创建了一个名为`idx_username`的索引，该索引是对`users`表的`username`列进行索引，这是一个单列索引。

#### 创建组合索引

如果我们想要查询 username 和 email 列的组合信息，比如查询用户名为`test123`且邮箱为`test123@qq.com`的用户信息，我们可以创建一个组合索引：

```sql
CREATE INDEX idx_username_email ON users (username, email);
```

这是一个多列索引，使用场景是查询 username 和 email 列的组合信息。它和分别创建单列索引的区别是：

- 单列索引只能用于查询单列信息，而多列索引可以用于查询多列信息
- 多列索引的查询效率高于单列索引，因为多列索引可以减少数据库系统的扫描次数

#### 创建全文索引

还有一种情况是模糊查询，具体的sql执行过程可能是：

```text
SELECT * FROM users WHERE username LIKE '%Love%';
``` 

（为了演示用了 * 号，生产环境不要使用 * 号）

为了提高模糊查询的效率，我们可以为`username`列创建全文索引：

```sql
CREATE
FULLTEXT INDEX idx_users ON users (username);
```

如何使用全文索引：

```sql
SELECT *
FROM users
WHERE MATCH (username) AGAINST('Love');
```

- `MATCH` 关键字用于指定要匹配的列
- `AGAINST` 关键字用于指定要匹配的字符串

全文索引可以提高模糊查询的效率，但是需要注意的是，全文索引只能用于 MyISAM 和 InnoDB 引擎，且只能用于 CHAR、VARCHAR、TEXT 类型的列。


