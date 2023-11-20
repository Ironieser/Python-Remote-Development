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
  - ```ssh-keygen -t rsa```
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
    - A bug:
        - ```shell
          C:\Users\admin>ssh container-12555
          CreateProcessW failed error:2
          posix_spawn: No such file or directory
          ```
        - Environment: window10 1909, PowerShell or CMD.
      - Solved:
        - ```shell
  
          Host gpu0
              HostName 192.168.4.129
              User user_name
              Port 22
          Host container-12555
              HostName 127.0.0.1
              Port 12555
              User root
              ProxyCommand c:/Windows\System32\OpenSSH/ssh.exe gpu0 -W %h:%p  # The key point.
              # ProxyJump gpu0
        ```
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
If the firewall banned all port but 22, you could use ```ssh -L``` at cmd/PowerShell of local host,
```
ssh -L 8888:localhost:8888 username@compute_node_ip -p 22
```

# WSL
- install wsl on your windows10
- install openssh-server on wsl
- ssh port:2222 (avoid port conflict), listening_add: 0.0.0.0,  set login in by password and pub key
-  ``` service ssh reload start status ```
- add them in .bashrc or .zshrc
- wsl: enable it to start on boot by running:```sudo update-rc.d ssh defaults```
- open windows10 port 2222
- add auto-start in windows10,
  ```kotlin
  @echo off
  netsh interface portproxy show all
  netsh interface portproxy rese
  netsh interface portproxy add v4tov4 listenaddress=192.168.0.196 listenport=2222 connectaddress=172.30.131.45 connectport=2222

  ```
  - Save the file with a .bat extension, for example, setup_port_forwarding.bat.
  - Press Win + R, type shell:startup, and press Enter to open the Startup folder.
  - Place a shortcut to your batch script in this folder.
- in your local pc/mac: ```ssh -p wsl_port wsl_user@window10_ip``` or add it in ```~/.ssh/config```

  # Proxy
  This is for setting remote Linux to use your local proxy
  - (such as v2rayN clash, check http/socks5 proxy port, v2rayN's default is 10808 for socks5, 10809 for https/http)
  - Turn on "LAN sharing proxy" of your proxy tool.
  - Adjust the firewall for your proxy core (such as vwray, xray, shadowsocks ), and open every option.
  ```bash
  export hostip=$(ip route | grep default | awk '{print $3}')
  export hostport=10809
  alias proxy='
      export HTTPS_PROXY="https://${hostip}:${hostport}";
      export HTTP_PROXY="http://${hostip}:${hostport}";
      export ALL_PROXY="socks5://${hostip}:10808";
      echo -e "Acquire::http::Proxy \"http://${hostip}:${hostport}\";" | sudo tee -a /etc/apt/apt.conf.d/proxy.conf > /dev/null;
      echo -e "Acquire::https::Proxy \"http://${hostip}:${hostport}\";" | sudo tee -a /etc/apt/apt.conf.d/proxy.conf > /dev/null;
  '
  alias unproxy='
      unset HTTPS_PROXY;
      unset HTTP_PROXY;
      unset ALL_PROXY;
      sudo sed -i -e '/Acquire::http::Proxy/d' /etc/apt/apt.conf.d/proxy.conf;
      sudo sed -i -e '/Acquire::https::Proxy/d' /etc/apt/apt.conf.d/proxy.conf;
  '
    
  ```
  Note: it  works for every LAN device, such as phone, PC, mac. Just add ```http_proxy=proxy_pc_ip:http_proxy_port```

  # Tool
  - Oh-my-zsh
    - wget ```sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"```
    - curl ```sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"```
    - plugins:
      - recommend theme: ys   
      - z
      - zsh-autosuggestions ``` git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions```
      - zsh-syntax-highlighting ```git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting```
      - vscode ```git clone https://github.com/valentinocossar/vscode.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/vscode```
