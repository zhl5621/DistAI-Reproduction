本文提供的工件包括所有 DistAI 源代码以及相应的 docker 镜像。docker 镜像已经安装了所有依赖项，使用 docker 镜像可以方便地重现本文的实验结果和进行测试。
因此进行实验所需的软件环境为安装了 docker 的 Linux 系统，我们的测试工作也在docker容器内进行。论文中的协议名称与程序中的缩写词之间的映射见GitHub 中README.md，例如对于协议名”hashed sharding”在程序中用缩写词”shard”代替。
整个测试过程需要用到下面四个Python文件：testMain.py用来测试DistAI的性能，testI4.py用来测试对照算法I4的性能，testFolic3.py用来测试对照算法Fol-IC3的性能。drawFigure用来绘制number of samples与sampling runtime、enumeration runtime、refinement runtime之间的图像，用来验证文中所提出的采样帮助降低refinement开销的结论。
（一）测试过程说明：
1.安装docker和加载docker镜像
安装 docker.
for Windows/Mac OS: https://www.docker.com/get-started
for Linux: https://docs.docker.com/engine/install/
下载docker镜像: https://drive.google.com/file/d/1ogBU9KvZsvSRhXerY9Bv-MuiW9oOezBU/view?usp=sharing
加载docker 镜像
docker load -i the_address_to_the_image
运行docker container
docker run -it distai /bin/bash

2.将测试代码复制到docker容器中，需在终端（而不是docker容器内）执行如下指令：
docker cp distai/testMain.py 2a7ee2d5cc64:/testMain.py
docker cp distai/testI4.py 2a7ee2d5cc64:/testI4.py
docker cp distai/testFolic3.py 2a7ee2d5cc64:/testFolic3.py
docker cp distai/drawFigure.py 2a7ee2d5cc64:/drawFigure.py
其中distai/为测试代码存放路径，可根据实际进行修改，2a7ee2d5cc64为docker容器id，在测试时需修改为测试人员的docker容器id。这些指令将测试代码复制进了docker容器根目录。

3.测试DistAI
首先进入DistAI，需执行如下指令：
cd /DistAI
然后执行如下指令，对DistAI进行一键自动化测试：
python ../testMain.py
测试完成后，测试结果将输出到docker容器/testResult/testMainOutput.csv中，testMainOutput.csv包含如下字段：
PROBLEM、simulation_time、enumeration_time、refinement_time、total_time、Invariants、isProved
其中PROBLEM为协议名称简写，Invariants为所求解到的Invariant数量，isProved表示该协议是否通过证明

4.验证采样帮助降低refinement开销的结论
首先进入DistAI，需执行如下指令：
cd /DistAI
然后执行如下指令进行一键自动化测试：
python ../drawFigure.py
测试完成后，所获得的图像将保存在/testResult中，图像名为PROBLEM_tradeoff.png，其中PROBLEM相应代表协议名的简写。可通过观察这些图像是否均满足随着采样量增大，refinement时间降低，来验证文中结论。

5.测试对照算法I4，获取其性能数据
进入docker容器中的myI4
cd /myI4
切换为Python 2环境 
conda activate py2
进行I4测试
python ../testI4.py
测试结果将以表格的形式打印到控制台
表格包含protocol、final time、total time字段。对I4进行测试，需要进行total和final两部分测试，相应的时间即为total time、final time。
测试完成后切换为Python 3环境
conda activate base

6.测试对照算法FOL-IC3，获取其性能数据
进入docker容器中的myfolic3
cd /myfolic3
进行FOL-IC3测试
python ../testFolic3.py
FOL-IC3测试包括forall测试和default测试，执行上述指令后系统会自动对各个协议进行forall测试和default测试，测试结果（运行时间）将打印到控制台上。

（二）代码开发工作描述
testMain.py
testMain.py基于DistAI中的main.py进行修改，原文中的main.py需要用户对于每一个协议都手动输入一系列指令并手动记录实验结果，而testMain.py实现了进行自动化测试、控制台输出测试进度提示和自动汇总测试数据并输出为csv文件等功能。
testMain.py中的新增代码部分：
8-23行的problemList为待测试协议列表，可修改该列表内容进行测试用例修改。
28-37行为创建输出结果文件及其提示
40行为遍历problemList，对待测试协议进行逐个测试。
98-102行为将实验结果汇总并按格式写入输出文件中。
testI4.py
基于myI4.py进行修改，原文中的myI4.py需要用户对于每一个协议都手动输入一系列指令和选项并手动记录实验结果，testI4.py实现了进行自动化测试、控制台输出测试进度提示和自动汇总测试数据并整理为表格进行输出。
tips：testI4.py是用python2 进行开发的，在Python3环境下查看其代码可能会误报语法错误，但在Python2环境下该程序可正常运行。
由于实现了自动化测试，所以原先myI4.py中接受控制台输入参数的部分代码即被删除。
testI4.py中的新增代码部分：
56-66行的problemList为待测试协议列表，可修改该列表内容进行测试用例修改。
68行result为输出结果列表，用来记录输出结果。
70、88、89行等部分为测试进度提示语句。
105-109行为整理输出结果为表格，并输出。
testFolic3.py
testFolic3.py中所有代码均为我们新增加的代码。
4-22行run_mypyvy函数为构建控制台指令函数。
23-34行的forAllList为对FOL-IC3进行forall测试的待测试协议列表。
35-39行的defaultList为对FOL-IC3进行default测试的待测试协议列表。
其余部分为测试功能主体部分，testFolic3.py需要构建控制台指令以调用原文的mypyvy.py并获取其输出结果。
drawFigure.py
基于原文的tradeoff_figure.py进行修改。实现了自动化测试、控制台进度提示和自动绘制图像并命名保存。
drawFigure.py中：
10-22行为待测试协议列表，可修改该列表内容进行测试用例修改。
24-83行基于tradeoff_figure.py进行修改，并将其封装为函数，记录实验数据。
86-145为根据实验数据绘制图像并根据命名格式进行命名和保存。原文只展示了对一个协议的测试结果图像。我们对11个协议进行了测试并得到了结果图像。原文实验图像用pdf进行保存，不便于查看，我们修改了对应代码将图像转为png格式进行保存。











