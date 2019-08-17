---
title: IDEA显示Run Dashboard窗口
categories: IDEA相关
tags: IDEA相关
date: 2019-08-10 19:49:51
---

方法一：(好用)

找到项目中.idea文件下的workspace.xml开打　　　 
     
     接下来找到 
     <component name="RunDashboard"> 
     代码中加入
     
     <option name="configurationTypes">  
          <set>
             <option value="SpringBootApplicationConfigurationType" />  
          </set>  
     </option>  
这样Run Dashboard这个窗口就可以弹出来了。

方法二：
打开该项目后，再打开一次（不关闭之前打开的项目）。
即依次点击“File -> Open”，选择项目。
打开时，会弹出提示"Multiple Spring Boot run configurations were detected. Run Dashboard allows to manage multiple run configurations at once."。见下图：

![2019-08-10-20-40-40.jpg](http://pv125k1gl.bkt.clouddn.com/2019-08-10-20-40-40.jpg)

点击“Show run configurations in Run Dashboard”，"Run Dashboard"面板重新在底部区域展示了出来。
一般有时候创建springboot项目的时候右下角可以提示你打开Run Dashboard，但是如果不提醒就需要自己配置了。

