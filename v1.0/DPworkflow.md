
DPworkflow交接文档
```
这是一个markdown文档，建议使用vscode/typora软件打开、编辑、查看大纲，同时我也会转成pdf格式
```
## 服务器配置和PBS作业管理系统
### 一、服务器配置  
172.18.85.141  四台服务器，n1~n4，纯CPU，每台服务器有52个核  
172.18.85.182  七台服务器，n1~n7，每台服务器含一张GPU卡，每台服务器有64个核 

只要提交任务，每个任务只能用一台服务器

**VASP**是CPU任务，选用多少个核需要取决于服务器排队情况，有条件的话核数越多越好，比如在182站点上提交一个任务，可用的最大核数是64  
**DeepMD-kit**和**LAMMPS**需要用GPU计算，GPU任务提交作业时，只需要用1个核计算就行，尽量一台服务器只交一个GPU任务，如果多交的话计算速度会减慢

服务器密码均为```rocky_shi```
### 二、连接服务器需要用到的软件和其他软件  
**WinSCP**：
可视化强，可以像windows操作系统一样打开linux系统中的文件，上传下载文件比较方便。  
**MobaXterm**:
使用linux命令，在命令行中输入命令就可以实现一些功能，很方便。  
**Vscode**：
可以在服务器上进行代码调试，debug很方便。需要在网上学习一下如何使用vscode、[如何连接服务器](vscode连接服务器方法：https://blog.csdn.net/zhaxun/article/details/120568402)，用vscode查看和编辑markdown（很方便）。  
Chemdraw、Materials Studio、VESTA、OVITO  
### 三、PBS作业管理系统
在服务器上提交任何作业都需要一个pbs脚本，提交作业时，在linux里cd进入相应的目录，然后输入```qsub 名字.pbs```，就能提交作业了。  
可以参考[PBS作业管理系统](https://blog.csdn.net/ZuoShifan/article/details/80472681)。  
需要注意的地方是：
如果想要查看服务器中作业运行情况和运行核数，需要进入root用户（```su```,输密码），再输入```qstat -n```。因为婷霞师姐也在跑任务，要进入root用户才能看到服务器里所有的作业和所用核数。

## VASP
### 一、参考网站和资料  
[VASP入门教程系列-b站视频](https://www.bilibili.com/video/BV1r4411B7JE?p=1&vd_source=1221699045050df76df4cda5ffe4c4dc): 比较详细且简单介绍VASP软件的使用  
[VASP官网手册](https://www.vasp.at/wiki/index.php/The_VASP_Manual): 可以在输入框中输入问题查询，一般用来查询INCAR中的关键词含义  
### 二、跑VASP的流程
VASP过程共有三步，也就是要跑三次，分别是<u>原子位置优化</u>、<u>晶格参数优化</u>、<u>第一性原理计算（也称AIMD步）</u>
### 三、VASP的输入输出文件
**输入文件**（除了pbs文件能够自己命名以外，其它文件必须按照如下命名才能被VASP程序识别）：                  
1. INCAR     输入参数设置         
2. POSCAR    结构文件             
3. POTCAR    赝势文件  
4. KPOINTS   k点文件  
5. vasp.pbs  提交作业的脚本   

重要**输出文件**：  
1. CONTCAR 更新结构文件（POSCAR是初始结构，CONTCAR是跑完VASP后的结构）  
2. OUTCAR  完整输出信息  

#### INCAR  
我会提供三个INCAR文件，分别用于①结构弛豫，②单点能量计算，③第一性原理分子动力学计算。里面的内容无需变动。 
#### POSCAR
本次实验用的体系为```Ni3(HIB)2```,详见```articles```文件夹。  

**Chemdraw画好初始结构**→在**Materials Studio中建模**（我发给你的视频。后面想要改晶胞参数，直接在MS中改就行，Chemdraw只用于一开始把结构画出来），注意z方向要建一个20埃的真空层，也就是把二维层状材料放在中间10埃的位置，建好后导出cif文件→在**VESTA**中转换格式，打开cif文件，导出为POSCAR格式，如果弹一些选项不用管，直接默认，此时POSCAR是分数坐标形式（Fractional coordinates）→文件名要改为POSCAR，不带后缀  

#### POTCAR
1.赝势文件需要自己拼接，单元素的赝势文件在172.18.85.182站点下的目录为```/data/potpaw/potpaw_PBE.54.tar_0_0```，在172.18.85.141站点下的目录为```/data/vasp/potpaw/potpaw_PBE.54.tar_0_0```中。  
2.赝势文件中元素的顺序必须与POSCAR中元素的顺序一致，对于你研究的那个材料，由于POSCAR元素顺序一般不会改变，在VASP的三步流程中一直沿用同一个POTCAR就行。  
举例：以下是一个POSCAR文件,元素顺序为S、C、Pt  
```
A-material
1.0
       15.0600004196         0.0000000000         0.0000000000
       -7.5300002098        13.0423429444         0.0000000000
        0.0000000000         0.0000000000        20.0000000000
    S    C   Pt
    1    2    3
Direct
     0.081589937         0.637929916         0.500000000
     0.918410063         0.362070084         0.500000000
     0.362070084         0.443660021         0.500000000
     0.637929916         0.556339979         0.500000000
     0.556339979         0.918410063         0.500000000
     0.443660021         0.081589937         0.500000000
```
如果想得到对应元素顺序的POTCAR文件，并放置在我的目录/share/wxm下，可以在命令行中依次输入以下命令或者建立一个bash脚本（把以下内容都放进脚本里）并运行：（注意：千万不要删了赝势文件目录里的东西）
```bash
'''
这是我自己弄的方式，你们可以修改一下这个脚本
'''
cd /data/potpaw/potpaw_PBE.54.tar_0_0     # 进入赝势文件目录
cp S/POTCAR /share/wxm/POTCAR_S           # 将S的POTCAR文件复制到我自己的目录下，并命名为POTCAR_S
cp C/POTCAR /share/wxm/POTCAR_C
cp Pt/POTCAR /share/wxm/POTCAR_Pt
cd /share/wxm                             # 进入我自己的目录
cat POTCAR_S POTCAR_C POTCAR_Pt >POTCAR   # 拼接三个文件并命名为POTCAR
```
#### KPOINTS
我会提供三个KPOINTS文件，分别用于①结构弛豫，②单点能量计算，③第一性原理分子动力学计算。里面的内容无需变动。  
#### vasp.pbs  
我会提供一个vasp.pbs文件，只要在命令行中qsub vasp.pbs就可以跑任务了。想要查看任务运行状态，输入qstat。以下中文注释需要留意一下。
```bash
#!/bin/bash
#PBS -q ykq                     这一行写队列名，方便区分是谁的作业                         
#PBS -V
#PBS  -l   walltime=1000:00:00  最大运行时间
#PBS  -l   nodes=n1:ppn=63      指定n1节点跑任务，用63个核，根据服务器具体情况修改此项
#PBS  -N   name                   给作业命名

cd $PBS_O_WORKDIR

ulimit -s unlimited

NP=`cat $PBS_NODEFILE|wc -l`
mpirun -np $NP vasp_std  > run.log
```
服务器上默认的队列名为batch，如果想要把队列名改为ykq，可以如下操作：
```bash
su    进入root用户，即此时权限最高（然后输入密码rocky_shi）
qmgr -c "create queue ykq queue_type = Execution"   创建ykq队列
qmgr -c "set queue 队列名 enabled = True"           永久开启队列
qmgr -c "set queue 队列名 started = True"           立刻开启队列
```

### 四、跑VASP的流程
VASP过程共有三步，分别是<u>原子位置优化</u>、<u>晶格参数优化</u>、<u>第一性原理计算（也称AIMD步）</u>
#### 1 原子位置优化  
**POSCAR**文件：先从Materials Studio中建好一个**单胞**结构（并且用VESTA转成POSCAR格式），跑一次结构弛豫，此时晶格参数是固定的，只会改变原子位置至最优值。  
此时采用我给的第一个**INCAR**文件：  
```
 SYSTEM = 自己随便命名
 ISTART = 0; ICHARG = 2
 ENCUT = 500
 PREC = Normal
 ISMEAR = 0; SIGMA = 0.01
 ALGO = Fast; LWAVE = .FALSE.; LCHARG = .FALSE. 
 EDIFF = 1.0E-05
 EDIFFG = -0.01; ISIF = 2; IBRION = 2; NSW = 1000
 ISYM = 0

 GGA = PE
 IVDW = 11
 VDW_RADIUS = 50.2
 VDW_CNRADIUS = 20.0
 VDW_S8 = 0.722
 VDW_SR = 1.217
```
**POTCAR**文件自己从赝势库里拼接  
此时采用我给你的第一个**KPOINTS**文件：
```
K-Points
 0
Gamma
 4  4  1
 0  0  0
```
**vasp.pbs**文件如下，凡是调用计算VASP的任务，都可以用此脚本

```bash
#!/bin/bash
#PBS -q ykq                     这一行写队列名，方便区分是谁的作业                         
#PBS -V
#PBS  -l   walltime=1000:00:00  最大运行时间
#PBS  -l   nodes=n1:ppn=63      指定n1节点跑任务，用63个核，根据服务器具体情况修改此项
#PBS  -N   name                   给作业命名

cd $PBS_O_WORKDIR

ulimit -s unlimited

NP=`cat $PBS_NODEFILE|wc -l`
mpirun -np $NP vasp_std  > run.log
```
#### 2 晶格参数优化  
##### 2.1 晶格参数优化  
**POSCAR**文件准备过程：取出上一步的结果文件**CONTCAR**→在VESTA中转换为cif格式→将cif格式的结构文件在Materials Studio打开→在空白位置右键→Lattice Parameters→在Lengths中修改a、b轴的长度（注意a、b轴必须等长），即可实现修改不同的晶格参数→保存为cif格式→在VESTA中转换为POSCAR格式（如前所述）<u>这段步骤我是这么操作的，可能很繁琐，你可以查一下或者问石老师有没有可以能够批量修改晶格参数的方法，或者写个脚本计算换算一下坐标位置，直接生成一系列不同晶格参数的POSCAR文件。</u>  
这里主要是为了**找到哪个晶格参数**的能量最低，体系最稳定。所以要设置不同晶格参数进行测试。  
[点击查看这个图：随着晶格参数不同，体系总能量有最低值](https://postimg.cc/zbRhmzMd)  

对于不同的晶格参数的POSCAR文件，我们都分别跑一下结构弛豫（沿用第一步的INCAR、POTCAR、KPOINTS、vasp.pbs），得到该晶格参数下最优的结构。  
##### 2.2 单点能计算  
在上一步中，我们计算得到了多组晶格参数下结构弛豫后的结果，现在我们需要分别在高精度状态下计算它们的单点能。  
**POSCAR**文件：在这轮计算中，我们把上一步计算的最终结构CONTCAR文件复制到当前计算目录下，并重命名为POSCAR。  
此时采用我给的第二个**INCAR**文件：  
```
 SYSTEM = 自己随便命名
 ISTART = 0; ICHARG = 2
 ENCUT = 600
 PREC = Accurate
 ISMEAR = -5
 ALGO = Normal; LWAVE = .False.; LCHARG = .False. 
 EDIFF = 1.0E-06
 ISYM = 0

 GGA = PE
 IVDW = 11
 VDW_RADIUS = 50.2
 VDW_CNRADIUS = 20.0
 VDW_S8 = 0.722
 VDW_SR = 1.217
```
**POTCAR**和**vasp.pbs**文件沿用之前的  
此时采用我给的第二个**KPOINTS**文件：  
```
K-Points
 0
Gamma
 6  6  1
 0  0  0
```
单点能计算结果可以通过如下方式查看：  
1. **OUTCAR**文件：从后往前找，有一行```  free  energy   TOTEN  =      -173.53180515 eV```,即为单点能结果  
2. **OSZICAR**或者**run.log**文件，最后一行```E0= -.17353181E+03```  

可以画出Total Energy对Lattice parameter的图，找到能量最小时对应的晶格参数。（或者写python脚本分析比较）  
#### 3 第一性原理计算（也称AIMD步）
这个过程最花时间，预计3周。  
<font color="#dd0000">我们重点关注从这一节开始和以后的workflow过程，因为workflow中的材料已在晶体库有标准cif文件（你们可以去问一下老师怎么在晶体数据库中获取cif文件），无需经历刚才的1、2步去优化材料结构参数，而可以直接跑第3步。而老师让你们尝试的这个材料在晶体库中没有cif文件，需要跑1、2步。你们也可以写脚本看看怎么简化1、2步。</font><br />   
**POSCAR**文件：在第2步中得到了对应与能量最低的晶格参数结构文件**CONTCAR**，此时需要把其**扩胞成2\*2\*1的超胞**。可以写脚本或者调包来扩胞。  
或者Materials Studio中扩胞方式为：build→symmetry→supercell→2 2 1  
此时采用我给的第三个INCAR文件：  

```
 SYSTEM = 自己随便命名
# electronic degrees 
 #IVDW   =  1 
 ENCUT = 400                                 
 LREAL = A                      # real space projection                         
 PREC = Normal                 # chose Low only after tests
 EDIFF = 1.0E-5                   # do not use default (too large drift)          
 ISMEAR = 0; SIGMA = 0.01
 ALGO = Fast               # recommended for MD (fall back ALGO = Fast)
 ADDGRID = TRUE     
 MAXMIX = 40                    # reuse mixer from one MD step to next                                 
 NELMIN = 4                     # minimum 4 steps per time step, avoid breaking after 2 steps
 # MD (do little writing to save disc space)        
 LBLUEOUT = .TRUE.
 IBRION = 0
 NSW = 15000
 LCHARG = .FALSE.; LWAVE = .FALSE.  
 TEBEG = 298.0; TEEND = 298.0
 POTIM = 1.0
 SMASS = 0
 MDALGO = 2
 ISIF = 2
 NBLOCK = 1
 KBLOCK = 1
```
**POTCAR**和**vasp.pbs**文件沿用之前的  
此时采用我给的第三个**KPOINTS**文件：  
```
K-Points
 0
Gamma
 1  1  1
 0  0  0
```