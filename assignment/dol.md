### Lab 3 - DOL实例分析&编程

#### 1.改完的*.dot截图

- example1:
  ![](https://raw.githubusercontent.com/junweiHuang45/learngit/master/lab3/1.png)


- example2:
  ![](https://raw.githubusercontent.com/junweiHuang45/learngit/master/lab3/2.png)

#### 2. 具体修改的步骤

##### 实验要求：

1. 修改example2，让3个square模块变成2个。

   ​	在 /dol/examples/example2/example2.xml中，<variable value="3" name="N"/> 改成<variable value="2" name="N"/> 即可。

   ​	因为这个是循环标志，value的值表示循环的次数。

2. 修改example1，使其输出3次方数

   ​	在 /dol/examples/example1/src/square.c 中，在函数int square_fire(DOLProcess *p) 中，将 i=i\*i 改成 i=i\*i\*i 即可。

   ​需要注意的是，每次更改好了的example，都需要把 /dol/build/bin/main 当中对应的example文件夹给删除掉。例如如果修改的是example1，就需要把example1的文件夹删除好在运行结果，否则无法得出正确答案。



#### 3.运行结果

1.  example1运行结果：
    ![](https://raw.githubusercontent.com/junweiHuang45/learngit/master/lab3/3.png)
    ​
2.  example2运行结果：
    ![](https://raw.githubusercontent.com/junweiHuang45/learngit/master/lab3/4.png)
    ​

#### 4. 实验感想

​	这次实验要求改的部分比较简单，但是看懂整个代码还是需要下一些功夫的。

​	在整个DOL的example里，分成三大部分，一个是生产者，一个是消费者，另一个是连接模块，生产者和消费者定义在src的各个文件中，而连接模块定义在 exampleX.dol当中。

​	在example1中，generotor.c是一个生产者的代码，而consumer.c是消费者的代码，square.c是定义了平方进程的代码，如果想让其做立方操作，即把 i=i\*i 修改即可。

​	在example2中，我好好地分析了一下.xml的代码，



```DOL、
 <!-- instantiate resources -->

  <process name="generator">

<port type="output" name="10"/>
<source type="c" location="generator.c"/>
  <!-- instantiate resources -->
  <process name="generator">
    <port type="output" name="10"/>
    <source type="c" location="generator.c"/>
  </process>

  <iterator variable="i" range="N">
    <process name="square">
      <append function="i"/>
      <port type="input" name="0"/>
      <port type="output" name="1"/>
      <source type="c" location="square.c"/>
    </process>
  </iterator>

  <process name="consumer">
    <port type="input" name="100"/>
    <source type="c" location="consumer.c"/>
  </process>

  <iterator variable="i" range="N + 1">
    <sw_channel type="fifo" size="10" name="C2">
      <append function="i"/>
      <port type="input" name="0"/>
      <port type="output" name="1"/>
    </sw_channel>
  </iterator>
```
在resource部分，首先设置了开头的generator和结尾consumer，并且用迭代的方式创建了C2\_0, C2\_1,… 和square\_0, square\_1,...



      <!-- instantiate connection -->
      <iterator variable="i" range="N">
    
    <connection name="to_square">
      <append function="i"/>
      <origin name="C2">
        <append function="i"/>
        <port name="1"/>
      </origin>
      <target name="square">
        <append function="i"/>
        <port name="0"/>
      </target>
    </connection>
    
    <connection name="from_square">
        <append function="i"/>
        <origin name="square">
          <append function="i"/>
          <port name="1"/>
        </origin>
        <target name="C2">
          <append function="i + 1"/>
          <port name="0"/>
        </target>
    </connection>
       <connection name="from_square">
            <append function="i"/>
            <origin name="square">
              <append function="i"/>
              <port name="1"/>
            </origin>
            <target name="C2">
              <append function="i + 1"/>
              <port name="0"/>
            </target>
        </connection>
      </iterator>
    
      <connection name="g_">
        <origin name="generator">
         <port name="10"/>
        </origin>
        <target name="C2"> 
          <append function="0"/>
          <port name="0"/>
        </target>
      </connection>
    
      <connection name="_c">
        <origin name="C2">
          <append function="N"/>
          <port name="1"/>
        </origin>
        <target name="consumer">
          <port name="100"/>
        </target>
      </connection>
 在connect部分，to_square是个迭代的部分，连接C2\_i 到square\_i，而from_square也是个迭代的部分，连接square\_i到C2\_i+1。因此这两者就迭代了所有中间的连线。

​	在加上开头的generator与C2\_0的连线 和 C2\_N与consumer的连线，这样就构成了完整的dol图。



​	这次实验的内容有点过于简单，导致一些很核心的知识点会忽视掉，这是这次实验设计的不太好的地方。