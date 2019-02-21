Title: kvm虚拟机系统的高分辨率调整
Date: 2019-02-21 20:26  
Modified: 2019-02-21 20:26  
Category: TECH SKILLS  
Tags: kvm, 分辨率, linux, mint  
Slug: kvm-virtual-monitor-resolution
Authors: Yullin

最近在服务器中用KVM虚拟机折腾Linux Mint, 因为对KVM并不是很熟悉，所以折腾了挺久。这里记录一下，遇到的其中一个问题。

系统的安装其实挺简单，但是在安装好之后发现虚拟机的最大分辨率也只有`1024X768`，无法再调整到更高。  

经过一番查询之后，才知道分辨率与创建虚拟机时所设置的显存有关。  

默认情况下，显存设置为`16MB`，所以分辨率最高只能到`1024X768`，具体代码如下：
```
    <video>
      <model type='cirrus' vram='16384' heads='1' primary='yes'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
    </video>
```
我们需要将vram调整得大一些，但是具体应该调整为多少呢？  

以4K分辨率(3840 x 2160)为例：
```
3840 x 2160 = 8294400     # 总共像素点
8294400 x 32 = 265420800  # 每个像素点占 4 个字节
265420800/8 = 33177600    # 转为 Byte 单位
33177600 /（1024 * 1024）= 31.640625 MB
```
可以看到基本上`32MB`足以支持4K显示，再考虑到其他的一些开销，我们将显存设置为`64MB`。
视频类型设置为 qxl ，修改后配置如下：
```
    <video>
      <model type='qxl' ram='65536' vram='65536' vgamem='65536' heads='1' primary='yes'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
    </video>
```

重启虚拟机之后，再看就可以调整为自己想要的分辨率了。