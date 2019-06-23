<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>uClinux driver module</title>
  <link rel="stylesheet" href="https://stackedit.io/style.css" />
</head>

<body class="stackedit">
  <div class="stackedit__left">
    <div class="stackedit__toc">
      
<ul>
<li><a href="#build-uclinux-driver-module">Build uClinux driver module</a></li>
<li><a href="#操作步骤">操作步骤</a>
<ul>
<li><a href="#修改根目录下makefile">修改根目录下Makefile</a></li>
<li><a href="#修改linuxdirmakefile">修改$(LINUXDIR)/Makefile</a></li>
<li><a href="#修改linuxdirdriversmakefile">修改$(LINUXDIR)/drivers/Makefile</a></li>
<li><a href="#添加自己的驱动代码">添加自己的驱动代码</a></li>
<li><a href="#todo">TODO</a></li>
</ul>
</li>
</ul>

    </div>
  </div>
  <div class="stackedit__right">
    <div class="stackedit__html">
      <h1 id="build-uclinux-driver-module">Build uClinux driver module</h1>
<p>这篇文档主要记录如何在uClinux-dist-20060311中编译驱动程序模块。过程中为了提升编译速度，只编译对应模块，对源码树中的Makefile文件做了定制化修改。编译驱动模块需要先配置内核支持模块加载功能。</p>
<pre><code>Loadable module support  ---&gt;
    [*] Enable loadable module support 
    [ ]   Set version information on all module symbols
    [*]   Kernel module loader
</code></pre>
<h1 id="操作步骤">操作步骤</h1>
<p>获取uClinux源码并解压，这里uClinux-dist-20060311为例，解压得到的源码根路径为/home/hector/s3c44b0/uClinux/uClinux-dist-20060311-work/uClinux-dist，文档后续给出的路径均以此为根路径。并以linux-2.4.x/drivers/misc模块为例演示如何修改Makefile以达到只编译misc模块。</p>
<h2 id="修改根目录下makefile">修改根目录下Makefile</h2>
<p>在根目录Makefile文件末尾添加如下代码段</p>
<pre><code>.PHONY: mod_misc
mod_misc:
	@echo "build misc driver module"
	$(MAKEARCH) -C $(LINUXDIR) mod_misc
</code></pre>
<h2 id="修改linuxdirmakefile">修改$(LINUXDIR)/Makefile</h2>
<p>这里$(LINUXDIR)即为linux-2.4.x，在该Makefile文件末尾添加如下代码段</p>
<pre><code>.PHONY: mod_misc
mod_misc: include/linux/version.h include/config/MARKER
	@echo "build misc driver module"
	$(MAKE) -C drivers CFLAGS="$(CFLAGS) $(MODFLAGS)" MAKING_MODULES=1 mod_misc
</code></pre>
<h2 id="修改linuxdirdriversmakefile">修改$(LINUXDIR)/drivers/Makefile</h2>
<p>在该Makefile文件末尾添加如下代码段</p>
<p>.PHONY: mod_misc<br>
mod_misc: _modsubdir_misc</p>
<p>修改完这3个Makefile文件后，在根目录下执行如下命令即完进行misc驱动模块的打包了</p>
<pre><code>make mod_misc
</code></pre>
<h2 id="添加自己的驱动代码">添加自己的驱动代码</h2>
<p>因为misc模块是uClinux自带的一个空模块，实际没有驱动代码。下面添加自己的代码文件srf05.c文件，具体内容如下</p>
<pre><code>#include &lt;linux/init.h&gt;
#include &lt;linux/module.h&gt;
#include &lt;linux/kernel.h&gt;

MODULE_LICENSE("Dual BSD/GPL");

static int srf05_init(void)
{
    printk(KERN_ALERT "Hello, world\n");
    return 0;
}

static void srf05_exit(void)
{
    printk(KERN_ALERT "Goodbye, cruel world\n");
}

module_init(srf05_init);
module_exit(srf05_exit);
</code></pre>
<p>然后修改misc/Makefile内容如下</p>
<pre><code># parent makes..
#

obj-m += srf05.o

include $(TOPDIR)/Rules.make

fastdep:
</code></pre>
<p>然后再次回到根目录执行</p>
<pre><code>make mod_misc
</code></pre>
<p>即可生成srf05.o文件。<br>
下一步打开开发板，通过nfs挂载misc目录到开发板系统中</p>
<pre><code>mount -t nfs 192.168.3.4:/home/hector/s3c44b0/uClinux/uClinux-dist-20060311-work/uClinux-dist/linux-2.4.x/drivers/misc /lib/modules/2.4.24-uc0/kernel/drivers/misc -o nolock
</code></pre>
<p>在开发板系统中执行如下命令即可加载srf05的驱动了</p>
<pre><code>insmod srf05
</code></pre>
<h2 id="todo">TODO</h2>
<p>上面的修改只是添加了一条misc模块的编译目标，进一步使用makefile的通配符语法可以实现任意的驱动模块独立编译。</p>
<blockquote>
<p>Written with <a href="https://stackedit.io/">StackEdit</a>.</p>
</blockquote>

    </div>
  </div>
</body>

</html>
