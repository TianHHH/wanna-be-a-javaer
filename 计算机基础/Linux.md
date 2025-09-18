# 命令相关问题汇总

Q: Linux 中查看具体进程的命令

**`ps -ef | grep <进程名>`**

- ps 命令用于显示进程状态; **<font style="color:blue">-e 参数表示显示系统中的所有用户的所有进程</font>**; -f 参数表示**以 full-format (完整格式)**显示进程信息

- 此外**使用 `ps aux | grep <进程名>` 也能用来查看具体进程**,但两者显示的参数有些许区别. -ef 偏向于显示进程的基础信息; **<font style="color:blue">aux 偏向于展示资源的使用情况</font>**

---

Q: 查看 Linux 主机 CPU 使用情况的命令

**top 命令**: 可以实时动态监控系统资源的使用情况, 包括 **<font style="color:blue">CPU 使用率、内存占用、负载情况等信息</font>**. 此外在 top 界面中, **按 1 键还能分列显示每个 CPU 核心的使用情况**

---

Q: 查看具体端口被谁占用的命令

最好用的是 **<font style="color:blue">`lsof -i:<端口号>`</font>**

**另外也可以用 `[netstat / ss] -tunlp | grep <端口号>`**, -tunlp 中的各参数分别表示: 显示 TCP; 显示 UDP; 数字形式显示地址和端口; 只显示监听中的服务; 显示进程 PID 和程序名

- **更推荐使用 ss 命令**, 是 netstat 的现代替代工具, 性能更好

---

