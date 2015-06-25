##docker最基本的知识点
1. 用的最多的就是run命令（备注：在macos下需要使用 docker + 命令，在ubuntu下就需要使用sudo docker + 命令），run命令是一个将镜像转变为容器的命令换言之就是启动一个容器（换言之容器就是动态的镜像，当启动容器的时候，在镜像层覆盖一层读写层）。
例如：    
`sudo docker run -i -t --name firstContainer ubuntu:14.04 /bin/sh`
> 这个代码的意思就是以ubuntu镜像（先会在本地仓库寻找镜像，若此时没有则会去dockerHub上去抓取。同时我们指定了版本号，如果我们没有指定版本号，那么docker会自动的pull下最新的版本。）；-i -t 会创建一个新的窗口并且开启一个新的终端。--name 我们指定了该容器的名字。不然docker 会随机分配一个名字，另外每一个容器都有一个容器ID。当我们执行完该命令后docker会返回一个container ID 。

2. 在第一个命令中，我们使用了官方的仓库 Ubuntu镜像来启动容器，在docker中镜像分为俩个类型：1.官方的 2.非官方的 。我们可以从命名来区分官方和非官方。官方的就像ubuntu：14.04 nginx mysql 这样我们经常看见的。非官方的一般才用 用户名/镜像名：标签这样的命名方式来进行命名，一般这个用户名都是我们在dockerHub（类似于github）上注册账户名，当然我们也可以自己取 。比如我自己制作了一个mysql镜像，我可以使用以下命令
`sudo docker build -t mysql .`
其中点代表当前目录，说明我的制作镜像的文件在当前目录下，我此时取名为mysql，当然在本机时不会报错，但当我们
`sudo docker push mysql`
会出错！为了避免错误，在一开始命名的时候就需要使用非官方标准格式来命名。同样可以可以使用 `sudo docker search imageId`命令来搜索镜像。

3. 当我们制作镜像的时候可以有俩种方式，一种是以基础镜像（诸如ubuntu，centos操作系统的基镜像）开启一个容器，然后一步一个命令（一般来说一个命令就会在文件系统中覆盖一个新的文件层）执行，最后使用 
`sudo docker commit -t image_name`命令来产生一个镜像。很明显这样的方式产生一个镜像很繁琐，不方便，我们不能每到一个环境就重复做这么多此命令来产生一个镜像。那么就可以使用另一种方式Dockerfile，将创建镜像的命令全部装到这个文件中，然后使用`sudo docker build -t image_name .`这个命令就可以依照dockerfile中的指令来完成镜像的创建。dockerfile就是docker中的集装箱，无论你有什么，全部塞进dockerfile集装箱中，不论部署环境怎么变化，直接使用上面的build命令，完成所有生产环境的搭建，因为是操作系统级别的，所以启动速度会很快，秒级的。换句话说，因为有了dockerfile才使docker可以这般敏捷。

4. `FROM Ubuntu:14.04`           
   `MAINTAINER 操作人 操作人email `     
   `RUN apt-get update && \
	apt-get install python-dev`    
    `RUN sudo echo 'hello word'`    
	`ADD ./requirements.tx /home`    
	`COPY ./requirements.txt /home`     
	`ENTRYPOINT /bin/sh echo 'hello word'`    
	`CMD /bin/sh echo 'hello world'`    
	`USER xu`     
	`WORKDIR /home #指定工作目录`     
	`VOLUME /home/xu/fitness`    
	`ENV PATH /home/you/path`       
	以上是几乎所有的饿dockerfile中的命令，我们使用这些命令就可以自动化的搭建一个完整的部署开发环境。
	1. FROM 是每个Dockerfile文件必须的，也是Dockerfile开头的第一行一定是这个，可以是dockerHub官方仓库的，也可以是我们自己的镜像。
	2. MAINTAINER 指定的是操作人，很简单。
	3. RUN 命令是我们需要执行的命令，在另外一种创建镜像的方式中，我们就是一条一条命令来操作创建镜像的，同样这里的RUN 命令就是我们一个个命令，只是在Dockerfile中使用RUN指定了而已，在Dockerfile中每一个指定是一行，每一行都会生成一个镜像层，所有的镜像层是叠加的，这样在我们这个Dockerfile中最底层的就是Ubuntu镜像。由于镜像层是叠加的，因此就会可能我们的镜像层会有很多层，那么我们创建出来的镜像image就会很大，当我们很大的镜像启动一个容器的时候，再在上面覆盖一个读写层，由于我们在容器中的操作都是遵循写时复制的原则，所谓写时复制就是在我们操作容器的时候我们会将镜像中的相关文件复制到读写层进行操作，由于镜像层数越多，那么容器去搜索相关的文件时间就会越长，自然操作就会放缓，所以我们在书写Dockerfile的时候应该要精简。精简会在第五点说到。
	4. ADD 原文件（目录） 目标目录，文件 ，ADD是一个强大的指定，他可以将指定的文件或目录复制进入容器中，若源文件是压缩包，ADD命令会自动解压赋值进入容器，另一个命令COPY就不可以。ADD区分是文件还是目录的方式就是看源文件，若源文件是.*开头的话，那么就默认为文件，反之则是目录。另外提一点是ADD会是缓存失效，缓存会在后面使用
	5. COPY和ADD的功能差不多，就是比ADD低级点，不提供解压功能。就是很呆板的直接复制。
	6. ENTRYPOINT命令是一个入口命令，这个命令会在完成镜像创建后第一个执行的命令，也就是container启动成功后最先运行的命令，CMD命令和ERTRYPOINT命令功能差不多，但是不同的是ENTRYPOINT指定的命令在容器启动会一定会执行，即使我们在`sudo docker run -it ubuntu /bin/sh`即使我们启动容器后指定了命令/bin/sh，都会执行ENTRYPOINT指定的命令，但是此时不会执行CMD 指定的命令，此时CMD命令会被覆盖，我们通常会使用以下语法来结合ENTRYPOINT CMD 命令来完成一些有意思的工作例如：    
	`ENTRYPOINT['/bin/sh/do/app']`    
	`CMD ['--help']`       
	此时当我们没有在启动容器的时候指定初始命令的时候，这样容器会将ENTRYPOINT，CMD结合起来，然后执行命令，由CMD提供参数，ENTRYPOINT代表命令
	7. USER 是指定容器启动后用户是谁？ VOLUME指定的是挂载的目录，挂载是docker中一个很好的功能，挂载的目录，文件会绕过docker的文件系统，直接挂载docker容器上，我们对挂载的文件的每一次修改都会生效，这个功能很适合开发测试。VOLUME格式：VOLUME 主机目录或者文件:[挂载到的容器的目录文件]其中若我们没有指定挂载到docker安装目录中的volume目录下，若我们在启动容器的时候，若我们挂载了容器，当我们在删除容器的时候，我们在删除的时候需要指定-v 选项来删除
	`sudo docker rm -v containerId`
	8. 最后一个env命令可以指定在contain内部一些环境变量，在我看来这个变量和dockerfile的缓存有很大关系。在dockerfile中每一步都会将结果提交为镜像，所以docker的构建镜像过程就显得很聪明，他会将之前的镜像层看成缓存，例如我们在构建一个新的dockerfile中前四步在缓存中有，那么我们在构建新的镜像的时候，docker就会直接从第四步开始，这样当前面几步没有多大的变化的时候，这样docker会节省很多的时间，但是当前面几步产生变化，但是docker的缓存依然有效的话，那么就会出现各种问题，我们当然可以在build的过程中指定 --no-cache如下`sudo build --no-cache .`这样我们可以避免一些缓存造成的各种奇奇怪怪的问题，但是这样每一步的建立都不使用混存的话，那么我们会浪费很多的时间，会造成浪费。因此为了避免缓存造成的问题，我们应该使用合理利用缓存，这样就要合理组织dockerfile的结构。我们可以在dockerfile中设置一个名叫REFRESHED_AT环境变量，用来告诉docker我这个模板构建的时间，紧接着可以刷新我们的APT包，用来确保我们的包都是最新版本，具体如下：    
	`FROM ubuntu:14.04`  
	`MAINTAINER name email`    
	`ENV REFRESHED_AT 2015-03-30`     
	`RUN apt-get -qq update`     
	基于此模板只要我们在使用这个模板构建新的dockerfile的时候只要修改环境变量REFRESHED_AT的值，就开始重置这个缓存，并运行后续的命令而无需依赖缓存。这样我们就可以合理利用缓存。
5. 上一点讲了所有的Dockerfile的命令，可以使用上述命令来构建所有类型的docker容器。但是docker作为轻量化，简便的部署方案，我们需要精简我们的dockerfile 尤其是dockerfile的层数。按照docker容器化的思想，所有的应用程序都容器化，每个容器只做一个服务，做一件事情，多个容器组合起来完成一个大的任务。既然是这样的，我们的dockerfile就应该是精简的，控制好镜像的层数。
    1. WORKDIR , CMD , ENV 这样的命令尽量在dockerfile的底部,诸如RUN apt-get install这样的命令应该放在最前面，这个命令是很浪费时间。
    2. 尽量选择最合适的基础镜像，如果我们是一个python应用那么我们的基础镜像就可以指定为python：2.7，若我们的容器就是一个nginx，我们的基础镜像就可以是nginx ，如果我们的一个容器就是一个挂载代码的容器，我们可以使用最精简的镜像 busybox，总之一切从简
    3. 在dockerfile中每一个命令都会生成一个镜像层，所以我们可以通过组合命令来缩减镜像层，通过将RUN命令分组，可以在容器之间共享更多的层，如果我们有一组命令是被多个镜像公用的，那么我们很明显可以将这组命令组合起来创建一个新的镜像。
    4. 当我们发现在构建安装的过程中一个包丢失了，我们可以直接在dockerfile的底部增加，一定不要在底部添加。
    5. 其实所谓的分组就是将安装nginx相关的放在一起，将安装mysql放在一起，这样他们可以更好的共享利用缓存。
    6. 在构建的最后，我们需要清空apt-get更新后那些用不掉的包。
6. 在上面我们知道如何构建镜像，基于镜像启动一个容器，如何精简镜像，精简镜像的原则，那么镜像到底是什么？要理解镜像需要知道docker的文件系统，docker使用联合文件系统（AUFS），镜像是只读的文件层，那么容器其实就是在镜像上加了一个读写层，那么这个就应该是读写的，通过联合文件系统将只读的镜像和可读写的读写层结合在一起，在我们看来就是一个文件系统，形成一个整体的文件系统。镜像的最底层是Rootfs，和Linux的rootfs一样的。传统来说，Linux操作系统内核启动时，内核首先会挂载一个只读（read-only）的rootfs，当系统检测其完整性之后，决定是否将其切换为读写（read-write）模式，或者最后在rootfs之上另行挂载一种文件系统并忽略rootfs。Docker架构下，依然沿用Linux中rootfs的思想。当Docker Daemon为Docker Container挂载rootfs的时候，与传统Linux内核类似，将其设定为只读（read-only）模式。在rootfs挂载完毕之后，和Linux内核不一样的是，Docker Daemon没有将Docker Container的文件系统设为读写（read-write）模式，而是利用Union mount的技术，在这个只读的rootfs之上再挂载一个读写（read-write）的文件系统，挂载时该读写（read-write）文件系统内空无一物。     
  在AUFS的强大功力下，我们是不知道那些文件是可读写的，那些是只读的，因此会出现一些情况是当我们删除了会怎么办呢？其实这不用关心，因为我们所有的操作都是在读写层的，而读写层是在只读的镜像层最上面的，即使我们操作的只读层的内容，我们操作的也是只读层的拷贝。因此我们删除的是拷贝，只读层对我们用户来说应该是不可见的。我们无法撼动它。在镜像中，我们的镜像是可叠加的，因此我们可以组织镜像进行叠加，产生一个继承树那要的image镜像。     
7. 前面6点说的都是使用docker直接创建容器，满足最基本的创建容器的需求。足够我们可以将应用容器化。第七点说的就是docker的架构，docker怎么运行的。docker是一个C/S结构的应用我们平时使用的都是client端，使用命令行来与S端交互，其中S端就是docker daemon一个常驻主机内核的进程，他一直在后台监听client端的请求，其中这个client可以是本机的，也可以是其他主机的。从client传来的请求会经过docker daemon中的docker server，然后经过route（具备路由功能）的解析，分配给具体的handler去处理，这些handler会被分解为一个个的job交给docker engine来处理；（其中job如果与镜像相关的情况下，首先会在交由本地的graphdirver处理查看本地是否有相关的镜像，若没有则docker engine会直接在docker registry查找，直接pull下来，然后再交由驱动层的graphdriver处理存储在本地，其中会转换为特有的docker image格式，并存在在GraphDB中，该数据库是一个sqlite3数据库）然后这些job又传递到下一层的驱动层来处理，在docker驱动中有三个驱动来处理所有的job ，graphdriver处理与docker image相关的job，和处理网络相关的networkdriver驱动模块，该驱动模块会在docker容器初始化的时候完成所有的网络相关的配置，另外一个模块就是处理一些命令相关的驱动模块execdirver。经过驱动层解析后所有的请求会调用libcontainer这一层中相关的指令类库，libcontainer是一个用go语言实现的库，设计初衷是希望该库可以不依靠任何依赖，直接访问内核中与容器相关的API。正是由于libcontainer的存在，Docker可以直接调用libcontainer，而最终操纵容器的namespace、cgroups、apparmor、网络设备以及防火墙规则等。这一系列操作的完成都不需要依赖LXC或者其他包。docker container是docker架构中服务提交的最终部分。
8. docker使用命令行与后台进行交互，在命令行中主要有以下四类命令：
   1. 环境信息相关：info ：这个命令在开发者报告Bug时会非常有用，结合docker vesion一起，可以随时使用这个命令把本地的配置信息提供出来，方便Docker的开发者快速定位问题。 version ：显示Docker的版本号，API版本号，Git commit， Docker客户端和后台进程的Go版本号。
   2. 运维相关：attach ：个人感觉我对这个命令是不太理解，每次使用都感觉没什么用。那可能是我在容器中跑的命令是后台运行的没有输出的原因。一般我们使用这个命令可以挂载正在后台运行的容器，在开发应用的过程中运用这个命令可以随时观察容器內进程的运行状况。开发者在开发应用的场景中，这个命令是一个非常有用的命令。    
   build：命令在之前我们创建image的时候就使用了，是用来创建一个image的命令与之相关的动态的提交将容器转化为image的一个命令，但我们希望部署一体化的话一般情况下不推荐使用，这个commit是将我们当前的容器转化为镜像。     
   diff： sudo docker diff containerId 用来查看容器中文件的变化，A - Add, D - Delete, C - Change 

