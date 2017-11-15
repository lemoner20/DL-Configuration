# Ubuntu DL Configuration

This applies only to my machine.

Caffe  Tensorflow Configuration
==========
环境

	OS：Ubuntu 16.04(64 bit) 
	GCC/G++: gcc5.4.x / g++5.4.x
	GPU：Nvidia GTX 1080Ti × 1
	CPU:  Intel® Core™ i7-7700k CPU @ 4.20GHz × 8 
	内存：32 GB
	CUDA：CUDA 8.0
	cuDNN：cuDNN v8.0 for CUDA 8.0
	OpenCV: opencv-3.2.0 
	Python: Anaconda2=python2.7 + python3.5&3.6 

目录

	1. 安装Ubuntu 16.04.3 LTS
	2. 安装NVIDIA显卡驱动
	3. 安装CUDA
	4. 安装cuDNN
	5. 安装OpenCV
	6. 安装Anaconda2
	7. 安装Python3.6
	8. 安装Pycharm & Spyder
	9. 安装chrome，teamviewer，sogoinput等软件
	10. 安装Tensorflow
	11. 安装Caffe和pycaffe


1.安装Ubuntu 16.04 LTS

	制作U盘安装。
	安装14.04 再升级为16.04；
	或直接安装16.04。
	注意安装中的黑屏问题，修改grub.cfg文件，将其中的quite splash修改为nomodeset，重启安装即可。
	安装中的分区问题：
		1。格式化原有分区
		2。主分区：挂载点 “/”，存放系统文件，位置是空间起始位置，用于EXT4日志文件系统，设置为20G；
		3。交换空间：Swap，逻辑分区，虚拟内存，设置为30G；
		4。驱动分区：挂载点  “/boot”，逻辑分区，引导系统启动，设置为512M；
		5。用户空间：挂载点 “/home”，逻辑分区，存放个人文件，设置为40G；
		6。自动安装系统即可。

2.安装NVIDIA显卡驱动

	禁用Ubuntu自身的nouveau驱动，改用NVIDIA独立显卡的驱动：
		1。lsmod | grep nouveau
		2。sudo gedit /etc/modprobe.d/blacklist.conf  打开黑名单文件
		3。添加： blacklist nouveau
		4。再次检查： lsmod | grep nouveau
	安装NVIDIA驱动：
		1。输入 nvidia-smi 查看驱动信息；
		2。关闭Xorg图形窗口： sudo service lightdm stop
		3。进入控制台： Ctrl + Alt + F1 
		4。删除原有驱动： sudo apt-get remove --purge nvidia*
		5。赋予权限： sudo chmod +x NVIDIA-Linux-x86_64-384.59.run
		6。安装驱动： sudo ./NVIDIA-Linux-x86_64-384.59.run
		7。打开图形界面： sudo service lightdm start
		8。查看驱动信息： nvidia-smi
	注意：如果出现循环登录界面的问题，是因为驱动冲突，前面nouveau没有彻底禁用，只需要重新禁用再次安装NVIDIA驱动即可。或者直接 sudo naulitius，root权限打开文件目录，直接删除nouveau下的的全部文件，路径是：/lib/modules/xx-generic/kernel/drivers/gpu/drm/nouveau 。
	安装完后可以在系统设置的软件与更新中的附加驱动里面手动设置显卡驱动。

3.安装CUDA

	到cuda官网下载对应版本cuda的runfile文件，直接安装，设置cuda的环境路径等。
		1。sudo chmod +x cuda_8.0.61_375.26_linux.run
		2。sudo service lightdm stop
		3。sudo ./cuda_8.0.61_375.26_linux.run  #不要安装OPENGL
		4。sudo service lightdm start
		5。打开环境变量文件：sudo gedit ~/.bashrc
		6。在最后加入：
			export PATH=/usr/local/cuda-8.0/bin:$PATH
			export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64:$LD_LIBRARY_PATH
		7。生效：source ~/.bashrc
		8。查看环境：echo $PATH； 
		9。查看cuda版本： nvcc -V 
		10。cd  ~/NVIDIA_CUDA-8.0_Samples ，make编译，完成后执行： deviceQuery ，查看更多的信息
	另外，建立动态链接： sudo gedit /etc/profile，写入： export PATH = /usr/local/cuda/bin:$PATH
		sudo gedit /etc/ld.so.conf.d/cuda.conf  ，写入：/usr/local/cuda/lib64 ，生效： sudo ldconfig

4.安装cuDNN

	将cudnn中的相关文件复制到Ubuntu系统的相关目录中，建立软连接即可。
		1。sudo cp cudnn.h  /usr/local/cuda/include/    #复制头文件
		2。sudo cp lib*  /usr/local/cuda/lib64/    #复制动态链接库
		3。cd  /usr/local/cuda/lib64/
		4。sudo rm -rf libcudnn.so  libcudnn.so.6    #删除原有动态文件
		5。sudo ln -s libcudnn.so.6.0.21  libcudnn.so.6   #生成软衔接
		6。sudo ln -s libcudnn.so.6  libcudnn.so      #生成软链接
		7。sudo  ldconfig

5.安装OpenCV

	下载opencv3.2，直接编译即可，需要合适的gcc版本。注意，这还不能用python-opencv。
		1。cd ~/opencv
		2。mkdir build  //建立build文件夹
		3。cd build
		4。cmake -D CMAKE_BUILD_TYPE=Release   -D BUILD_TIFF=ON   -D CMAKE_INSTALL_PREFIX=/usr/local ..
		5。sudo make all  //编译
		6。sudo make install  //安装
	注意，建立文件：sudo gedit /etc/ld.so.conf.d/opencv.conf，加入/usr/local/lib ，sudo ldconfig ；
	编辑文件：sudo gedit /etc/bash.bashrc， 加入： PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/local/lib/pkgconfig
		export PKG_CONFIG_PATH
	安装完成后进行测试：进入：cd  /samples , cmake . , make ,之后，cd cpp/  ，执行： ./cpp-example-facedetect girls.jpg  
	另外，安装python-opencv的方法：利用Anaconda安装opencv-python，命令：conda install --channel https://conda.anaconda.org/menpo opencv3
	或：直接安装opencv，命令：sudo pip install  opencv-python

6.安装Anaconda2

	Anaconda是一个python的集成软件包，安装Anaconda2并把其设置成默认的python路径即可。
		1。bash  Anaconda2-4.3.1-Linux-x86_64.sh
		2。在最后输入yes，或者 sudo gedit ~/.bashrc ，手动写入： export PATH=/home/limy/anaconda2/bin:$PATH
		3。source ~/.bashrc
		4。命令行输入python，可以查看当前使用的python版本。
	另外，利用conda安装软件的办法：
		1。寻找安装的软件名：anaconda search -t conda tensorflow
		2。anaconda show jjhelmus/tensorflow  #选择一个合适的源，如jjhelmus
		3。执行安装： conda install --channel https://conda.anaconda.org/jjhelmus tensorflow

7.安装Python3.6

	当前最新版本的python为3.6，一些最新的研究可能会使用3.6，安装上用来跑别人的公开代码。
	这部分安装问题很大，目前还不够稳定，没有特殊要求就不要安装。

8.安装Pycharm和Spyder

	Pycharm和Spyder是两个ide，安装使用十分方便。
	安装Pycharm：
		1。解压Pycharm安装包
		2。进入文件的bin中，运行 ./pycharm.sh
		3。import导入个人配置
		4。导入PythonPath
	安装Spyder：
		1。pip install spyder
		2。sudo apt-get install python-pyqt*
		3。配置python路径和ide风格即可

9.安装相关软件

	常用有SogoPinYin、Chrome、Teamviewer等软件。
	下载deb包，直接dpkg解压安装即可。

10.安装Tensorflow

	pip install tensorflow_gpu-1.3.0rc0-cp27-none-linux_x86_64.whl
	adaconda会自动补全依赖的安装包。
	记得在ide中更新一遍pythonpath。

11.安装Caffe和pycaffe

	安装关于caffe的一些依赖，在编译安装caffe，最后编译pycaffe，可以再ide中调用。
		1。cd caffe-master/  
		2。sudo cp Makefile.config.example Makefile.config  #根据自己的需要设置
		3。make all   //编译
		4。make test  //测试
		5。make runtest   //搞定了！！！
	注意：makefile.config中的anaconda2路径要先写错，不然会发成冲突，等编译安装完成后再改为正确的路径，然后编译pycaffe。
	另外，caffe计算会用到cuda，创建一个路径：sudo gedit /etc/ld.so.conf.d/caffe.conf ，写入/usr/local/cuda/lib64， 然后生效 sudo ldconfig
	安装pycaffe：
		1。make pycaffe
		2。sudo gedit /etc/profile
			# 添加： export PYTHONPATH=/home/user/caffe/python:$PYTHONPATH
		3。source /etc/profile 
		4。写入ide的pythonpath中
