# Python-Remote-Development
The introduction for configuring the remote development with Pycharm or VSCode. 
[[中文版](https://github.com/Ironieser/Python-Remote-Development/blob/main/README_chs.md)][[英文版](https://github.com/Ironieser/Python-Remote-Development/blob/main/README.md)]
# Environment
## Platform
  - Windows 10/11
  - Pycharm 2023.1 professional.
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
# Analysis and Solution  
  - First of all, please config ../.ssh/config. Suggest using the default fold to save id_rsa and id_ras.pub.  
    ```
      C:\Users\account_name\.ssh
    
      #-------example--------
      C:\Users\ironieser\.ssh
      #----------------------
    ```
    ref: https://zhuanlan.zhihu.com/p/28423720
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
      - Need upload id_rsa.pub to login01, gpu0 and the container.
  - Login host
    - Support straight connection
  - Computer host
    1. Support LAN and WAN
    2. Only Support LAN.
      - Need Port forwarding  
        - ``` ssh -J login0 gpu0```
        - ``` ssh -J login0 gpu0```
       
  - Docker
    - Run with port forwarding
      ```
      docker run -it -v /public/homes/<usr_name>/project:/workdir  -w /workdir -p <contrainer_ssh_port_forward>:<container_ssh_port_listenning> --name=<your_container_name> <image_name>:<image_tag> <your shell>

      #-------example--------
      podman run -it -v /public/homes/ironieser/project:/workdir  -w /workdir -p 12555:2555 --name=talklip1 localhost/ironieser:zsh-cuda11.3-cudnn8-devel-ubuntu20.04 /bin/zsh
      #----------------------
      ```
      
    - Open ssh forwarding in the local host(WAN)
      ```
      ssh -N -f -L <local_host port>:<docker_host>:<container port> -p <computer_host port> name@<computer_host> -o TCPKeepAlive=yes
      ssh -N -f -L 6001:192.168.4.129:12555 -p 22 ironieser@192.168.4.129
      ```
    - Add ssh in pycharm  
      ```host IP: 127.0.0.1, port: 6001, name: root <container user name>```  
    - Now, we can connect the computer host or container from WAN.

# TensensorBoard / Jupyter / webui
```
ssh -N -f -L <local_host port>:<computer_host>:<computer_port> -p <login_host port> name@<login_host> -o TCPKeepAlive=yes
```
If the firewall banned all port but 22, you could use ```ssh -L```
```
ssh -L 8888:localhost:8888 username@compute_node_ip -p 22
```
