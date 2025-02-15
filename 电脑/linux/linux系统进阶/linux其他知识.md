<nav id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#orgheadline1">1. 让你的ubuntu有个分身</a></li>
<li><a href="#orgheadline2">2. 在ubuntu下删除ntfs分区的休眠文件</a></li>
<li><a href="#orgheadline3">3. 在ubuntu下通过ISO文件硬盘安装win7系统</a></li>
</ul>
</div>
</nav>


# 让你的ubuntu有个分身<a id="orgheadline1"></a>

移动home和让ubuntu拥有分身的技术。 [主要参考了这个网站](http://wangmm2008.blog.163.com/blog/static/1812740122011111112842470/) 。

1.  新开一个分区，格式化为ext4格式。
2.  将你的home目录复制过去。建议用root账户操作：

    su
    sudo cp -afrv /home/* /media/wanze/data

这里建议使用 **gparted** 分区工具将分区的卷标加号，比如上面我加上了data卷标，然后挂载就成了/media/wanze/data的地址，当然你也可以用ubuntu的文件浏览器设置那里选择输入位置，那个位置你复制了就是的。
上面选项加上v就是为了防止你出现系统卡死的错觉。。

1.  复制的时候你可以开始修改/etc/fstab文件。我之前没有/home设置，所以需要重新加上这样一句：

    # home was on /dev/sda6 during installation
    UUID=e56f656e-8ebb-4ab8-8f76-c1e26aba22a4  /home ext4 defaults 0 0

上面的UUID也就是你新分区的那个UUID，通过命令：

    ls -l /dev/disk/by-uuid

查看。日期时间后面那个就是，然后设置/home挂载点，其他就是defaults 0 0 0了。

1.  稳妥起见，我发现复制完了新的wanze文件夹权限不一样了，用sudo nautilus 修改下权限，和你之前的一模一样就行了。
2.  重启，发现简直一模一样，包括网络硬盘同步程序等等都没出错。
3.  将fstab那一句注释掉，重启，发现又进入原来的wanze主目录了。然后将不重要的音乐，下载的文件图片等，因为重复了，所以删除掉节省点根目录的空间。
4.  将fstab那句不注释，发现又进入新的主目录文件夹了。这样就感觉有了两个系统，毕竟个人电脑用户出错就出错在home文件夹里面的设置上，这样算是有了双保险了把。

# 在ubuntu下删除ntfs分区的休眠文件<a id="orgheadline2"></a>

这个问题是windows休眠了，然后不知怎么出错了然后就进入不了windows系统了。这个时候windows系统还是有救的。如果你装了双系统的话，进入ubuntu系统，删除掉ntfs分区的休眠文件即可。参考了 [这个网页](http://askubuntu.com/questions/204166/how-do-i-mount-a-hibernated-ntfs-partition) 。

    sudo mount -t ntfs-3g -o remove_hiberfile /dev/sda5  具体挂载的位置

这个命令其实就是mount命令，然后加上了 `-o remove_hiberfile` 。后面的第一个参数是待加载项，这里应该是你的win7安装的目标ntfs分区，可用 `sudo fdisk -l` 查看一下。然后具体挂载的位置随意:

    sudo mkdir /media/wanze/D
    sudo mount  /dev/sda5 /media/wanze/D

# 在ubuntu下通过ISO文件硬盘安装win7系统<a id="orgheadline3"></a>

1.  用gparter分区
2.  先mount
3.  把文件复制到d盘
4.  执行 sudo update-grub
5.  重启到新加入的那个恢复模式下即可﻿