# 外贸电视盒子H96MAX RK3318盒子安装Armbian镜像教程

## H96Max RK3318 armbian刷机教程以及安装docker和HASS的教程

### 0.准备工作

一台电脑，一个支持HDMI的显示屏，一张大于16G的TF卡，读卡器或SD卡套，USB接口的键盘(插在盒子上使用)以及需要的镜像文件、刷机工具、烧录工具和远程SHH工具。

### 1.下载Armbian镜像和刷机工具

使用作者提供的镜像和刷机工具或[进入](https://users.armbian.com/jock/rk3318/)Armbian镜像网站下载下载最新的镜像和刷机工具。

### 2.安装armbian[^1]

1. 使用`balenaEtcher`将`multitool.img`烧录到TF卡上。烧录完成后，把镜像文件复制进`MULTITOOL`分区的`images`文件夹里；

2. 将TF卡插入电视盒，连接至显示屏并插入键盘和电源线。几秒钟后，蓝色 LED 开始闪烁并出现 `Multitool`；

3. 从菜单中选择 “`Erase flash`” 格式化盒子的所有分区；

4. 选择 “`Burn image to flash`”，然后选择目标设备（通常为 `mmcblk2`）和要刻录的镜像；

5. 等待刷机过程完成，然后从主菜单中选择 “`Shutdown`”；

6. 拔掉电源线和TF卡，将然后重新插上电源线；

7. 等待 10 秒，然后 LED 应该开始闪烁并且 HDMI 将打开。第一次启动过程需要几分钟或更长时间，因为文件系统将被调整大小，所以请耐心等待登录提示。

8. 首次启动时，系统会要求您输入您选择的 `root` 用户的密码以及普通用户的名称和密码并选择时区和 `WiFi`。也可以在设置完root密码后按 `ctrl+c` 退出设置(可以用 `armbian-config` 设置时区，语言，WiFi和其他个人设置)；

9. 运行 `rk3318-config` 以配置板特定选项(可以更改CUP的运行最大频率和EMMC的频率以及板子的灯的设置)；

10. 更新包索引和升级包：

    ```shell
    sudo apt update
    sudo apt upgrade
    ```

11. 安装 `vim` ：

    ```shell
     sudo apt install vim -y
    ```

12. 安装 `git`：

    ```shell
    sudo apt install git -y
    ```

13. 安装 `pip`：

    ```shell
    sudo apt install python3-pip -y
    ```

14. 安装`armbian-config` ：在命令行输入 

    ```shell
    sudo apt install armbian-config -y
    ```

15. 命令行输入 `reboot ` 重启设备；

16. 如果前面没有设置时区等，通过 `armbian-config` 命令进入设置，选择 `Network` 和 `Personal` 设置WiFi和时区等；

17. 在`System`中可以选择 `CPU` 选项设置 `CPU` 最大频率和运行模式；

18. 恭喜你完成了 `Armbian` 的安装！！！

### 3、安装Docker

1. 如果安装过 `Docker` ，先卸载：

    ```shell
    sudo apt-get remove docker docker-engine docker.io containerd runc
    ```

2. 新刷的系统没有安装过 `Docker`，先安装依赖： 

    ```shell
    sudo apt-get install apt-transport-https ca-certificates curl gnupg2 software-properties-common
    ```

3. 信任 `Docker` 的 `GPG 公钥`  ： 

    ```shell
    curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    ```

4. 添加软件仓库:

    ```shell
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list
    ```

5. 安装 `Docker` ：

    ```shell
    sudo apt update
    sudo apt install docker-ce -y
    ```

6. 换上阿里云的镜像加速：

    ```shell
    sudo mkdir -p /etc/docker
    sudo tee /etc/docker/daemon.json <<-'EOF'
    {
      "registry-mirrors": ["https://0q59jibj.mirror.aliyuncs.com"]
    }
    EOF
    ```

    **P.S. **阿里镜像加速请换成自己的[^2]

7. 重新加载`daemon.json`文件内容并重启`Docker` ：

    ```shell
    sudo systemctl daemon-reload
    sudo systemctl restart docker
    ```

8. 进入 `resolv.conf` 修改 `DNS`： `vim /etc/resolv.conf`：

    按 `a` 或 `i` 进入编辑模式，在后面追加

    ```shell
    nameserver 8.8.8.8
    nameserver 8.8.4.4
    ```

    按  `esc` 键进入命令模式，输入`:wq` 或 按`shift+z z` 保存并退出。

### 4.安装HomeAssistant[^3]

1. 搜索镜像：

    ```Shell
    docker search home-assistant
    ```

2. 可以看到排在第一的 homeassistant/home-assistant 的星标最多，我们选择下载它：

    ```shell
    docker pull homeassistant/home-assistants
    ```

3. 创建容器：

    ```shell
    docker run -d --name="hass" -v ${HOME}/hass/mac_config -p 8123:8123 homeassistant/home-assistant
    ```

    - d：表示在后台运行
    - name：给容器设置别名（不然会随机生成，为了方便管理）；
    - v：配置数据卷（容器内的数据直接映射到本地主机环境，参考路径配置）；
    - p：映射端口（容器内的端口直接映射到本地主机端口最后便是刚才下载的镜像了，运行该容器）。

    可以把-v后面的路径改成你的本地存放该容器配置路径，运行成功会生成一串容器id；

4. 查看运行状态：

    ```shell
    docker ps
    ```

    有创建容器时指定的name的记录表示已经运行成功，直接打开 127.0.0.1:8123 可以进入配置你的 HomeAssistant ；

5. 启动/停止容器：

    ```shell
    ## 启动
    docker start hass
    ## 停止
    docker stop hass
    ```

6. **未完待续**



## 参考:

[^1]:https://www.znds.com/tv-1203362-1-1.html
[^2]:https://blog.csdn.net/qq_33973359/article/details/108320573
[^3]:https://blog.csdn.net/m0_67391683/article/details/124103709



