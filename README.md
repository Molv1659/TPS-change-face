# TPS-change-face
change faces of different people and some other interesting picture processing
#<center> 数值分析第一次大作业
##一.程序界面
###form1：
![avatar](form1.jpg)
左侧为结果显示，原始图像默认为
![avatar](THU.jpg)
右上方可选择插值方式：最近邻插值，双线性插值，双三次插值。
旋转模块可以旋转图片旋转的角度和半径，默认是逆时针方向旋转，可选上反向使变成顺时针方向旋转。选好参数后点击旋转扭曲即可在左方看到显示的结果：
![avatar](xuanzhuan2.jpg)
畸变模块点击两个按钮可看到相应结果。
![avatar](jibian1.jpg)
![avatar](jibian2.jpg)
可点击蓝色的“转到TPS变脸界面”切换到form2.
###form2
![avatar](form2.jpg)
分别导入原始图像和目标图像以及相应人脸关键点后，可进行变脸。导入人脸关键点后，关键点会标注显示在图像上，方便检查关键点是否导入正确。
插值方式有：最近邻插值，双线性插值，双三次插值。由于变脸后一些位置的像素无法在原图中找到，使用原图同位置的像素进行填充。对满意的变脸结果，可点击保存图片进行保存。
可点击蓝色的“返回上一个界面”返回form1.
下面是几个变脸效果展示：
![avatar](变脸2.jpg)
![avatar](变脸3.jpg)
![avatar](变脸4.jpg)
##二.任务分析
###任务一：图像旋转
旋转变换是将(x,y)变换到(x',y')
目标是知道变换后图像每个像素点的值，通过反变换可由(x',y')知道对应的(x,y)。由于是数字图像x'和y'都是整数，反变换回去的x和y不一定是整数，所以需要用插值函数来算出(x,y)的像素值。
###任务二：图像畸变
畸变分为桶形畸变和枕形畸变。
桶形畸变是越靠近图像中心的位置越膨胀，枕形畸变是越靠近图像中心的位置越被压缩。
设r是原图像中像素点(x,y)到图像中心的长度，r1是桶形畸变图像中像素点(x',y')到图像中心的长度，r2是枕形畸变图像中像素点(x'',y'')到图像中心的长度，$r_{max}$为图像所有像素点里里中心点最远的距离，到并且设桶形畸变使(x,y)变到(x',y')，枕形畸变使(x,y)变到(x',y')。则有0< r < $r_{max}$时，r'=f(r)为上凸函数，r''=g(r)为下凸函数（凹函数）。
我选择了指数对数作为变换函数。相比于PPT之中给出的变换函数，变化效果更明显。同时没有选幂函数是因为，使用幂函数做桶形畸变时，不妨设$r'=r^2$,（为方便说明，考虑都已经经过归一化处理）则$r=\sqrt{r'}$,在r'接近0时，r关于r'的导数过大，导致图变换的效果是最中心的像素放大得过多出现了明显几个大像素块，本例来说就是清华园的华字放地过大且很模糊，已无法辨认是华字。而选择指数对数函数则既有明显的畸变效果又不至于影响清晰度
###任务三：TPS变脸
要将原图像上的人脸关键点变换后变到目标图像的人脸关键点位置，其它位置的像素点的变换，根据薄板样条模型是使能量函数最小的。PPT中所给TPS求解是有原图像(x,y)变到(x',y')的公式，为求得变换后图像每个像素点的值需要知道反变换的公式。推出正变换后在数学上推到反变换不方便，直接将PPT中带撇不带撇交换，求得的是由(x',y')变到(x,y)的公式,即是我们所需的反变换公式。得到反变换公式后然后利用插值公式求变换后图像每个像素点的值即可。
对(x',y')做反变换得到的(x,y)可能不在原图的范围内，即找不到对应像素，考虑到这种情况主要发生在边缘，使用者主要关注的是变脸的部分，所以为提升视觉效果，将这部分像素用原图对应位置的像素填充而不是置为黑色。
由于图像像素大小不同，在图内脸的位置也不尽相同，所以有必要对两组人脸关键点对准处理。变脸主要信息来源是一组点内点的相对位置信息，对准中保留这些信息不丢失才行。我的处理是将两组点各自加入Graphicpath中，利用getbound函数得到包含这组点的最小矩形，然后将目标图像的矩形做平移和比例缩放变换，使得和原图像的矩形重叠。矩形内的点做同样的平移和比例缩放变换。
##三.数学原理
###3.1插值
(x',y')反变换后得到的(x,y)
####3.1.1最近邻插值
x1为和x最接近的整数，y1为和y最接近的整数，返回原图(x1,y1)处的像素值。
####3.1.2双线性插值
双线性插值是利用(x,y)周围四个点来计算其像素值。相当于在两条横边上先线性插值一次，在在插出来的纵边的两个点再线性插值一次。记i = floor(x), j = floor(y), u = x - i, v = y - j, f(i,j)表示原图像(i,j)处的像素点，则有:
$$
f(x,y)=
    \left[
       \begin{matrix}
       1 - u \\
       u
       \end{matrix}
    \right]^T
    \left[
       \begin{matrix}
       f(i,j) & f(i,j+1) \\
       f(i+1,j) & f(i+1,j+1)
       \end{matrix}
    \right] 
    \left[
       \begin{matrix}
       1 - v \\
       v
       \end{matrix}
    \right]^T
$$
####3.1.3双三次插值
双三次利用(x,y)周围的16个点来加权平均计算像素值，通常效果最好，速度最慢。同样记i = floor(x), j = floor(y), u = x - i, v = y - j, f(i,j)表示原图像(i,j)处的像素点，则有:
$$
f(x,y)=
    \left[
        \begin{matrix}
        S(u+1) \\ S(u) \\ S(u-1) \\ S(u-2)
        \end{matrix}
    \right]^T
    \left[
        \begin{matrix}
        f(i-1,j-1) & f(i-1,j) & f(i-1,j+1) & f(i-1,j+2) \\
        f(i,j-1) & f(i,j) & f(i,j+1) & f(i,j+2) \\
        f(i+1,j-1) & f(i+1,j) & f(i+1,j+1) & f(i+1,j+2) \\
        f(i+2,j-1) & f(i+2,j) & f(i+2,j+1) & f(i+2,j+2) 
        \end{matrix}
    \right]
    \left[
        \begin{matrix}
        S(v+1) \\ S(v) \\ S(v-1) \\ S(v-2)
        \end{matrix}
    \right]^T
$$
其中S(x)为三次插值基函数:
$$
S(x)=
\begin{cases}
|x|^3-2|x|^2+1 &|x|\le 1 \\
-|x|^3+5|x|^2+-8|x|+4 &1 < |x| < 2 \\
0 &otherwise
\end{cases}
$$
###3.2图像变换
####3.2.1图像旋转
(x',y')到图像中心点距离Distance，最大旋转角度$a_{max}$,扭曲旋转的半径Radius。
坐标(x',y')旋转的角度为:
$$a=a_{max}\times\frac{Radius-Distance}{Radius}$$
计算得到(x,y)的反变换公式为
$$
\begin{cases}
x=x'\cdot cosa-y'\cdot sina \\
y=x'\cdot sina+y'\cdot cosa
\end{cases}
$$
####3.2.2图像畸变
(x,y)到图像中心点距离r, (x',y')到图像中心点距离r', (x,y)经过图像畸变后变到(x',y')。
#####桶形畸变
正变换：
$$
r'=292\lg(\frac{9r}{292}+1)
$$
反变换：
$$
r=\frac{292}{9}(10^{\frac{r'}{292}-1})
$$
$$\begin{cases}
x=\frac{292}{9}(10^{\frac{\sqrt{x'^2+y'^2}}{292}-1})\cdot \frac{x'}{\sqrt{x'^2+y'^2}} \\
y=\frac{292}{9}(10^{\frac{\sqrt{x'^2+y'^2}}{292}-1})\cdot \frac{y'}{\sqrt{x'^2+y'^2}}
\end{cases}
$$
#####枕形畸变
正变换：
$$
r'=\frac{292}{9}(10^{\frac{r}{292}-1})
$$
反变换：
$$
r=292\lg(\frac{9r'}{292}+1)
$$
$$
\begin{cases}
x=292\lg(\frac{9\sqrt{x'^2+y'^2}}{292}+1)\frac{x'}{\sqrt{x'^2+y'^2}}\\
y=292\lg(\frac{9\sqrt{x'^2+y'^2}}{292}+1)\frac{y'}{\sqrt{x'^2+y'^2}}\\
\end{cases}
$$
####3.2.3TPS变脸
原图像人脸关键点为$(x_1,y_1)...(x_n,y_n)$,目标图像人脸关键点为$(x'_1,y'_1)...(x'_n,y'_n)$。
记
$$
K'=\left[
    \begin{matrix}
    0 & U(r'_{12}) & \cdots & U(r'_{1n}) \\
    U(r'_{21}) & 0 & \cdots & U(r'_{2n}) \\
    \vdots & \vdots & \ddots & \vdots  \\
    U(r'_{n1}) & U(r'_{n2}) & \cdots & 0
    \end{matrix}
\right]
P'=\left[
    \begin{matrix}
    1 & x'_1 & y'_1 \\
    1 & x'_2 & y'_2 \\
    \vdots & \vdots & \vdots \\
    1 & x'_n & y'_n
    \end{matrix}
\right]
L'=\left[\begin{matrix} K & P \\ P^T & 0 \end{matrix}\right]
$$
其中径向函数U(r)为：
$$
U(r)=\begin{cases}
r^2\lg(r^2),&r\neq0 \\
0 & r=0
\end{cases}
$$
$$
V'=\left[
    \begin{matrix} 
    x_1 & x_2 & \cdots & x_n \\
    y_1 & y_2 & \cdots & y_n 
    \end{matrix}
    \right]
Y'=\left(
    V' \middle|
    \begin{matrix}
    0 & 0 & 0 \\
    0 & 0 & 0
    \end{matrix}
    \right)^T
$$
解方程$L'[\bf w'_1,\cdots, \bf w'_n,\bf a'_1,\bf a'_x,\bf a'_y]=Y'$得$\bf w'_1,\cdots, \bf w'_n,\bf a'_1,\bf a'_x,\bf a'_y$则
$$
(x,y)={\bf a'_1}+ {\bf a'_x}x+ {\bf a'_y}y+\sum_{i=1}^{n}{{\bf w_i}U(|(x'_i,y'_i)-(x',y')|)}
$$
至此，得到反变换公式。
##四.误差分析
无模型误差
###4.1舍入误差
最终保存为int图像时，像素值为整数，误差最大值为0.5 。中间计算过程为C#的double类，精度很高，相对于最终保存图像时0.5的误差可忽略不计。所以，舍入误差为0.5。
###4.2观测误差
由最初模拟的光学信息存为int型数字图像，每个像素点位置都是选一个最接近真实颜色的像素值，所以观测误差0.5。
###4.3截断误差\方法误差
####4.3.1插值的方法误差

#####4.3.1.1最近邻插值
在数字图像中，信号是离散的，所以式子中偏导换成差分，设f(x,y)在两个方向上的一阶差分最大分别为$M_{x1}$和$M_{y1}$。
坐标舍入误差：$$\begin{cases} |\Delta x|\le0.5 \\ |\Delta y|\le0.5 \end{cases}$$
$$|\Delta A|\le max(|\frac{\partial f}{\partial x}|)|\Delta x| + max(|\frac{\partial f}{\partial y}|)|\Delta y|\le0.5\times [ max(|\frac{\partial f}{\partial x}|)+  max(|\frac{\partial f}{\partial y}|)]$$
则方法误差：$$|\Delta A|\le0.5(M_x+M_y)$$

#####4.3.1.2双线性插值
设f(x,y)在两个方向上的二阶差分最大分别为$M_{x2}$和$M_{y2}$。
双线性插值可分解为两次行上的线性插值，一次列上的线性插值。两次行的插值方法误差记为$|A_{r1}|$和$|A_{r2}|$
$$\begin{cases}|A_{r1}|\le \frac{M_{x2}}{8}\\|A_{r2}|\le \frac{M_{x2}}{8}\end{cases} $$再将行插值后得到的两点进行列方向上插值，线性组合系数为1，所以$$|\Delta A|\le \frac{M_{x2}}{8} + \frac{M_{y2}}{8}$$

#####4.3.1.3双三次插值
设f(x,y)在两个方向上的四阶差分最大分别为$M_{x4}$和$M_{y4}$。
双三性插值可分解为四次行上的三次插值，一次列上的三次插值。四次行的插值方法误差记为$|A_{r1}|$，$|A_{r2}|$，$|A_{r3}|$，$|A_{r4}|$
$$\begin{cases}
|A_{r1}|\le \frac{3}{128}M_{x4}\\
|A_{r2}|\le \frac{3}{128}M_{x4} \\
|A_{r3}|\le \frac{3}{128}M_{x4}\\
|A_{r4}|\le \frac{3}{128}M_{x4}
\end{cases} $$再将行插值后得到的四点进行列方向上插值，线性组合系数为1，所以$$|\Delta A|\le \frac{3}{128}M_{x4} + \frac{3}{128}M_{y4}$$

####4.3.2图像变换的方法误差
#####4.3.2.1图像旋转
$$
\begin{cases}
x=x'\cdot cosa-y'\cdot sina \\
y=x'\cdot sina+y'\cdot cosa
\end{cases}
$$
写出明确表达式，x'和y'无误差，计算过程中cosa和sina由于double类型与真实值之间的误差：
记$|\Delta r'|$为由于double类型的存储和真实根号后的值的误差，为$10^{-14}$到$10^{-12}$量级，$|x'|<300,|y'|<300$,所以x和y的误差不超过$10^{-10}$量级，相对于0.5可忽略不计，故可认为x和y为精确解，无方法误差。

#####4.3.2.2图像畸变
桶形畸变：
$$\begin{cases}
x=\frac{292}{9}(10^{\frac{\sqrt{x'^2+y'^2}}{292}-1})\cdot \frac{x'}{\sqrt{x'^2+y'^2}} \\
y=\frac{292}{9}(10^{\frac{\sqrt{x'^2+y'^2}}{292}-1})\cdot \frac{y'}{\sqrt{x'^2+y'^2}}
\end{cases}
$$
枕形畸变：
$$
\begin{cases}
x=292\lg(\frac{9\sqrt{x'^2+y'^2}}{292}+1)\frac{x'}{\sqrt{x'^2+y'^2}}\\
y=292\lg(\frac{9\sqrt{x'^2+y'^2}}{292}+1)\frac{y'}{\sqrt{x'^2+y'^2}}\\
\end{cases}
$$
写出明确表达式，x'和y'无误差，对于计算过程中由于double类型与真实值之间的误差：
$$
x=\frac{292}{9}(10^{\frac{r'}{292}-1})\frac{x'}{r'}
$$
记$|\Delta r'|$为由于double类型的存储和真实根号后的值的误差,为$10^{-14}$到$10^{-12}$量级，$r'<2\times 292$,有:$$
|\frac{\partial x}{\partial r'}|\le \frac{292}{9}(\frac{1}{292}10^{\frac{r'}{292}}\times 1+10^{\frac{r'}{292}}\frac{x'}{r'^2})\le\frac{292}{9}(\frac{1}{292}10^2+10^2\times 1)=3.26\times 10^3
$$
$|\frac{\partial x}{\partial r'}|\cdot|\Delta r'|$最大为$10^{-9}$量级，相比于0.5可忽略不计，基本可认为认为x和y为精确解，无方法误差。
#####4.3.2.3TPS变脸
计算各个U(r)后填充在K矩阵，double类型的存储和真实值间误差:
$$
r<300,所以|\Delta U(r)|=
|\frac{dU(r)}{dr}|\cdot|\Delta r|\le (2r+4r\lg(r))\cdot|\Delta r|\le3573|\Delta r|最大为10^{-9}量级
$$
求解矩阵的逆时使用的增广矩阵法求解，即在矩阵右侧放一个同阶的单位矩阵，经过一系列初等行列变换将原矩阵变为单位阵时，原来右侧的单位矩阵就变成了原矩阵的逆，求解过程$|\Delta U(r)|$会随行列变换而变，设求逆后每项误差平均为$|\Delta U^-(r)|$(这部分着实不知道怎么估算量级了)。$|x'|<300,|y'|<300$所以$\bf w'_1,\cdots, \bf w'_n,\bf a'_1,\bf a'_x,\bf a'_y$的误差$|\Delta S|\le|\Delta U^-(r)|\times 68\times 300\times 71$。所以反变换： 
$$
(x,y)={\bf a'_1}+ {\bf a'_x}x+ {\bf a'_y}y+\sum_{i=1}^{n}{{\bf w_i}U(|(x'_i,y'_i)-(x',y')|)}
$$
误差$|\Delta A|\le71\times |\Delta S|\cdot|300^2lg(300^2)|$
