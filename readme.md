---

typora-root-url: img
---

系统辨识及PID参数调节

[github更新地址](https://github.com/Skylark0924/System_Identification) 

## 基础知识

1. **系统辨识：** 系统辨识是根据系统的输入输出时间函数来确定描述系统行为的数学模型。现代控制理论中的一个分支。通过辨识建立数学模型的**目的**是估计表征系统行为的重要参数，建立一个能模仿真实系统行为的模型，用当前可测量的系统的输入和输出预测系统输出的未来演变，以及设计控制器。

   Zadeh(1962)指出：“系统辨识是在输入和输出数据的基础上，从一类模型中确定一个与所观测系统等价的模型”。

   Ljung(1978)指出：“系统辨识有三个要素——数据、模型类和准则，即根据某一准则，利用实测数据，在模型类中选取一个拟合得最好的模型”。

2. **[状态空间表达式](https://blog.csdn.net/weijifen000/article/details/83045930)：**

   - **经典控制理论：** 对一个线性定常系统，可用**常微分方程或传递函数**加以描述，可将某个单变量作为输出，直接和输入联系起来。实际上系统除了输出量这个变量之外，还包含有其它相互独立的变量，而微分方程或传递函数对这些内容的中间变量是不便描述的，因而不能包含系统的所有信息。显然，从能否完全揭示系统的全部运动状态来说，用微分方程或传递函数来描述一个线性定常系统有其不足之处。

   - **现代控制理论：** 在用**状态空间法**分析系统时，系统的动态特性是用由状态变量构成的一阶微分方程组来描述的。它能反映系统的全部独立变量的变化，从而能同时确定系统的全部内部运动状态，而且还可以方便地处理初始条件。这样，在设计控制系统时，不再只局限于输入量、输出量、误差量，为提高系统性能提供了有力的工具。加之可利用计算机进行分析设计及实时控制，因而可以应用于**非线性系统、时变系统、多输入—多输出系统以及随机过程**等。

   - **状态空间表达式：** 由**状态方程**和**输出方程**构成，在状态空间中对控制系统作完整表述的公式。

   系统的状态方程为：

   <div align=center>
   <img src="https://github.com/Skylark0924/System_Identification/blob/master/img/state1.jpg" width="60%">
   </div>
   或表示为：

   ​							$$\dot{x}(t)=Ax(t)+Bu(t)$$

   其中，A 为 n×n 的常系数矩阵，称作**系统矩阵** ； B 为 n×r 的常系数矩阵，称作**控制矩阵**。 A 与 B 都由系统本身的参数决定。 u 是输入信号， x 是状态向量。

   系统的输出方程：

   <div align=center>
   <img src="https://github.com/Skylark0924/System_Identification/blob/master/img/state2.jpg" width="60%">
   </div>

   或表示为：

   ​							$$y(t)=Cx(t)+Du(t)$$

   其中，C 为 m×n 的常系数矩阵，称为**输出矩阵**，它表达了输出变量与状态变量之间的关系； D 为 m×r 的常系数矩阵，称为**直接转移矩阵**，它表示输入变量通过矩阵 D 直接转移到输出。在大多数实际系统中， D=0 。 y 是输出， x 是系统状态， u 是输入。



## 系统辨识

### 云台YAW轴开环系统辨识

以云台6623电机为例

1. 利用MATLAB生成采样频率为500Hz，幅值为1500，从0Hz到10Hz的扫频信号，并生成为txt文件（程序：sweep_wave_script_txt.m)

   ![current_num1](https://github.com/Skylark0924/System_Identification/blob/master/img/current_num1.jpg)

2. 利用生成的扫频信号作为`GMY.Intensity`的输入激励电机转动，使用`J-LINK`代替`ST-LINK`作为`Debugger`，需要在`Settings`中检查一下连接。J-Link的SWD接线方式：

   <div align=center>
   <img src="https://github.com/Skylark0924/System_Identification/blob/master/img/20160308191319794.png" width="60%">
   </div>

   <div align=center>
   <img src="https://github.com/Skylark0924/System_Identification/blob/master/img/8.12.00-5.jpg" width="30%">
   </div>

   <div align=center>
   <img src="https://github.com/Skylark0924/System_Identification/blob/master/img/1551061273990.png" width="80%">
   </div>

   <div align=center>
   <img src="https://github.com/Skylark0924/System_Identification/blob/master/img/1552394220100.png" width="80%">
   </div>

   **注意：** `Port`修改为`SW`；`SW Device`中有序列号即表示`JLink`连接成功。

3. 利用`Jscope`监测电流、角度和角速度输出值。此时电机应该**已经安装好了实际的负载**，因为系统辨识是需要得到该电机在工作状态下的传递函数，用这个数学模型来模拟实际情况，因此**需要在装好的车上进行系统辨识**，并且**要保证电机和所带负载在运动行程中没有受到机械限位的约束**。

   **注意：**电流、角度和角速度对应的变量必须乘1000转化为整形才能被`Jscope`读取

   ```c++
   GMYtarget_int=(int)(GMY.Intensity*1000);
   GMYAngleSpeed_int=(int)(imu.wz*1000);
   GMYAngle_int=(int)(imu.yaw*1000);
   ```

   `Jscope`的配置如下，`Sample Rate`是`500Hz`，因此前面是`2000`；`Elf File`就是我们程序编译生成的`RM_frame.axf`文件：

   <div align=center>
       <img src="https://github.com/Skylark0924/System_Identification/blob/master/img/1551072707680.png" width="40%">
   </div>

   使用`Debug`模式运行代码，同时使`Jscope`开始记录。`Jscope`的输出形状大致如下图，蓝色为输入的扫频信号，绿色为角速度值，黄色为角度值：

   ![1551071825317](https://github.com/Skylark0924/System_Identification/blob/master/img/1551071825317.png)

4. 将`Jscope`的数据导出到CSV中，在Excel 365中用**数据导入**功能，将CSV文件转化为`.xlsx`格式，并在Excel中对数据进行必要的预处理，找出一个合适的测量段范围。例如，在`mydata.m`文件中所用的`0126_1502.xlsx`数据的测量段是`B3083:D15582`，从扫频起点开始，正好是12500个采样值，也就是一次扫频信号的输出结果，其中三列分别是扫频信号、角速度、角度。

   **注意：**此处三列数据均要除以1000，变换回`double`值。


   ![1](https://github.com/Skylark0924/System_Identification/blob/master/img/1.jpg)

   上图的输出结果和`Jscope`上意义相同，只不过我使用了相反的扫频信号（当时受到了另一侧的机械限位，以后可以直接用正的扫频信号）

5. 打开MATLAB的`System Identification`工具箱。

   - `Import data`中选择`Time domain data`，在弹出的对话框中输入：

     <div align=center>
         <img src="https://github.com/Skylark0924/System_Identification/blob/master/img/1551079116700.png" width="40%">
     </div>

   - `Estimate`中选择`State Space Models`，默认采用四阶的，修改为离散时间，点击`Estimate`，可以得到一份辨识报告，适配度在99.96%左右：

     ![1551079320799](https://github.com/Skylark0924/System_Identification/blob/master/img/1551079320799.png)

   - 此时`Model Views`中就生成了一个模型`ss1`，将其拖拽到`To Workspace`控件上，即可在工作区看到这个模型，**即为辨识到的系统状态空间表达式**。

   **注意：**可以使用扫频数据和角速度数据按照上述过程也辨识一个模型，用于`mydataplot.m`中进行测试比对。

   

### 结合Simulink调节PID参数

Simulink模型是`test5_RegularPID.slx`文件。其中`ss1`就是刚辨识出来的模型，`input1`是时间和角速度的增广，`input2`是时间和角度的增广，这两个在`mydataplot.m`中生成。该Simulink模型由速度环和位置环组成，位置环的输出作为速度环的输入，将其与实际角速度进行差分。

![1551079903739](https://github.com/Skylark0924/System_Identification/blob/master/img/1552395022478.png)

![1552395022478](/../../../Academic_information/To_be_a_Roboticist/1552395022478.png)

- 调节PID参数时，先调节速度环的PID：

  ![1551080530835](https://github.com/Skylark0924/System_Identification/blob/master/img/1551080530835.png)

- 点击`Tune...`，打开`PID Tuner App`，利用该工具箱调节PID，如下：

  ![1551081431700](https://github.com/Skylark0924/System_Identification/blob/master/img/1551081431700.png)

- 调节好速度环之后，输出PID参数，点击`Update Block`，即可在`Block Parameters`中看到调整好的PID参数；

- 位置环PID同理；

- 最后输入、输出的Scope如下图（本例前段数据有异常，只要输入输出相似就好）。

  ![1551081111162](https://github.com/Skylark0924/System_Identification/blob/master/img/1551081111162.png)

得到的速度PID和位置PID写入frame的相应电机PID参数中，即可得到较好的效果，接下来在此数据基础上微调即可。

### 云台PITCH轴闭环系统辨识

**使用闭环辨识的原因：** 与yaw轴不同，pitch轴需要克服弹仓的偏重，因此如果直接赋予pitch轴电机扫频信号的电流激励，会出现弹仓太沉，pitch轴完全抬不起的情况。即pitch轴辨识时，若切断反馈回路，会无法工作，这是就**要有位置环和速度环来保证云台在一定范围内运动而不失控**（初始PID）。

#### 实际系统分析

由于速度环的H(s) = 1，为单位反馈系统，因此原先的PID控制的云台系统简化如下，P(s)为PID控制器，M(s)为云台和电机模型

![img](https://github.com/Skylark0924/System_Identification/blob/master/img/020245to9ifzz6wzhz3jl0.png)

为了提高性能，我们在PID控制器之前加入补偿器，得到系统框图如下所示：

![img](https://github.com/Skylark0924/System_Identification/blob/master/img/020245fgtp8mbq5bt7yy8y.png)

这里将原来的PID控制器和云台电机模型看做一个整体，当做被控对象G(s)，针对G(s)设计一个新的控制器C(s)，此即我们所要设计的补偿器。

#### 闭环系统辨识

与开环不同，闭环辨识因为包含了PID反馈回路，不能直接给电流信号，而是应该输入目标角度，输出实际的`imu.pit`。按照开环的步骤进行`Jscope`数据采集以及系统辨识，然后使用`sisotool`工具箱进行环路整形（在MATLAB命令行窗口输入sisotool打开）。

#### 传递函数转换

假设辨识得到的闭环传递函数为Φ(s)，而`sisotool`需要根据开环传递函数进行补偿器的设计，因此需要进行一次变换。

由$$\Phi(s)=\frac{G(s)}{1+G(s)}$$

可得原系统的开环传递函数为：

$$G(s)=\frac{\Phi(s)}{1-\Phi(s)}​$$

#### sisotool工具使用和环路整形（这是一个强大的工具箱）

sisotool（Single Input Single Output Toolbox）是MATLAB提供的单输入单输出系统补偿器的设计工具。

在MATLAB的命令行窗口输入sisotool，打开sisotool工具。

点击 **Edit Architecture** ，打开系统框架对话框，默认框架即可，选择我们之前计算得到开环传递函数G(s)导入为G，其他保持不变。

导入G之后，界面默认显示内容如下所示：

![img](https://github.com/Skylark0924/System_Identification/blob/master/img/1.png)

左侧是框架选择和预览区，中间是开环传递函数bode图编辑区，右上方是根轨迹编辑区，右下方是系统的阶跃响应图。

- 点击 **New Plot** 可以根据自己的需求添加图形。例如添加闭环传递函数图、零极点图。

- 点击菜单栏里的 **Store** 按钮，可以将当前的设计保存下来，以方便后续进行对比和回溯。

控制器的设计一般都是基于开环传递函数的零极点，可以手动增加零极点来调整系统，通过观察伯德图和根轨迹图来确定参数是否合理。借助MATLAB的强大的自动调整功能，我们可以使用自动的方式来进行控制器设计。

1. `Tuning Methods`中的`PID Tuning`：

![1552406133882](https://github.com/Skylark0924/System_Identification/blob/master/img/1552406133882.png)

点击`Update Compensator`自动生成补偿器传递函数，可以通过阶跃响应曲线观察是否合适。

2. `Tuning Methods`中的`Optimization Based Tuning`：
![1552406344201](https://github.com/Skylark0924/System_Identification/blob/master/img/1552406344201.png)

再点击`Design Requirements`右下角的`Add new design requirement`

![1552406473499](https://github.com/Skylark0924/System_Identification/blob/master/img/1552406473499.png)

这里可以自定义想要的响应曲线的上升时间、超调量等参数，然后点击`OK`、`Start Optimization`进行优化迭代得到预想的响应曲线

![1552406649929](https://github.com/Skylark0924/System_Identification/blob/master/img/1552406649929.png)



最后，根据得到的补偿器传递函数写成C代码写入`RM_frame`中，完成补偿。



## 参考文献

[1]  刘豹, 唐万生. 现代控制理论[J]. 2006.

[2]  刘金琨, 沈晓蓉, 赵龙. 系统辨识理论及 MATLAB 仿真[J]. 2013.