# 并行算法设计与程序实践-笔记
> 写在前面:
> 这是 `SZU` **并行算法设计与实践课程**笔记，下课的时候有好几个同学问我要这个笔记，现在 `ta` 来了。
> 如果觉得有帮助，请帮忙右上角点个 `star` ，感谢:blush:~


## 一、日常使用

预习：咱们的试验箱分为前端处理器 `1` 个，加上后端处理器(节点) `4` 个，每个后端处理器都有 `4` 个核心。也就是说，咱们的试验箱一共有 `16` 个核心。

#### 1、并发方式：mpi、openmp

-  `mpi` ：并发系统手动挡，本实验使用

- ` openmp` ：并发系统自动挡，本实验不适用

- `mpi` 和 `openmp` 的对比 ：

  |   指标   |  MPI   | openMP |
  | :-----: | :------------: | :------------: |
  |   粒度   |  进程  |  线程  |
  |   存储   | 分布式 | 共享式 |
  | 数据分布 |  显式  |  隐式  |
  | 可扩展性 |  较好  |  较差  |


#### 2、登录高性能核心

- `minicom` ：主要用于修改 ` IP`  地址，修改密码等。（`  已失效  `）

  ```
  进入方式：首先选择进入的核心编号 `gpio number` ，再输入minicom。
  退出方式：ctrl+A  ->  x  ->  回到前端
  ```

-  `SSH` 登录：本实验日常使用方式，例：

  ```
  ssh sdbox@192.168.100.1         //ssh登录
  ```

## 二、实验运行
>#### 1、单实验箱运行步骤(基本完善）
>
>**下面以实验 `13-1` 为例，展示具体过程**：
>
>- 在前端状态下，利用 ` scp ` 工具把代码上传到待使用的核心，例如: 
>
>  ```
>   scp /path/file  User@host:/LocalPathorFile      //这是标准语句格式
>   scp rank_sort.c sdbox@192.168.100.1:/root         //这是实际用例。上传到主目录的root文件夹
>  ```
>
>- 此时进入高性能核心，如有必要，请修改 `mpd.hosts` 文件
>
>  ```
>  ssh root@192.168.100.1          // 进入方法1
>  minicom                         // 进入方法2
>  ```
>
>- 编译程序:（具体程序文件夹的 `README` 有具体编译参数）
>
>  ```
>  mpicc rank_sort.c –o rank
>  ```
>
>- 设置运行程序的**核心数量**：
>
>  ```
>   mpdboot -n 1 -f mpd.hosts       //这里使用 1 个处理器（节点）
>   mpdtrace                        //若启用成功，此命令会输出启用节点的>列表
>  ```
>- 运行程序:（可在程序文件夹的 `README` 查看具体用法）
>
>  ```
>  mpirun –np 4 ./rank              // 这里使用 4 个核心(一个处理器有4个核心)
>  ```


#### 2、多盒子串联运行步骤（待完善...）
-  `ifconfig`  查看当前  `IP` 

-  `gpio 1`  启用  `1` 号多处理器

- 设置  ` IP`  ：

  ```
  ifconfig eth1 192.168.100.38 netmask 255.255.255.0
  ```

- 重启系统，倒数时按任意键暂停，此时修改启动 ` IP`  ：

  ```
  （PMON串口调试）：setmac2：~~~~~~~~~~~~：0x（ x 为想要的编号）
  set nfsip 192.168.0.0x（ x 为想要的编号）（默认 IP：192.168.0.250）
  ```

- 正常重启

-  `ping`  另一个箱子的处理器  `IP`  （成功即连上）

  


## 三、问题处理
#### 1、更改核心的登录密码

- 有的核心的密码被改过，上传文件和登录核心都会失败，此时需要更改核心登陆密码。具体代码如下：

  ```
  gpio 1             //选择更改的核心序号：1~4
  minicom            //进入此核心
  passwd             //输入更改密码命令
  请输入你的密码：2次
  ```

#### 2、查看核心的IP地址

- 登录核心后输入：`ifconfig `  即可 

#### 3、分辨前端节点和高性能核心

- `" [root ~*%^%*&$]" `：      带中括号[ ]的是前端节点。

-  `"  sdbox ~*%^%*&$ "`  ：      **不带**中括号[ ]的是高性能核心。

#### 4、程序可以编译但是无法运行

- 可尝试在可尝试在运行文件前加上  “ ` ./`  ” ，使用当前目录
- 可能没有设置启动参数，可以通过  `mpdboot -n 1~4 -f mpd.hosts`  设置。
- 设置启用参数仍然无法运行，可能  `mpd.hosts`  文件内容需要更改。
- 核心损坏

#### 5、`scp` 上传代码成功，但是在高性能核心无法找到文件

- 上传文件时如果没有写目标地址，则会传到主目录下
- 我们登录远程高性能核心以后在 `root` 目录下，所以会造成找不到的错觉
- 2 个解决办法：返回主目录查找或者 `scp` 上传时把目的地址写上

#### 6、关于 mpd.hosts 文件

- 放在主处理单元的 `home` 目录下即可

- `mpdboot -n 2 -f mpd.hosts` ，此命令中间的 `2` 的意思是启用 `mpd.hosts` 文件中前两个节点

- `mpdtrace` 命令可查看启用核心的列表

- `mpd.hosts` 参考写法：

  ```
  loongsonbox-n1
  loongsonbox-n2
  loongsonbox-n3
  loongsonbox-n4
  ```

#### 7、远程登录并控制-龙芯盒子

- 在关机状态下，把网线插入最左边网线接口，开机

- 用 ` ifconfig` 查看前端系统的 `eth0  IP` 地址，并记住

- 用 `SSH` 登录，最后一行表示登录成功

  <div align=center>
  <img src="images\002.png#pic_center" width="40%" alt="标题"/>
  </div>

- 远程控制硬件开关：在自己电脑浏览器地址中输入： `IP/loongson-start.html` 即可控制核心开关。

  <div align=center>
  <img src="images\001.png#pic_center" width="85%" alt="标题"/>
  </div>

#### 8、搭建mpi并行运算中遇到的问题与解决方案

[排除bug](https://blog.csdn.net/Fangrn/article/details/83770816)










<details>
<summary>写在最后</summary>
- 本笔记只是课程知识量很小的一部分，详细且全面的内容请查阅老师发的实验手册。
- 记录的不是很详细，如果有想完善的同学可直接提交新版本，并通知我合并。
- 本人不对内容正确性做任何保证，如有错误可联系我及时修改。
- 感谢提出意见的同学，感谢雷老师和赖师兄的精彩讲课！
</details>

- [ ] 学会了吗?

### Reference

[并行计算：MPI总结](https://blog.csdn.net/qq_40765537/article/details/106425355)

[此实验箱简介](http://www.loongson.cn/business/general2/jiaoxue/jiaoxueshiyanxiang/2015/09/69.html)

[scp命令语法](https://blog.csdn.net/weixin_34177064/article/details/92177168)

[README书写规范](https://github.com/guodongxiaren/README)

