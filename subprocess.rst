python 创建子进程模块subprocess
===============================

subprocess的目的就是启动一个新的进程并且与之通信。

subprocess模块中只定义了一个类: Popen。可以使用Popen来创建进程，并与进程进行复杂的交互。它的构造函数如下：
::

    subprocess.Popen(args, bufsize=0, executable=None, stdin=None, stdout=None, stderr=None, preexec_fn=None, close_fds=False, shell=False, cwd=None, env=None, universal_newlines=False, startupinfo=None, creationflags=0)

参数args可以是字符串或者序列类型（如：list，元组），用于指定进程的可执行文件及其参数。如果是序列类型，第一个元素通常是可执行文件的路径。我们也可以显式的使用executeable参数来指定可执行文件的路径。

参数stdin, stdout, stderr分别表示程序的标准输入、输出、错误句柄。他们可以是PIPE，文件描述符或文件对象，也可以设置为None，表示从父进程继承。

如果参数shell设为true，程序将通过shell来执行。

参数env是字典类型，用于指定子进程的环境变量。如果env = None，子进程的环境变量将从父进程中继承。

subprocess.PIPE

　　在创建Popen对象时，subprocess.PIPE可以初始化stdin, stdout或stderr参数。表示与子进程通信的标准流。

subprocess.STDOUT

　　创建Popen对象时，用于初始化stderr参数，表示将错误通过标准输出流输出。

Popen的方法：

Popen.poll()

　　用于检查子进程是否已经结束。设置并返回returncode属性。

Popen.wait()

　　等待子进程结束。设置并返回returncode属性。

Popen.communicate(input=None)

　　与子进程进行交互。向stdin发送数据，或从stdout和stderr中读取数据。可选参数input指定发送到子进程的参数。Communicate()返回一个元组：(stdoutdata, stderrdata)。注意：如果希望通过进程的stdin向其发送数据，在创建Popen对象的时候，参数stdin必须被设置为PIPE。同样，如果希望从stdout和stderr获取数据，必须将stdout和stderr设置为PIPE。

Popen.send_signal(signal)

　　向子进程发送信号。

Popen.terminate()

　　停止(stop)子进程。在windows平台下，该方法将调用Windows API TerminateProcess（）来结束子进程。

Popen.kill()

　　杀死子进程。

Popen.stdin，Popen.stdout ，Popen.stderr ，官方文档上这么说：

stdin, stdout and stderr specify the executed programs’ standard input, standard output and standard error file handles, respectively. Valid values are PIPE, an existing file descriptor (a positive integer), an existing file object, and None.

Popen.pid
---------

　　获取子进程的进程ID。

Popen.returncode

　　获取进程的返回值。如果进程还没有结束，返回None。
　　
  ::
　
　   proc = subprocess.Popen("dir", shell=True)
　   retcode = proc.wait()
　   
　   shell参数根据你要执行的命令的情况来决定，上面是dir命令，就一定要shell=True了，p.wait()可以得到命令的返回值。

如果上面写成a=p.wait()，a就是returncode。那么输出a的话，有可能就是0【表示执行成功】。
　　
进程间的通信
============

获得进程的输出::

    p=subprocess.Popen("ls", shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    (stdoutput,erroutput) = p.communicate()  
  
p.communicate会一直等到进程退出，并将标准输出和标准错误输出返回，这样就可以得到子进程的输出了。
  
上面，标准输出和标准错误输出是分开的，也可以合并起来，只需要将stderr参数设置为subprocess.STDOUT就可以了，这样子::

    p=subprocess.Popen("dir", shell=True, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)  
    (stdoutput,erroutput) = p.<span>commu</span>nicate()  

如果你想一行行处理子进程的输出，也没有问题::

    p=subprocess.Popen("dir", shell=True, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)  
    while True:  
        buff = p.stdout.readline()  
        if buff == '' and p.poll() != None:  
            break  

死锁

但是如果你使用了管道，而又不去处理管道的输出，那么小心点，如果子进程输出数据过多，死锁就会发生了，比如下面的用法::

    p=subprocess.Popen("longprint", shell=True, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)  
    p.wait()  

longprint是一个假想的有大量输出的进程，那么在我的xp, Python2.5的环境下，当输出达到4096时，死锁就发生了。当然，如果我们用p.stdout.readline或者p.communicate去清理输出，那么无论输出多少，死锁都是不会发生的。或者我们不使用管道，比如不做重定向，或者重定向到文件，也都是可以避免死锁的。

subprocess还可以连接起来多个命令来执行。

在shell中我们知道，想要连接多个命令可以使用管道。

在subprocess中，可以使用上一个命令执行的输出结果作为下一次执行的输入。例子如下::

    proce1 = subprocess.Popen("cat filename", shell=True, stdout=subprocess.PIPE)
    proce2 = subprocess.Popen("head 2", shell=True, stdin=proce1.stdout, stdout=subprocess.PIPE)

例子中，p2使用了第一次执行命令的结果p1的stdout作为输入数据，然后执行tail命令。

下面是一个更大的例子。用来ping一系列的ip地址，并输出是否这些地址的主机是alive的。代码参考了python unix linux 系统管理指南。::

    #!/usr/bin/env python  
      
    from threading import Thread  
    import subprocess  
    from Queue import Queue  
      
    num_threads=3  
    ips=['127.0.0.1','116.56.148.187']  
    q=Queue()  
    def pingme(i,queue):  
        while True:  
            ip=queue.get()  
            print 'Thread %s pinging %s' %(i,ip)  
            ret=subprocess.call('ping -c 1 %s' % ip,shell=True,stdout=open('/dev/null','w'),stderr=subprocess.STDOUT)  
            if ret==0:  
                print '%s is alive!' %ip  
            elif ret==1:  
                print '%s is down...'%ip  
            queue.task_done()  
      
    #start num_threads threads  
    for i in range(num_threads):  
        t=Thread(target=pingme,args=(i,q))  
        t.setDaemon(True)  
        t.start()  
      
    for ip in ips:  
        q.put(ip)  
    print 'main thread waiting...'  
    q.join();print 'Done'  

在上面代码中使用subprocess的主要好处是，使用多个线程来执行ping命令会节省大量时间。

假设说我们用一个线程来处理，那么每个 ping都要等待前一个结束之后再ping其他地址。那么如果有100个地址，一共需要的时间=100*平均时间。

如果使用多个线程，那么最长执行时间的线程就是整个程序运行的总时间。时间比单个线程节省多了.

这里要注意一下Queue模块的学习。

pingme函数的执行是这样的：

启动的线程会去执行pingme函数。

pingme函数会检测队列中是否有元素。如果有的话，则取出并执行ping命令。

这个队列是多个线程共享的。所以这里我们不使用列表。(假设在这里我们使用列表，那么需要我们自己来进行同步控制。Queue本身已经通过信号量做了同步控制，节省了我们自己做同步控制的工作=。=)

代码中q的join函数是阻塞当前线程。下面是e文注释::

　Queue.join()

　　Blocks until all items in the queue have been gotten and processed(task_done()).


学习Processing模块的时候，遇到了进程的join函数。进程的join函数意思说，等待进程运行结束。与这里的Queue的join有异曲同工之妙啊。processing模块学习的文章在这里
