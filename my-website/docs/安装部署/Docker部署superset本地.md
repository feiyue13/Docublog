---
sidebar_label: '使用docker compose本地创建superset'
sidebar_position: 1
---
由于本平台采用的superset，所以我们可以在本地尝试部署superset
1.	使用 Docker 在本地安装superset
在本地尝试Superset的最快方法是在Linux或Mac OSX上使用Docker和Docker Compose。 计算机。Superset 没有对 Windows 的官方支持，因此我们提供了一个 VM 解决方法 下面。
1. 安装 Docker 引擎和 Docker Compose

Mac OSX
安装 Docker for Mac，其中包括 Docker 引擎和开箱即用的最新版本。docker-compose
安装 Docker for Mac 后，打开 Docker 的首选项面板，转到 “资源”部分，并将分配的内存增加到 6GB。仅分配了 2GB 的 RAM 默认情况下，superset将无法启动。
Linux目录
按照 Docker 在 Linux 上安装 Docker。 说明适合您的 Linux 版本。因为未安装 Linux 上的基本 Docker 安装的一部分，一旦您有一个有效的引擎，请按照 Linux 的 docker-compose 安装说明进行操作。docker-compose窗户

不幸的是，Superset在Windows上不受官方支持。Windows用户的一个选项 在本地尝试Superset是通过VirtualBox安装Ubuntu桌面VM，并在Linux上继续Docker指令 的虚拟机。我们建议为虚拟机分配至少 8GB 的 RAM 以及 配置至少 40GB 的硬盘驱动器，以便为操作系统和 所有必需的依赖项。Docker Desktop 最近添加了对 Windows Subsystem for Linux （WSL） 2 的支持，这可能是另一种选择。

2. 克隆超集的 GitHub 存储库
在终端中使用 以下命令：
git clone https://github.com/apache/superset.git

该命令成功完成后，您应该会在 当前目录。superset
3. 通过 Docker 撰写启动超集
导航到您在步骤 1 中创建的文件夹：
cd superset


在主分支上工作时，运行以下命令：
docker-compose -f docker-compose-non-dev.yml pull
docker-compose -f docker-compose-non-dev.yml up

或者，您也可以通过先签出来运行特定版本的superset 分支/标记，然后从变量开始。 例如，要运行 1.4.0 版本，请运行以下命令：docker-composeTAG
git checkout 1.4.0
TAG=1.4.0 docker-compose -f docker-compose-non-dev.yml pull
TAG=1.4.0 docker-compose -f docker-compose-non-dev.yml up

您应该会看到计算机上启动的容器的日志记录输出墙。一次 此输出速度变慢，您应该在本地计算机上有一个正在运行的 Superset 实例！
注意：这将在非开发模式下启动超集，对代码库的更改将不会反映。 如果要在开发模式下运行超集以测试本地更改，只需将前面的命令替换为：， 并等待容器完成资产构建。docker-compose upsuperset_node

配置 Docker 撰写
以下内容适用于想要在 Docker Compose 中配置超集启动方式的用户;否则，您可以跳到下一部分。
您可以分别使用 和 配置 Docker 撰写模式的开发模式和非开发模式的 Docker 撰写设置。这些环境文件为 Docker Compose 设置中的大多数容器设置环境，某些变量影响多个容器，而其他变量仅影响单个容器。docker/.envdocker/.env-non-dev
一个重要的变量是确定容器是否将示例数据和可视化加载到数据库和超集中。这些示例对大多数人很有帮助，但对于有经验的用户来说可能是不必要的。加载过程有时可能需要几分钟和大量的 CPU，因此您可能需要在资源受限的设备上禁用它。SUPERSET_LOAD_EXAMPLESsuperset_init
注意：用户通常希望从 Superset 连接到其他数据库。目前，执行此操作的最简单方法是修改文件并将数据库添加为其他服务所依赖的服务（通过 ）。其他人尝试设置超集服务，但这些服务通常会中断安装，因为配置需要使用 Docker 撰写 DNS 解析程序作为服务名称。如果您有很好的解决方案，请告诉我们！docker-compose-non-dev.ymlx-superset-depends-onnetwork_mode: host

4. 登录超集
您的本地超集实例还包括一个用于存储数据的 Postgres 服务器，并且已经 预加载了超集附带的一些示例数据集。您现在可以通过您的 通过访问 Web 浏览器。请注意，许多浏览器现在默认为 - if 您的是其中之一，请确保它使用 .http://localhost:8088httpshttp
使用默认用户名和密码登录：
username: admin

password: admin

5. 将超集连接到本地数据库实例
当使用或运行超集时，它在自己的 docker 容器中运行，就好像超集完全在单独的机器中运行一样。因此，尝试使用主机名连接到本地数据库将不起作用，因为指的是 docker 容器 Superset 正在运行，而不是您的实际主机。幸运的是，docker 提供了一种从容器内部访问主机中的网络资源的简单方法，我们将利用此功能连接到本地数据库实例。dockerdocker-composelocalhostlocalhost
这里的说明是从Superset（在其docker容器中运行）连接到postgresql（在你的主机上运行）。其他数据库的配置可能略有不同，但要点是相同的，归结为 2 个步骤 -
1.	（Mac 用户可以跳过此步骤）配置本地 postgresql/数据库实例以接受公共传入连接。默认情况下，postgresql 只允许来自主机和 docker 容器的传入连接，但再次重复是不同的。对于 postgresql，这涉及对文件进行一行更改，并且您可以在 Web 上轻松找到针对您的 OS / PG 版本量身定制的有用链接。对于 docker 来说，只需将 IP 列入白名单就足够了，但无论如何，当您向公共互联网开放数据库时，在生产数据库中执行此操作可能会产生灾难性的后果。localhostlocalhostspostgresql.confpg_hba.conf172.0.0.0/8*
2.	尝试连接到数据库时，请尝试使用 （Mac 用户、Ubuntu） 或 （Linux 用户） 作为主机名，而不是 。这是 docker 内部细节，正在发生的事情是，在 Mac 系统中，docker 为主机名创建一个 dns 条目，该条目解析为主机的正确地址，而在 linux 中并非如此（至少默认情况下）。如果这两个主机名都不起作用，那么您可能需要找到要使用的确切主机名，为此您可以这样做或查看必须由 docker 为您创建的接口的 IP 地址。或者，如果您甚至没有看到界面，请尝试（如果需要使用 sudo）并查看是否有条目并记下 IP 地址。
localhosthost.docker.internal172.18.0.1host.docker.internalifconfig
ip addr showdocker0docker0docker network inspect bridge"Gateway"
