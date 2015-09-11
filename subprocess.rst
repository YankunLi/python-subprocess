subprocess的目的就是启动一个新的进程并且与之通信。

subprocess模块中只定义了一个类: Popen。可以使用Popen来创建进程，并与进程进行复杂的交互。它的构造函数如下：

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
　　
进程间的通信
============
