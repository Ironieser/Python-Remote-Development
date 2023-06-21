# Python 远程开发
介绍如何在windows系统上配置python的远程开发，IDE为pycharm，或者使用vscode作为编辑器。
[[中文版](https://github.com/Ironieser/Python-Remote-Development/blob/main/README_chs.md)][[英文版](https://github.com/Ironieser/Python-Remote-Development/blob/main/README.md)]
# 环境
## Platform
  - Windows 10/11
  - Pycharm 2023.1 专业版.
  - Vscode
## Remote host
  - Login host, (called login0)
    - Port 22 is open for the user (only)
    - Support ssh connection with port 22 from WAN.
  - computer host, (called gpu0)
    - Conda environment, python
    - Port 22 is open for the user (only)
    - support ssh connection from WAN
      - May not support being connected from WAN, similar to the container.
  - the docker's container, which runs on the computer host (called container)
    - Conda environment, python   
    - It cannot be connected by the user from the outside of the computer host (or LAN).
# 分析与解决  
  - 首先, 需要配置本地的ssh默认配置文件 ../.ssh/config. 建议使用默认目录保存公钥和私钥，以下均为默认路径下的示例.  
    ```
      C:\Users\account_name\.ssh
    
      #-------example--------
      C:\Users\ironieser\.ssh
      #----------------------
    ```
    参照: https://zhuanlan.zhihu.com/p/28423720
    ```config
    Host login01
        HostName 10.15.15.15
        User your_name
        Port your_port
    #-------example--------
    Host login01
        HostName 10.15.15.15
        User ironieser
        Port 223
    #----------------------
    ```
      - 需要将公钥文件id_rsa.pub 上传到登陆节点，计算节点和容器内（打包镜像保存）。
  - 登陆节点/堡垒机
    - Support straight connection
  - 计算节点
    1. 支持局域网或公网访问
    2. 仅支持局域网访问
      - 需要端口转发或多级跳转，下面为多级跳转示例   
        - ``` ssh -J login0 gpu0```
        - ``` ssh -J login0 gpu0```
       
  - Docker
    - 运行容器，并同时开启端口转发，如示例，将端口12555转发到端口2555，容器内配置ssh端口为2555，实现通过连接12555，进而连接容器。  
      ```
      docker run -it -v /public/homes/<usr_name>/project:/workdir  -w /workdir -p <contrainer_ssh_port_forward>:<container_ssh_port_listenning> --name=<your_container_name> <image_name>:<image_tag> <your shell>

      #-------example--------
      podman run -it -v /public/homes/ironieser/project:/workdir  -w /workdir -p 12555:2555 --name=talklip1 localhost/ironieser:zsh-cuda11.3-cudnn8-devel-ubuntu20.04 /bin/zsh
      #----------------------
      ```
      
    - 在本地cmd/shell中开启端口转发
      ```
      ssh -N -f -L <local_host port>:<computer_host>:<container port> -p <computer_host port> name@<computer_host> -o TCPKeepAlive=yes
      ssh -N -f -L 6001:192.168.4.129:12555 -p 22 ironieser@192.168.4.129
      ```
    - 在pycharm中添加ssh配置   
      ```host IP: 127.0.0.1, port: 6001, name: root <container user name>```  
    - 现在，可以从公网直接访问计算节点或是容器了，并且支持远程调试  
      
