##Docker服务端的防护
运行一个容器或应用程序的核心是通过Docker服务端。Docker服务的运行目前需要root权限，因此其安全性十分关键。

首先，确保只有可信的用户才可以访问Docker服务。Docker允许用户在主机和容器间共享文件夹，同时不需要限制容器的访问权限，这就容易让容器突破资源限制。例如，恶意用户启动容器的时候将主机的根目录`/`映射到容器的`/host`目录中，那么容器理论上就可以对主机的文件系统进行任意修改了。这听起来很疯狂？但是事实上几乎所有虚拟化系统都允许类似的资源共享，而没法禁止用户共享主机根文件系统到虚拟机系统。

这将会造成很严重的安全后果。因此，当提供容器创建服务时（例如通过一个web服务器），要更加注意进行参数的安全检查，防止恶意的用户用特定参数来创建一些破坏性的容器

为了加强对服务端的保护，Docker的REST API（客户端用来跟服务端通信）在0.5.2之后使用本地的Unix套接字机制替代了原先绑定在127.0.0.1上的TCP套接字，因为后者容易遭受跨站脚本攻击。现在用户使用Unix权限检查来加强套接字的访问安全。

用户仍可以利用HTTP提供REST API访问。建议使用安全机制，确保只有可信的网络或VPN，或证书保护机制（例如受保护的stunnel和ssl认证）可以进行访问。此外，还可以使用HTTPS和证书来加强保护。

最近改进的Linux名字空间机制将可以实现使用非root用户来运行全功能的容器。这将从根本上解决了容器和主机之间共享文件系统而引起的安全问题。

终极目标是改进2个重要的安全特性：
* 将容器的root用户映射到本地主机上的非root用户，减轻容器和主机之间因权限提升而引起的安全问题；
* 允许Docker服务端在非root权限下运行，利用安全可靠的子进程来代理执行需要特权权限的操作。这些子进程将只允许在限定范围内进行操作，例如仅仅负责虚拟网络设定或文件系统管理、配置操作等。

最后，建议采用专用的服务器来运行Docker和相关的管理服务（例如管理服务比如ssh 监控和进程监控、管理工具nrpe、collectd等）。其它的业务服务都放到容器中去运行。

