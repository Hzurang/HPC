# HPC 算力集群使用指南（author: 林俊燃）

## ssh 连接方式

使用 xshell、Termius、puty 等 ssh 工具进行连接，具体地址根据不同学校的算力集群进行了解，输入对应的账号和密码即可登录。（x86 和 arm架构的电脑具体地址可能会有差别）



## 数据传输

可使用 xftp 或 Scpftp 工具进行数据传输，或是利用 Termius 软件直接进行拖拽传输（界面化，不用输入相关的命令，方便）



## 环境管理工具

```shell
# 查看当前 module path 中的所有可用软件环境模块文件
module avail

# 载入某个软件
module load xxx
# 本集群中常需要加载的软件如下，cuda 需根据用户的需要选择不同的版本
module load anaconda3
module load cuda-10.0
module load cuda-11.1

# 查看已加载的软件环境
module list

# 取消载入的某个软件环境
module upload xxx

# 取消所有已载入的软件环境，即重置
module purge
```



## 作业提交

### 作业提交命令

```shell
jsub -n <CPU 数> -q <队列名> [-J <作业名> -o output.%J -e error.%J] ./commandline
# -n <CPU 数> 指定执行该作业所需的 CPU 数
# -q <队列名> 指定该作业运行时使用的队列，所有的作业都会被提交到队列里在资源不足时排队，在资源满足时派发并执行。
# -J <作业名> 指定作业名，方便辨识作业
# -o output.%J 指定输出文件，这个输出指的是调度系统的输出，如果作业的输出没有重定向到应用自己的 log 文件里也会输出在这个文件里。%J 表示的作业号，及最终会产生一个 output.<作业号>的文件。
# -e error.%J 指定错误输出文件，这个输出指的是调度系统的输出，如果作业的错误输出没有重定向到应用自己的 log 文件里也会输出在这个文件里。%J 表示的作业号，及最终会产生一个 error.<作业号>的文件。
```



### 作业查看命令

```shell
jjobs [[-a] [-l] <作业号>]

# 不使用参数，显示目前正在排队或运行的作业
jjobs

# 显示最近完成的作业及正在运行或排队的作业
jjobs -a

# 显示指定的作业状态 e.g. jjobs 213
jjobs <作业号>

# 显示指定作业的详细信息
jjobs -l <作业号>
```



### 提交通用串行作业

```shell
jsub -q normal -n 1 -e error.%J -o output.%J -J my_job ./test
# <jsub>：提交作业
# <-q normal>：指定 normal 队列
# <-n 1>：指定该作业所需 1 核
# <-e error.%J>：错误信息输出到 error.%J 文件中
# <-o output.%J>：正确信息输出到 output.%J 文件中
# <-J my_job>：指定作业名字为 my_job
# <test>：执行的脚本
```



### 提交 mpi 并行作业

```shell
jsub -n 80 -q para -e error.%J -o output.%J ./test
```



### 提交 matlab 作业

```shell
jsub -q matlab -e error.%J -o output.%J matlab -nodisplay -r test2

# 需要启动多少个 worker 需要在 test.m 里设置，比如 parpool(48) 表示即将启动48个 worker 来计算该作业
```



### 提交 gpu 作业（重要）

```shell
 jsub -q qgpu -e error.%J -o output.%J test
```



## 资源查看

```shell
# 查看可用的 cpu 资源
jhosts

# 查看所有资源包含cpu、内存、显卡等资源
jhosts attrib -l

# 查看指定机器的资源情况，hostname是指定机器的名字，例如:hpc1 -4
jhosts attrib -l hostname 
```



## 作业管理（命令行模式）

在进行作业管理时，小心误删别人正在运行的作业。

```shell
# 作业提交命令（shell 脚本方式）
jsub < xxx.sh

# 作业查询命令
jjobs
jjobs –l jobid

# 作业置顶命令
jctrl top

# 作业置底命令
jctrl bot

# 终止作业命令
jctrl kill jobid

# 作业进度跟踪
jctrl peek -f jobid

# 作业挂起（PSUSP或USUSP）
jctrl stop jobid

# PSUSP和USUSP状态作业恢复
jctrl resume jobid

# 历史作业命令
jhist
jhist –l jobid
```



## 队列管理

一般用户应该没有权限进行该项操作

```shell
# 列出自己可用的队列 e.g. jqueues -u user001
jqueues -u <自己账户名>
```



## 具体实例

这里我就以 yolov5 的训练过程来举例，其他的深度学习以及别的领域需要用到算力进行的训练或计算过程，举一反三即可。

```shell
#!/bin/bash -l
#JSUB -J train
#JSUB -m hpc1
#JSUB -R span[hosts=1]
#JSUB -gpgpu 4
#JSUB -n 1
#JSUB -e err.log
#JSUB -o ${OUT}/out.log

module load cuda-10.0
module load anaconda3
source activate
conda activate yolov5
cd /xxx/xxx/xxx/yolov5-pytorch-main/yolov5-pytorch-main
pip install -r requirements.txt
python train.py
```

```shell
# 1.加载软件环境
# 2.创建用于项目的 conda 虚拟环境并激活
# 3.进入具体目录
# 4.pip 安装对应项目的依赖环境
# 5.指定训练文件
# 整体流程基本和 linux 一致，最好有 shell 基础

# JSUB -J 作业名称
# JSUB -m hpc1 这是指定对应的机器，可通过查看集群资源的命令看到
# JSUB -gpgpu 4 使用 gpu 时补充的命令
```

