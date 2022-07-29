---
sidebar_position: 1
---

<a name="oHZmO"></a>

![image.png](https://cdn.nlark.com/yuque/0/2022/png/29660210/1658239739247-c3cc10aa-666c-4e51-9eb2-e14dc50f7689.png#clientId=ub3d8e3b8-e54a-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uad9fd83b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=426&originWidth=640&originalType=url&ratio=1&rotation=0&showTitle=false&size=460640&status=done&style=none&taskId=u9d5d36c9-dd66-48c9-8c33-b3c4621b4cd&title=)
<a name="WqiFM"> </a>
## 引言

一般PCB基本设计流程如下：<br /> **前期准备**->**原理图设计**->**PCB结构设计**->**PCB布局**->**布线（布线设计规范具体应用来设计）**->**布线优化和丝印**->**网络和DRC检查和结构检查**->**制版**
<a name="eDumU"></a>
## 前期准备

准备**元件库**和**原理图**，在进行PCB设计之前，首先要准备好原理图SCH的元件库和PCB的元件库。元件库可以用peotel自带的库，但一般情况下很难找到合适的，最好是自己根据所选器件的标准尺寸资料自己做元件库。原则上先做PCB的元件库，再做SCH的元件库。PCB的元件库要求较高，它直接影响板子的安装；SCH的元件库要求相对比较松，只要注意定义好管脚属性和与PCB元件的对应关系就行。PS：注意标准库中的隐藏管脚。之后就是原理图的设计，做好后就准备开始做PCB设计了。

<a name="dAH51"></a>
## 原理图设计

在设计原理图之前，可以去网上参考Github、CSDN、立创开源广场等平台中的开源项目。这样可以使我们事半功倍。<br />在设计原理图时，最好列举好自己要的功能、器件选好型、购买是否方便（防止设计了市面上已经绝版的芯片主要还是价格）<br />**一定要看芯片手册！！**<br />在芯片手册中一定会给出引脚功能，设计注意事项（器件供电电压范围等），有些厂商还会有参考设计在里面，这些对我们都会有帮助。

<a name="NsMkc"></a>
## PCB结构设计

这里主要是电路板**尺寸**和各项**机械定位，**在PCB设计环境下绘制PCB板面，并按定位要求放置所需的接插件、按键/开关、螺丝孔、装配孔等等。并充分考虑和确定布线区域和非布线区域（如螺丝孔周围多大范围属于非布线区域）。
<a name="kJVQ6"></a>
## PCB布局

按照软件的飞线提示进行大致排列，同一电路的器件摆放在一起。**一定要看着原理图进行连线操作，不要只看网络！！**

---

1. 电气分类

一般分为：**数字电路区(即怕干扰、又产生干扰)**、**模拟电路区(怕干扰)**、**功率驱动区（干扰源）**；

2. 完成同一功能的电路，应尽量靠近放置
2. 对于**质量大**的元器件应考虑安装位置和安装强度；**发热元件**应与**温度敏感**元件分开放置，必要时还应考虑热对流措施；
2. **I/O驱动器件**尽量靠近印刷板的边、靠近引出接插件；
2. **时钟产生器**（如：**晶振**或**钟振**）要尽量靠近用到该时钟的器件；
2. 在每个集成电路的电源输入脚和地之间，需加一个**去耦电容**（一般采用高频性能好的**独石电容**）；电路板空间较密时，也可在几个集成电路周围加一个**钽电容**。
2. **继电器线圈**处要加**放电二极管**（1N4148即可）；
2. 布局要求要均衡，疏密有序，不能头重脚轻或一头沉

前提下，适当修改器件的摆放，使之整齐美观，如同样的器件要摆放整齐、方向一致，不能摆得“错落有致”。这个步骤关系到板子整体形象和下一步布线的难易程度，所以一点要花大力气去考虑。布局时，对不太肯定的地方可以先作初步布线，充分考虑。
<a name="uLE9y"></a>
## 布线

布线是整个PCB设计中最重要的工序。这将直接影响着PCB板的性能好坏。在PCB的设计过程中，布线一般有这么三种境界的划分：首先是布通，这时PCB设计时的最基本的要求。如果线路都没布通，搞得到处是飞线，那将是一块不合格的板子，可以说还没入门。其次是电器性能的满足。这是衡量一块印刷电路板是否合格的标准。这是在布通之后，认真调整布线，使其能达到最佳的电器性能。接着是美观。假如你的布线布通了，也没有什么影响电器性能的地方，但是一眼看过去杂乱无章的，加上五彩缤纷、花花绿绿的，那就算你的电器性能怎么好，在别人眼里还是垃圾一块。这样给测试和维修带来极大的不便。布线要整齐划一，不能纵横交错毫无章法。这些都要在保证电器性能和满足其他个别要求的情况下实现，否则就是舍本逐末了。布线时主要按以下原则进行：

---

**1、布线优先次序**<br />** 关键信号线优先**：电源、摸拟小信号、高速信号、时钟信号和同步信号等关键信号优先。<br />**布线密度优先原则**：从单板上连接关系最复杂的器件着手布线。从单板上连线最密集的区域开始布线。<br />**关键信号处理注意事项**：尽量为时钟信号、高频信号、敏感信号等关键信号提供专门的布线层，并保证其最小的回路面积。必要时应采取屏蔽和加大安全间距等方法。保证信号质量。<br />**有阻抗控制要求的网络应布置在阻抗控制层上，须避免其信号跨分割。**<br />**2、布线窜扰控制**<br />**3W原则**释义<br />线与线之间的距离保持3倍线宽。是为了减少线间串扰，应保证线间距足够大，如果线中心距不少于3倍线宽时，则可保持70%的线间电场不互相干扰，称为3W规则。或者也可以是2倍的线宽。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/29660210/1658241619082-ed43fdb9-1117-4d65-bc0c-e4ee3e1cba53.png#clientId=ua4ca5e70-8101-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=udc07ca8c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=122&originWidth=482&originalType=url&ratio=1&rotation=0&showTitle=false&size=8164&status=done&style=none&taskId=uf3b9ec1b-9e72-431f-ba51-876788dd828&title=)<br />**窜扰控制**：串扰（CrossTalk)是指PCB上不同网络之间因较长的平行布线引起的相互干扰，主要是由于平行线间的分布电容和分布电感的作用。克服串扰的主要措施是：<br />i. 加大平行布线的间距，遵循3W规则；<br />ii. 在平行线间插入接地的隔离线<br />iii. 减小布线层与地平面的距离。<br />**3、布线的一般规则要求**<br />**相邻平面走线方向成正交结构**<br />避免将不同的信号线在相邻层走成同一方向，以减少不必要的层间窜扰；当由于板结构限制（如某些背板）难以避免出现该情况，特别是信号速率较高时，应考虑用地平面隔离各布线层，用地信号线隔离各信号线。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/29660210/1658241720661-a455c13f-673c-4255-9a0d-73d3317fce0e.png#clientId=ua4ca5e70-8101-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u5479f59e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=161&originWidth=331&originalType=url&ratio=1&rotation=0&showTitle=false&size=44575&status=done&style=none&taskId=u87fa04dd-2511-40e3-a6fe-88613b4940c&title=)<br />**小的分立器件走线须对称**，间距比较密的SMT焊盘引线应从焊盘外部连接，不允许在焊盘中间直接连接。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/29660210/1658241720689-421b75e0-81bb-48c8-a79a-b8c21df2b564.png#clientId=ua4ca5e70-8101-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ua3983bfc&margin=%5Bobject%20Object%5D&name=image.png&originHeight=264&originWidth=811&originalType=url&ratio=1&rotation=0&showTitle=false&size=32592&status=done&style=none&taskId=udf1b80d4-098f-4921-8ef8-5dc1788a080&title=)<br />**环路最小规则**，即信号线与其回路构成的环面积要尽可能小，环面积越小，对外的辐射越少，接收外界的干扰也越小。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/29660210/1658241720703-953946c9-afa8-4b0a-8460-cf97487d87a5.png#clientId=ua4ca5e70-8101-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ufe3d7bae&margin=%5Bobject%20Object%5D&name=image.png&originHeight=434&originWidth=457&originalType=url&ratio=1&rotation=0&showTitle=false&size=33864&status=done&style=none&taskId=ufa7db3a8-4539-40c8-9ccd-ca9b7e32592&title=)<br />**走线不允许出现STUB**<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/29660210/1658241720750-572982e1-fa84-4075-a1db-69a380444d9e.png#clientId=ua4ca5e70-8101-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u6257eb64&margin=%5Bobject%20Object%5D&name=image.png&originHeight=473&originWidth=782&originalType=url&ratio=1&rotation=0&showTitle=false&size=29381&status=done&style=none&taskId=ud37e3739-f210-4054-b611-1fc274b75b0&title=)<br />**同一网络的布线宽度应保持一致**，线宽的变化会造成线路特性阻抗的不均匀，当传输的速度较高时会产生反射。在某些条件下，如接插件引出线，BGA封装的引出线类似的结构时，因间距过小可能无法避免线宽的变化，应该尽量减少中间不一致部分的有效长度。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/29660210/1658241720747-d7dfb9e0-b56e-44f3-ba8a-868a156e441a.png#clientId=ua4ca5e70-8101-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uea802552&margin=%5Bobject%20Object%5D&name=image.png&originHeight=462&originWidth=497&originalType=url&ratio=1&rotation=0&showTitle=false&size=17432&status=done&style=none&taskId=uade0675b-c126-461e-a4cf-68595fc0318&title=)<br />**防止信号线在不同层间形成自环**。在多层板设计中容易发生此类问题，自环将引起辐射干扰。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/29660210/1658241721106-476ed2a3-92ff-46e1-9d49-6bba31e1e839.png#clientId=ua4ca5e70-8101-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ue0b2ce7f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=445&originWidth=952&originalType=url&ratio=1&rotation=0&showTitle=false&size=27717&status=done&style=none&taskId=u558aba50-3112-43fb-8d77-d72552d9cd5&title=)<br />**PCB设计中应避免产生锐角和直角**，产生不必要的辐射，同时PCB生产工艺性能也不好。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/29660210/1658241721251-a7174fb1-95af-4cae-93c1-b3d61275f3b3.png#clientId=ua4ca5e70-8101-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uc0cc6b46&margin=%5Bobject%20Object%5D&name=image.png&originHeight=333&originWidth=1022&originalType=url&ratio=1&rotation=0&showTitle=false&size=18972&status=done&style=none&taskId=ud134cf88-cd70-4b09-93ef-2bc82e49582&title=)<br />**4、关于布线线宽以及铺铜设计有关事项**<br />我们首先要指导厂家可以生产怎么样的板子，设计的极限在哪里。以嘉立创为例：

| <br />最小线宽/线隙（1OZ） | 单双面板：0.127/0.127mm(5mil/5mil) | ![image.png](https://cdn.nlark.com/yuque/0/2022/png/29660210/1658242192000-2a1807ea-fbe8-4e39-9d82-31fc718acdff.png#clientId=ua4ca5e70-8101-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u6772bd4f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=104&originWidth=193&originalType=url&ratio=1&rotation=0&showTitle=false&size=7727&status=done&style=none&taskId=uc6c65fd2-4329-4db8-a4ec-2070b56a360&title=) |  |
| --- | --- | --- | --- |
|  | 多层板：0.09/0.09mm(3.5mil/3.5mil) |  |
| 最小线宽/线隙（2OZ） | 单双面及多层板：0.2/0.2mm(8mil/8mil) |  |
| 线宽公差 | ±20% | 例如线宽0.1mm，实板线宽为0.08-0.12mm是合格允许的 |  |
| 焊盘边到线边间距 | ≧0.127mm(极限值)，尽量大于此参数 | ![image.png](https://cdn.nlark.com/yuque/0/2022/png/29660210/1658242192001-f69751df-6dcf-4fad-a554-fb36a15f0f19.png#clientId=ua4ca5e70-8101-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ub8f885e2&margin=%5Bobject%20Object%5D&name=image.png&originHeight=104&originWidth=193&originalType=url&ratio=1&rotation=0&showTitle=false&size=12436&status=done&style=none&taskId=u0bcf7ad7-2a0a-4433-a60e-7293b8722d0&title=) |  |
| 有铜插件孔焊环 | ≧0.25mm(建议值)，极限值为0.18mm，下单提出评审 | ![image.png](https://cdn.nlark.com/yuque/0/2022/png/29660210/1658242192029-6ff52d6c-92e3-4a2c-b274-ea2949a96647.png#clientId=ua4ca5e70-8101-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uf9f67e1a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=107&originWidth=192&originalType=url&ratio=1&rotation=0&showTitle=false&size=11418&status=done&style=none&taskId=ub53f2eb5-0411-400b-93a1-80e1a00000c&title=) |  |
| 无铜插件孔焊环 | ≧0.45mm(建议值)，因为采用干膜封孔，无铜孔周围会掏空0.2MM的焊盘或铜面，请尽量加大焊盘以便焊接，焊盘过小可能就是一个线圈或无焊盘 | ![image.png](https://cdn.nlark.com/yuque/0/2022/png/29660210/1658242192033-0eb059d3-618c-43a3-b824-1e35ab615548.png#clientId=ua4ca5e70-8101-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ud03c745c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=111&originWidth=190&originalType=url&ratio=1&rotation=0&showTitle=false&size=20117&status=done&style=none&taskId=u90c457eb-32df-4923-99af-dc8cba321e9&title=) |  |
| BGA | ① BGA焊盘直径：≧0.25mm<br />② BGA焊盘边到线边：≧0.127mm<br />③ 暂不制作BGA焊盘中间钻孔的盘中孔工艺（油墨没法塞孔） | ![image.png](https://cdn.nlark.com/yuque/0/2022/png/29660210/1658242192007-73c5f0b3-aafe-4896-99ee-8768272af91a.png#clientId=ua4ca5e70-8101-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u62b15045&margin=%5Bobject%20Object%5D&name=image.png&originHeight=129&originWidth=150&originalType=url&ratio=1&rotation=0&showTitle=false&size=17906&status=done&style=none&taskId=u995e6613-a719-4858-ab49-a5e78a50576&title=) |  |
| 阻焊 | 阻焊开窗 | 开窗比焊盘单边≧0.05mm，开窗距线边间距≧0.07mm | ![image.png](https://cdn.nlark.com/yuque/0/2022/png/29660210/1658242192453-58e4e744-524a-4435-8904-c9070fc80b68.png#clientId=ua4ca5e70-8101-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uf606a4d9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=100&originWidth=183&originalType=url&ratio=1&rotation=0&showTitle=false&size=14050&status=done&style=none&taskId=u39d21086-6bd0-4b78-b61e-eb4e4193fda&title=) |
|  | 阻焊厚度 | ≧10um | ![image.png](https://cdn.nlark.com/yuque/0/2022/png/29660210/1658242192457-59d9de83-bc31-4aca-b841-8295d77f5ba3.png#clientId=ua4ca5e70-8101-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=uca6c77f5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=80&originWidth=227&originalType=url&ratio=1&rotation=0&showTitle=false&size=9408&status=done&style=none&taskId=u7dba5469-1ef6-43f5-896c-69e9cffc3fa&title=) |
|  | 过孔塞油 | 用阻焊油墨塞进过孔达到不透光效果([查看详情说明](https://www.sz-jlc.com/portal/server_guide_15544.html)):<br />① 只有双面焊盘盖油的过孔才能塞油<br />② 塞油的过孔孔径尽量≦0.5mm<br />③孔边到开窗焊盘边≤0.35mm的过孔，不便塞油 | ![image.png](https://cdn.nlark.com/yuque/0/2022/png/29660210/1658242192564-47ba5f4e-47f7-4c00-b4e0-1e5267fcee34.png#clientId=ua4ca5e70-8101-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=ue6543551&margin=%5Bobject%20Object%5D&name=image.png&originHeight=139&originWidth=258&originalType=url&ratio=1&rotation=0&showTitle=false&size=15101&status=done&style=none&taskId=udfeccaa8-086f-4ec4-a08c-cf2ba5d0368&title=) |
|  | 最小阻焊桥宽度 | 0.1mm（黑油和白油0.13mm）。 [查看详情说明](https://www.sz-jlc.com/portal/server_guide_34402.html) | 绿油极限最小可以做0.08mm阻焊桥，需客户下单时备注好 |
| 字符 | 字符高度 | ≧1mm（特殊字体，中文，掏空字符视情况需更高） | ![image.png](https://cdn.nlark.com/yuque/0/2022/png/29660210/1658242192569-ad213b1b-6388-4787-bee7-d69ff0a19763.png#clientId=ua4ca5e70-8101-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u75528e56&margin=%5Bobject%20Object%5D&name=image.png&originHeight=102&originWidth=256&originalType=url&ratio=1&rotation=0&showTitle=false&size=17812&status=done&style=none&taskId=u8f5f60c6-2bba-4ab5-ba51-d582ddb8185&title=) |
|  | 字符粗细 | ≧0.15mm（低于此值可能印不出来） |  |
|  | 字符到露铜焊盘间隙 | ≧0.15mm（低于此值会掏空字符避免上焊盘） |  |

嘉立创链接：[https://www.jlc.com/portal/vtechnology.html](https://www.jlc.com/portal/vtechnology.html)<br />我们要知道**信号线越细越好，电源越粗越好。**<br />**5、布线优化和丝印**<br />“没有最好的，只有更好的”！不管你怎么挖空心思的去设计，等你画完之后，再去看一看，还是会觉得很多地方可以修改的。一般设计的经验是：优化布线的时间是初次布线的时间的两倍。感觉没什么地方需要修改之后，就可以铺铜了（Place->polygonPlane）。铺铜一般铺地线（注意模拟地和数字地的分离），多层板时还可能需要铺电源。时对于丝印，要注意不能被器件挡住或被过孔和焊盘去掉。同时，设计时正视元件面，底层的字应做镜像处理，以免混淆层面。<br />**6、网络和DRC检查和结构检查**<br />首先，在确定电路原理图设计无误的前提下，将所生成的PCB网络文件与原理图网络文件进行物理连接关系的网络检查（NETCHECK），并根据输出文件结果及时对设计进行修正，以保证布线连接关系的正确性；网络检查正确通过后，对PCB设计进行DRC检查，并根据输出文件结果及时对设计进行修正，以保证PCB布线的电气性能。最后需进一步对PCB的机械安装结构进行检查和确认。<br />**7、制版**<br />在此之前，最好还要有一个审核的过程。<br />PCB设计是一个考心思的工作，谁的心思密，经验高，设计出来的板子就好。所以设计时要极其细心，充分考虑各方面的因数（比如说便于维修和检查这一项很多人就不去考虑），精益求精，就一定能设计出一个好板子。

