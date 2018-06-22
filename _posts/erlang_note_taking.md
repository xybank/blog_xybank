
| title | date | tags | categories |
| --- | ---  | --- | --- |
| erlang | 2018-06-22 10:55:28 | erlang | 李关浩 |

# erlang 随手记

erlang进程数量

```
(a@localhost)12>erlang:length(erlang:processes()).
42
(a@localhost)13>
```

erlang内存使用情况

```
(a@localhost)11> erlang:memory().
[{total,20257328},
 {processes,6017000},
 {processes_used,6016776},
 {system,14240328},
 {atom,331249},
 {atom_used,302818},
 {binary,98104},
 {code,8111152},
 {ets,438096}]
(a@localhost)12>
```

查看erlang进程的消息队列排名，内存排名，前5名

```
内存：
(a@localhost)25> lists:sublist(lists:reverse(lists:sort(lists:map(fun(P)->case erlang:process_info(P,memory) of {_,M} -> {M,P}; _->{0,P} end end,processes()))),5).
[{831280,<0.25.0>},
 {426448,<0.31.0>},
 {263856,<0.80.0>},
 {263792,<0.3.0>},
 {263752,<0.96.0>}]
(a@localhost)26>

消息队列：
(a@localhost)26> lists:sublist(lists:reverse(lists:sort(lists:map(fun(P)->case erlang:process_info(P,message_queue_len) of {_,MQ} -> {MQ,P}; _->{0,P} end end,processes()))),5).
[{0,<0.96.0>},
 {0,<0.80.0>},
 {0,<0.79.0>},
 {0,<0.78.0>},
 {0,<0.77.0>}]
(a@localhost)27>
```

用代码检测base64:encode的效率

```
(a@localhost)20> F2 = fun(C)-> A = crypto:rand_bytes(100*1024), T1=now(),lists:map(fun(_)-> catch base64:encode(A) end, lists:seq(1,C)),T2=now(),io:format("~p time use ~.2f ms ~n",[C,timer:now_diff(T2,T1)/1000]) end.
#Fun<erl_eval.6.90072148>
(a@localhost)22> F2(10).
10 time use 166.91 ms
ok
(a@localhost)23> F2(100).
100 time use 1612.59 ms
ok
(a@localhost)24> F2(1000).
1000 time use 16845.46 ms
ok
```

如何用kill取dump文件和如何查看dump文件

```
kill -s USR1 $pid

查看dump文件操作，进入erlang shell，输入：  
(bb@lghdeMacBook)1> webtool:start().
WebTool is available at http://localhost:8888/
Or  http://127.0.0.1:8888/
{ok,<0.42.0>}
(bb@lghdeMacBook)2>

根据url “http://127.0.0.1:8888/” 在ie上打开，录入dump文件即可查看dump文件。

```

对core文件的操作：

```
命令：dbx 应用程序 core
aix的应用程序默认路径： /opt/freeware/lib/erlang/erts-5.9/bin/beam.smp 
linux应用程序默认路径：/usr/lib64/erlang/erts-5.9/bin/beam.smp
更多详情参考文章： http://eric-gcm.iteye.com/blog/2149842
```

erlang节点相关操作：

```
启动widows的shell 输入 
erl -sname Nodename  （-setcookie COOKIENAME ）同时设置cookie名
PS：其中节点名必须满足erlang原子类型
（1）net_adm:ping(otherNode) pong表示成功，pang表示失败。
（2）auth:get_cookie()  获取cookie
（3）auth:set_cookie(node(),'myselfcookie')  修改cookie
（4）nodes()查看节点
（5）rpc:call(OtherNodeName, Mod, Func, [Arg1, Arg2, ...]) 在Node上执行一个远程调用。
 例：rpc:call('b@cib23219',ets,i,[]).
```

子父进程有趣之处：
```
parent() ->
    Pid = self(),
    % spawn(fun() -> child(self()) end).  **--- 1**
    spawn(fun() -> child(Pid) end).   **--- 2**

child(P) ->
    io:format("Parent Pid is = ~p,myself pid is = ~p~n",[P, self()]).

PS: 标注的代码段1和代码段2输出的结果不一样， 代码段1输出的是子进程的PID， 代码段2输出的是父进程的PID， 这是易错点。
```

**套接字 -----  socket - gen_tcp**

gen_tcp:connect(Address, Port, Options)或gen_tcp:listen(Port, Options)
erlang的套接字可以用三种方式打开，即通过Options选项配置{active， true|false|once}选项来实现。
1. {active, true}是主动型的（非阻塞)，建立一个套接字之后，当数据到达时会向控制进程发送{tcp, Socket, DataBin}消息；但是控制进程无法控制这些消息，即不能停止接收数据。由于控制进程无法控制消息的数量，服务器可能会因为大量的数据把数据缓冲区填满导致系统崩溃。
2. {active, false}是被动型的(阻塞)，即控制进程必须调用gen_tcp:revc(Socket,N)来接收数据，其中N表示从套接字中接收N个字节的数据，当N为0时则接收全部。
3. {active, once}混合型的(半阻塞)，建立一个套接字之后，控制进程可以接收第一条数据，但是如果想在接收下一条数据必须调用inet:setopts(Socket,[{active,once}])来激活，这个是上面两个的集合体，可以灵活使用。

**套接字 -----  socket - gen_udp**

gen_udp是一个无连接的协议，即如果客户端要向服务器发一个消息时，无须创建一个连接。直接用{ok, Socket} = gen_udp:open(Port, [Option]). 打开一个套接字，用ok = gen_udp:send(Socket, "localhost", Port, Bin). 发送数据即可。


...... 待续 ......


