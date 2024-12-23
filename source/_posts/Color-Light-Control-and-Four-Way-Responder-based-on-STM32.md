---
title: Color Light Control and Four Way Responder based on STM32
date: 2024-12-21 11:07:41
tags: STM32
---

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Apollo-kernel/PicGo-kernel/media53fc30198d3e4a2ab7e86567db310641.jpeg)

{% video https://cdn.jsdelivr.net/gh/Apollo-kernel/PicGo-kernel/media202412232057491.mov %}

<blockquote>
<p>
若要实现同样效果请看源码:<a href="https://gitee.com/apollo_666/Color-Light-Control-and-Four-Way-Responder">
gitee.com/apollo_666/Color-Light-Control-and-Four-Way-Responder</a>
</blockquote>

# Abstract

The design project for a decorative lighting controller enhanced our practical skills and engineering capabilities. During our study of the foundational course in electronic technology, we consistently observed the integration of engineering concepts. "The design and implementation of practical electronic circuits play a crucial role in electronic engineering technology, and the 'Fundamentals of Electronic Technology Course Design' should closely align with real-world engineering practices (P4)." In this project, we utilized ME6211C33M5G-N, STM32F103C8T6, and SSD1306 as the primary components to control various lighting sequences of decorative lights. By configuring the STM32 clock tree to select an external 8MHz crystal oscillator as the clock source and setting the Phase-Locked Loop (PLL) clock multiplier to 9, we generated a 72MHz clock frequency as the main system clock for the STM32, meeting the clock requirements of various peripherals.

#### Keywords: Decorative Lighting Controller, ME6211C33M5G-N, STM32F103C8T6, SSD1306

# 一、设计任务及主要技术指标和要求

## 1．设计任务题目

彩灯控制器

## 2．设计要求

设计并制作一个彩灯控制器，要求如下：
 1）以LED作为数码管作为控制器的显示元件，它自动地依次显示出数字0、1、2、3、4、5、6、7、8、9（自然数列），1、3、5、7、9（奇数列），0、2、4、6、8（偶数列）和1、2、3、4、5、6、7（音乐符号数列），然后又依次显示出自然数列、奇数列、偶数列和音乐符号数列……如此周而复始，不断循环。
2）打开电源时控制器可自动清零，从接通电源时刻起，数码管最先显示出自然数列的0，再显示出1，然后按上述规律变化。
3）每个数字显示的时间（从数码管显示出它之时起到它消失之时止）基本相等，这个时间在0.5~2s范围内连续可调。
4）画出总体电路图并列出元器件清单。

## 2．安装调试要求

1）完成脉冲源（震荡电路）接线，并用示波器观察并测量它的周期。
2）完成控制电路接线，并检验它的功能。
3）完成计数器电路接线，并用发光管检验计数状态。
4）完成译码器、显示电路、安装、调试。
5）整机联调并记录数据，使之满足设计要求。

## 3．发挥部分要求

(1)显示输出脉冲信号（彩灯变换）周期和频率，保留到小数点后两位
(2)使用旋钮式电位器控制输出脉冲信号周期在0.5~2s范围内连续可调，分辨率精确到0.01S
(3)实时显示使用的微控制器的温度
(4)加入4枚按键，加入抢答功能

## 4．分析、整理实验结果，写出数据总结报告。

# 二、总体方案设计

在充分了解设计要求，查阅有关资料、广开思路后我们提出了以下两种方案。
方案1：用74LS160做电路的计数部分，74LS161做四进制计数器，并分别结合三态门、74LS139译码器及NE555构成时钟频率选择电路和模式选择电路，再结合适当的门电路即可实现基本功能要求。
  方案2： 用ME6211C33M5G-N做电路的电源部分，STM32F103C8T6微控制器计数和控制彩灯模式，再结合旋钮式电位器与OLED显示屏做人机交互界面，加以按键添加抢答功能更能完成发挥部分要求。

## 1.	方案可行性分析

方案一以一片NE555构成的多谐振荡器为时钟源，产生0.5s的脉冲。D触发器构成T’ 触发器用于将时钟源信号分频，得到1s的脉冲，作为第二个时钟信号。计数器（74ls161）通过两个三态输出缓冲器选择不同的脉冲信号。对于自然数列（M0）与音乐数列（M3），一个时钟就改变一次数字，所以选用1s脉冲作为时钟信号。而奇数列（M1）和偶数列（M2）每两个时钟才改变一次数字，故采用0.5s脉冲作为时钟信号，这样才能保证每个数字显示的时间为1s。
74LS160芯片作为计数部分，打开电源时，上电复位电路产生的RD信号作为清零信号让74LS160输出为0，从而实现数码管一开始从零开始的设计要求。74LS161作为4进制计数器，结合其他的器件组成脉冲信号选择器和模式选择器。
循环控制部分采用74LS161四进制计数器。74LS160作为计数部分，其产生的进位信号与其他控制电路共同作用并产生一个进位信号（RCO），该信号决定74LS161的工作状态。当RCO=1时，74LS161开始计数，输出的Q0、Q1一方面通过异或门和三态门进行脉冲信号的选择，另一方面作为74HC139译码器的输入信号，从而产生四种模式(M0,M1,M2,M3)，即自然数列模式（M0）、奇数模式（M1）、偶数模式（M2）、音乐数模式（M3）。
最后根据不同模式对计数器（74ls161）输出的数据进行不同的处理，再接到译码器（74ls48）进行译码显示。
	方案二以一片CMOS低压差线性LDO稳压器芯片/ME6211C33M5G-N将四节干电池提供的4.8~6V直流电稳压至3.3V提供给STM32F103C8T6微控制器。通过配置STM32时钟树选择时钟源为外部8MHz晶振和设置锁相环(PLL)的时钟分频系数为9，产生72MHz时钟频率作为STM32的系统主频,以满足各外设的时钟需求。通过更改定时器预分频系数(PSC)寄存器精确调节输出时钟脉冲周期，控制数码管依次以预设时间间隔显示出自然数列、奇数列、偶数列和音乐符号数列，通过采集按键的数字信号控制彩灯序列的开始与否，通过采集旋钮式电位器和片上温度传感器的模拟信号，经过片内ADC，将连续量离散化，通过SPI总线Transmit Only Master模式与SSD1306-0.96寸OLED显示屏控制芯片通信作为人机交互界面部分，可以在OLED上显示输出脉冲信号（彩灯变换）周期和频率，四舍五入到小数点后两位，实时显示四舍五入到个位数的温度。通过加入四枚按键，可以完成发挥部分要求的抢答功能。

## 2. 方案优缺点比较

方案一所用器件成本：NE555_0.69¥+74LS74_2.9¥+74LS161_0.59¥+74LS160_0.67¥+74LS139_1.18¥+74LS48_0.45¥+74LS86_0.27¥+74LS125*2_1.22¥+74LS32_0.24¥+74LS10_0.42¥+10UF电容*2_0.48¥+0.01UF电容_0.021¥+1KΩ电阻0.015+2KΩ电阻0.021¥+35K电阻0.021¥+按键_0.08¥+共阳极数码管_0.32¥=9.588¥
方案二基本要求所用器件成本：
ME6211C33M5G-N_0.2¥+STM32F103C8T6_3.6¥+0.1uf电容*6_0.06¥+ 1uf电容*2_0.08¥+跳线帽*2_0.044¥+顶调电位器_0.69¥+20pf电容*2_0.04¥+220Ω电阻_0.16¥+10kΩ电阻_0.16¥+100kΩ电阻*2_0.32¥+8MHz无源晶振_0.231¥+按键_0.08¥+共阳极数码管_0.32¥=5.825¥
方案二发挥部分要求所用器件成本：
SSD1306_0.96寸OLED显示屏_13.5¥+按键*4_0.56¥+按键帽_0.2¥+发光红色LED_0.025¥+发光翠绿LED_0.04¥+510Ω电阻*2_0.032¥+220Ω电阻_0.48¥+绿色共阳极数码管_0.55¥+白色共阳极数码管_0.6¥+蓝色共阳极数码管_0.55¥=16.537¥
综上，我们最终决定选用方案二。	


## 3. 总体方案需结合的参考电路

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Apollo-kernel/PicGo-kernel/mediaf2dbcb52be7b433b83916972492eaa77.png)

## 4. 总体方案原理框图

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Apollo-kernel/PicGo-kernel/media3e93a27742de4bedbd2abc4c231dbfd0.png)




# 三、单元电路的设计

## 1. 电源电路

ME6211C33M5G-N是一颗CMOS低压差线性LDO集成稳压芯片,属于大规模集成电路(LSI)，其工作电压范围是2V~6V，最大输出电流为500mA (Vin=4.3, Vout=3.3) ,待机模式下电流为0.1uA。能够将四节干电池提供的4.8~6V直流电稳压至3.3V提供给微控制器。

CMOS即 complementary MOSFET，互补型MOSFET，在大规模集成电路里面，NMOS和PMOS被集成在一起，通过同一个信号来控制，从而实现数字信号的逻辑NOT功能。这种结构是组成集成电路的基础单元。

开关电源与线性稳压电源(LDO)是电源的两种主要类型，线性稳压电源的电压反馈电路工作在线性区，开关电源是指用于电压调整的管子工作在饱和区和截至区即开关状态的。它们各有各的优点和应用场合。

LDO技术很成熟，制作成本较低，可以达到很高的稳定度，纹波也很小，而且没有开关电源具有的干扰与噪音，但是效率低，效率低意味着发热量大，相当于把多余的压差转化成了热量，适用于供给微控制器作为电源。

线性电源一般是将输出电压取样然后与参考电压送入比较电压放大器，此电压放大器的输出作为电压调整管的输入，用以控制调整管使其结电压随输入的变化而变化，从而调整其输出电压。开关电源是通过改变调整管的开和关的时间即占空比来改变输出电压的。开关电源效率高、损耗小、可以降压也可以升压，但是交流纹波稍大些，适用于高的压差转化。

“M5”代表该芯片封装形式是SOT23-5，允许最大功率Pd=0.6W。
“G” 是该芯片是环保标识。
“-N”代表该芯片适用于中等元件密度的产品。
L、N、M表示焊盘伸出的大小的几何形状变化。
L = Least Use Environment (Level C - High Density)，密度等级L：
最小焊盘伸出——适用于焊盘图形具有最小的焊接结构要求的微型器件，可实现最高的元件组装密度。

N = Nominal Use Environment (Level B - Medium density), 密度等级N：
中等焊盘伸出——适用于中等元件密度的产品，提供坚固的焊接结构。

M = Most Use Environment (Level A - Low Density),密度等级M：
最大焊盘伸出——适用于高元件密度应用中，典型的像便携/手持式或暴露在高冲击或震动环境中的产品。焊接结构是最坚固的，并且在需要的情况下很容易进行返修。

电源电路对于一个控制系统来说极其重要，关系到整个系统是否能够正常工作， 因此选择了ME6211C33M5G-N这款LDO为电源电路的核心。
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Apollo-kernel/PicGo-kernel/mediae0de4317988946e19c316b91dd71462b.png)

输入滤波电容可以对输入电流滤波，同时防止断电后出现电压倒置。

输出滤波电容可以抑制自激震荡和稳定输出电压。稳压电路的工作过程需要从输出采样，然后根据其反馈值调节输出以达稳压的目的。如果此时没有输出滤波电容，只要因负载变化带来的电压波动频率恰好与稳压电路的调节速率差不多就会产生振荡效应，导致输出失控，所以稳压输出也必须加滤波电容，而且增加滤波电容也可以进一步增加稳压输出的稳定性。

大小电容并联时，大电容滤除低频噪声，小电容滤除高频噪声，电容值和需要滤除噪声频率的平方成反比。

## 2. 微控制器电路

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Apollo-kernel/PicGo-kernel/media8e3352ba05dd4e1aa1b3ce84a3d8b659.png)
退耦电容并接于放大电路的电源正负极之间，防止由电源内阻形成的正反馈而引起的寄生振荡。
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Apollo-kernel/PicGo-kernel/media15b9e0a86d5746078583a65630f2f6ab.png)
20pF起振电容是8MHz无源晶振正常震荡需要匹配的电容电容无源晶振和电容需尽可能近地靠近微控制器的引脚，以减小输出失真和启动稳定时间，线路太长会增加寄生电容，而且容易发生串扰，而且会影响其他信号线；其他信号线需远离晶振线，因为晶振线路信号跳动频繁，产生的磁场不断变化，附近的线易受到干扰。
STM32F103C8T6是ST公司的一款ARM Cortex-M3内核的32位微控制器, 封装为LQFP48, 采用0.18μm高度集成的CMOS工艺制造，芯片面积不超过7mm*7mm,属于超大规模集成电路(VLSI)，与早期的SSI、MSI和LSI芯片相比,它具有功能更强、体积更小和功耗更低的显著优势。微控制器芯片上集成了CPU核心、SRAM存储器、Flash闪存、定时器、外设接口等复杂的数字电路,元件数远远超过10万个，它出现标志着集成电路技术的发展达到较高水平，随着技术的不断进步,未来微控制器的集成度还将进一步提高。
此微控制器所有引脚在非ADC模式下均可承受5V电压，所有引脚都可以提供最高25mA的拉电流或灌电流。在ADC转换范围为0至3.6V，内置温度传感器。引脚总共可提供最高120mA的拉电流或灌电流，3.3V引脚输出不超过500mA的电流，72MHz下工作最大电流32.8mA，最低工作电流可以达到5μA。
对于电路设计人员来说，微控制器应该被视为一个电路元件，就像运算放大器一样，有时甚至更便宜。此次设计用到的片内集成外设有ADC和SPI总线，数据从外设到内存和从内存到外设的传递均采用DMA(Direct Memory Access)即直接存储器访问方式搬运。

## 3. 彩灯显示电路

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Apollo-kernel/PicGo-kernel/media32f64246a2f942c7abebd21afdbc0c47.png)
数码管按发光二极管单元连接方式可分为共阳极数码管和共阴极数码管。
共阴数码管指将所有发光二极管的阴极接到一起形成公共阴极(COM)的数码管，共阴数码管在应用时应将公共极COM接到地线GND上，当某一字段发光二极管的阳极为高电平时，相应字段就点亮，当某一字段的阳极为低电平时，相应字段就不亮。
共阳数码管指将所有发光二极管的阳极接到一起形成公共阳极(COM) 的数码管，共阳数码管在应用时应将公共极COM接到高电平，当某一字段发光二极管的阴极为低电平时，相应字段就点亮， 当某一字段的阴极为高电平时，相应字段就不亮。
如果采用共阴极数码管，由于它的驱动端在非公共端，个数码管需要外加八个上拉电阻。
如果采用共阳极数码管，由于它的驱动端在公共端，控制一只需要外加一个上拉电阻。
所以，我们选择采用共阳极数码管。
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Apollo-kernel/PicGo-kernel/mediac6ff4533d235405cb3704d7b22ebd3da.png)
鉴于八个发光二极管总电流在160mA左右，我们选用阻值为220Ω的电阻为上拉电阻,（3.3-2.1）/220=5.45mA，经过测试，彩灯光强满足需求。
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Apollo-kernel/PicGo-kernel/media5bbc3b2e587c45f28eead15c6ee205db.png)

## 4. 顶调电位器电路

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Apollo-kernel/PicGo-kernel/media3d1bbca914624742a9a9d880ede940fd.png)
模拟地AGND和数字地 DGND采用单点接地，由于星形接地会在一点上连接两种接地，这样高噪声数字电流都会流过数字电源，一直流到数字接地平面，并回到数字电源，同时与敏感的模拟电路隔离。
我们用的是72MHZ主频的微控制器，72MHz的HCLK经6分频得到ADC_Clock = 72/6 = 12MHz。12位ADC最低转换周期 = 1.5(min Sampling Time)+12=13.5个周期 = 13.5*(1/12MHz) = 1.125us > 1us(最小转换周期) 。12位ADC最高转换速率 = 1/1.125us = 0.88MSPS < 1MSPS(ADC最高转换速率)
我们的微控制器的型号是STM32F103C8T6，片上共2个ADC，最高12位ADC，最高转换速率1MSPS（最小转换周期1us）
AD转换周期=采样时间+转化时间
AD转换速率=转换周期的倒数
我们启用了一个12位模数转换器，使能了双通道12位模数转换，使能了连续转换模式，电位器电压与内部温度传感器的 Sampling Time均设置为239.5 Cycles(周期)。
所以双通道12位模数转换器的转换周期为
239.5*2+12=491个周期 = 491*(1/12MHz) = 40.917us
我们的中值平均滤波方案是将30个周期的一个通道的30个数据去除最大值和最小值后求平均，这样转换后的数据的更新周期为
30*40.917=1.228ms
为了使输出脉冲信号周期在0.5~2s范围内连续可调，分辨率精确到0.01S，我们将这连续可调的1.5S分为150份。因为12位ADC的分辨率是1/4096，为了结合实际，我们的控制区域选用了40~4090这4050份，4050/150=27，所以我们以27份为一个调节单位，更改脉冲周期。
在查阅数据手册后，我们得出内部温度传感器的温度是
(1.43-采集到的值*0.0008057)/ 0.0043+25 摄氏度
ADC转换完成的数据从外设到内存的传递采用DMA(Direct Memory Access)即直接存储器访问方式搬运，提高了系统效率。

## 5. 按键电路

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Apollo-kernel/PicGo-kernel/media6396bf31570d4626a0e0d2a8114d18e8.png)
在配置STM32F103C8T6微控制器的管脚为上拉输入模式后，通过软件状态机消抖方案即可控制彩灯序列的起始和实现抢答功能。

## 6. OLED显示电路

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Apollo-kernel/PicGo-kernel/media38fbdf7268aa49df897c85096c8d301c.png)
SPI总线Transmit Only Master模式的数据从内存到外设的传递采用DMA(Direct Memory Access)即直接存储器访问方式搬运
SSD1306芯片作为核心存在于0.96寸OLED显示屏模块内，显示屏作为人机交互界面部分，可以显示输出脉冲信号（彩灯变换)周期和频率，四舍五入到小数点后两位，实时显示四舍五入到个位数的温度。

# 四、整体电路图 

### 1. 原理图

![1. 原理图](https://cdn.jsdelivr.net/gh/Apollo-kernel/PicGo-kernel/media913cdd2559af4013a67221e278d153cc.png)
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Apollo-kernel/PicGo-kernel/media6b1f4396749b4d94b8d41b3718d963fe.png)
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Apollo-kernel/PicGo-kernel/media202412232026326.png)
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Apollo-kernel/PicGo-kernel/media202412232027168.png)
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Apollo-kernel/PicGo-kernel/media202412232027470.png)

### 2. 整体装配图

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Apollo-kernel/PicGo-kernel/media202412232028515.jpeg)

### 3.	元器件明细表

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Apollo-kernel/PicGo-kernel/media202412232028628.png)

# 五、实际电路指标性能测试

功能正常

# 六、	对设计成果的评价

### 1.	本设计的特点

工作稳定可靠，保证能达到所要求的性能指标，并留有裕量；电路简单；成本低；耗电少；所采用的元器件品种少，体积小且货源充足；便于生产，容易调试；维修方便。

### 2. 本设计存在的问题

按键部分稳定性不够强，在抢答时按键不灵敏，平均按下10下有一下识别不到，可能是面包板线路接触不良。

### 3.	本设计改进的建议

在按键电路的按键上并联一个0.1uF的电容以实现硬件消抖。

# 七、参考文献

【1】电子技术基础.数字部分/华中科技大学电子技术课程组编；康华光，张林主编. -7版.-北京：高等教育出版社，2021.8
【2】模拟电子技术基础/钟化兰，周霞主编.-成都：西南交通大学出版社，2021.5
【3】数字电子技术试验与课程设计/钟化兰主编.-西安：西北工业大学出版社，2015.8
【4】The Art of Electronics/ Paul Horowitz, Winfield Hill主编. -3版-纽约：CAMBRIDGE UNIVERSITY PRESS，2015
【5】ADI-PCB布局布线要点/ ADI主编.- Analog Devices, Inc，2021





# Markdown 是什么？

Markdown 是一种轻量级的标记语言，可用于在纯文本文档中添加格式化元素。Markdown 由 John Gruber 于 2004 年创建，如今已成为世界上最受欢迎的标记语言之一。

<p>1.专注于文字内容；<br>2.纯文本，易读易写，可以方便地纳入版本控制；<br>3.语法简单，没有什么学习成本，能轻松在码字的同时做出美观大方的排版。</p>    使用 Markdown 与使用 Word 类编辑器不同。在 Word 之类的应用程序中，单击按钮以设置单词和短语的格式，并且，更改立即可见。而 Markdown 与此不同，当你创建 Markdown 格式的文件时，可以在文本中添加 Markdown 语法，以指示哪些单词和短语看起来应该有所不同。</p>
