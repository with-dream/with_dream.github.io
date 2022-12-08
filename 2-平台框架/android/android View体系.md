### 一、窗口系统
1、window类型：
系统Window		低电量、输入法弹框等
应用程序Window
子Window			PopWindow，Dialog

2、z序
WindowState是WMS中事实的窗口，是在WMS的addWindow方法中创建，包含了一个窗口的所有的属性
其中一个属性为mLayer，表示窗口在Ｚ轴的位置，mLayer值越小，窗口越靠后
