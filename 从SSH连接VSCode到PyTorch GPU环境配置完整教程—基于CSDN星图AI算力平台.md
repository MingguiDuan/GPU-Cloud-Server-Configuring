# 从SSH连接VSCode到PyTorch GPU环境配置完整教程——基于(CSDN星图AI)[https://ai.csdn.net/]平台

---

## 第一部分：VSCode通过SSH连接远程服务器
### 前置必看：SSH连接需要的4个核心信息
所有连接所需的信息，**全部在你购买GPU服务器的云平台后台获取**（你使用的是星图平台，阿里云/腾讯云/AutoDL等通用平台同理），必须拿到这4个信息才能完成连接：
1.  服务器**公网IP地址**（必须是公网IP，内网IP只能平台内部访问，本地电脑连不上）
2.  SSH连接端口（默认是22，多数云平台会提供自定义端口，比如10022、30000等）
3.  登录用户名（云服务器默认大多是root，部分镜像用ubuntu、admin等）
4.  登录凭证（二选一即可）：登录密码 / SSH密钥对的私钥文件

---

### 1. 手把手教你查找SSH连接信息
#### 通用版：所有云平台通用查找步骤（阿里云/腾讯云/AutoDL/恒源云等）
1.  打开你购买服务器的云平台官网，登录你的账号
2.  进入【控制台】/【实例管理】/【我的服务器】页面（不同平台名字略有差异，核心是找到你购买的GPU服务器列表）
3.  在服务器列表中，找到你要连接的那台服务器，点击服务器名称/【管理】按钮，进入服务器详情页
4.  在详情页的【实例信息】/【网络信息】板块，就能找到核心信息：
    - 公网IP地址：标注为「公网IP」「弹性IP」「外网IP」的字段，就是你需要的IP
    - SSH端口：标注为「SSH端口」「端口映射」的字段，没有标注就用默认22
    - 登录用户名：标注为「登录用户」「默认用户名」的字段，没有标注就默认用root
5.  登录凭证获取：
    - 密码：创建服务器时，平台会让你设置root密码；如果没设置，详情页会有【重置密码】按钮，点击设置你的登录密码即可
    - SSH密钥：创建服务器时，平台会让你绑定/创建SSH密钥对，私钥文件会在创建时让你下载到本地，公钥平台会自动配置到服务器里

#### 专属版：你使用的星图平台查找步骤
1.  打开星图平台官网，登录你的账号
2.  进入【算力市场】→【我的实例】页面，找到你正在运行的RTX 4090D服务器
3.  点击实例卡片上的【详情】按钮，进入实例详情页
4.  在【网络信息】板块，直接就能看到完整SSH连接信息：
    - 公网IP地址：页面标注的「SSH连接地址」中，@后面的数字就是IP，比如 `ssh root@123.45.67.89` 中，123.45.67.89就是公网IP
    - SSH端口：如果地址是 `ssh root@123.45.67.89 -p 12345`，那么-p后面的12345就是端口；没有-p参数，端口就是默认22
    - 登录用户名：@前面的root就是默认用户名
5.  登录密码：在实例详情页的【登录信息】板块，会显示初始密码，也可以点击【重置密码】修改成你自己的密码

---

### 2. 安装VSCode Remote - SSH插件
1.  打开本地电脑上的VSCode软件
2.  点击左侧侧边栏的「扩展图标」（长得像四个方块的图标，快捷键：Windows/Linux按Ctrl+Shift+X，Mac按Cmd+Shift+X）
3.  在顶部的搜索框中，输入 `Remote - SSH`
4.  在搜索结果中，找到**微软官方出品**的Remote - SSH插件（发布者是Microsoft），点击插件卡片上的【Install（安装）】按钮
5.  等待10秒左右，安装完成后，按钮会变成【已安装】，插件就准备就绪了

---

### 3. 配置SSH连接信息
#### 情况A：用密码登录（新手推荐，操作最简单）
1.  点击VSCode左下角的**绿色远程连接图标**（在窗口最左下角，长得像><的图标）
2.  弹出的命令面板中，选择【Remote-SSH: Connect to Host...】
3.  接下来选择【Add New SSH Host...】
4.  在弹出的输入框中，输入你的完整SSH连接命令，固定格式如下：
    ```bash
    ssh 用户名@公网IP地址 -p 端口号
    ```
    - 示例1（默认22端口）：`ssh root@123.45.67.89`
    - 示例2（自定义端口）：`ssh root@123.45.67.89 -p 12345`
    - 注意：把用户名、公网IP、端口号，替换成你刚才在平台后台查到的信息
5.  输入完成后按回车，接下来会让你选择SSH配置文件的保存位置，直接选择第一个（你电脑用户目录下的`.ssh/config`文件），按回车确认
6.  此时VSCode会提示「Host added!」，代表连接信息已经配置完成

#### 情况B：用SSH密钥登录（更安全，平台默认推荐）
1.  先找到你下载到本地的SSH私钥文件（通常是.pem格式，比如id_rsa、server_key.pem）
2.  打开VSCode，按F1打开命令面板，输入【Remote-SSH: Open SSH Configuration File...】，选择刚才的`.ssh/config`文件打开
3.  在文件中，粘贴以下配置，把<>里的内容替换成你自己的信息：
    ```
    Host 我的GPU服务器（可以自定义名字，比如RTX4090D-星图）
      HostName 你的公网IP地址
      Port 你的SSH端口号
      User 你的登录用户名
      IdentityFile 你本地私钥文件的完整路径
    ```
    - 完整示例：
    ```
    Host RTX4090D-星图
      HostName 123.45.67.89
      Port 12345
      User root
      IdentityFile C:/Users/张三/.ssh/starcloud_key.pem
    ```
4.  按Ctrl+S保存文件，配置就完成了

---

### 4. 连接到远程服务器
1.  再次点击VSCode左下角的绿色远程连接图标
2.  弹出的列表中，选择你刚才配置好的服务器（就是Host后面的名字，比如RTX4090D-星图）
3.  点击后，VSCode会打开一个新的窗口，首次连接会在服务器上自动安装VSCode Server，等待10-30秒完成
4.  密码登录：会弹出输入框，让你输入服务器的登录密码，输入后按回车即可
5.  密钥登录：如果配置正确，会直接跳过密码，自动完成连接
6.  **连接成功标志**：VSCode左下角的绿色图标，会显示「SSH: 你的服务器名字/IP」，此时你已经成功在VSCode里操作远程GPU服务器了！

---

### 5. 常见连接失败排查
1.  提示「连接超时」：检查是不是用了内网IP，必须用公网IP；检查端口号是否正确；检查本地网络能不能正常访问公网
2.  提示「权限拒绝」：检查用户名、密码是否输入正确；私钥文件路径是否正确；Mac/Linux系统需要给私钥设置权限，执行`chmod 600 你的私钥文件路径`
3.  提示「主机不可达」：去云平台后台，检查服务器是否处于「运行中」状态，关机的服务器无法连接

---

## 第二部分：环境排查与问题定位（核心前置步骤，必做！）
安装PyTorch之前，必须先确认服务器硬件和基础环境，避免盲目操作踩坑。
### 操作入口：
VSCode连接成功后，点击顶部菜单栏 → 【Terminal（终端）】→ 【New Terminal（新建终端）】，就能打开服务器的命令行终端，所有命令都在这里输入执行。

---

### 1. 确认服务器显卡是否存在、驱动是否正常（必做第一步）
#### 执行命令：
```bash
nvidia-smi
```
#### 命令解释：
`nvidia-smi` 全称NVIDIA System Management Interface，是NVIDIA显卡的官方管理工具，用来查看显卡型号、驱动版本、显存使用、CUDA支持上限等核心硬件信息，是确认GPU可用的第一步。

#### 怎么看输出结果：
##### 正常输出示例（你的RTX 4090D）：
```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 550.90.07    Driver Version: 550.90.07    CUDA Version: 12.4 |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  Off  | 00000000:5A:00.0 Off |                    0 |
| N/A   32C    P8    26W / 425W |      1MiB / 23028MiB |      0%      Default |
|                               |                      |                  N/A |
+-----------------------------+----------------------+----------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```
##### 关键信息提取&确认：
1.  **Driver Version: 550.90.07**：显卡驱动版本，这个决定了你的服务器最高能支持的CUDA版本
2.  **CUDA Version: 12.4**：驱动最高支持的CUDA版本（注意：这不是系统已经安装的CUDA Toolkit版本，PyTorch自带CUDA runtime，不用自己单独装CUDA Toolkit）
3.  **NVIDIA GeForce RTX 4090 D**：显卡型号，确认是你购买的显卡
4.  **23028MiB**：显存大小（约24GB），确认显存符合你的购买规格
5.  **GPU 0**：代表服务器识别到了第1张显卡，有几张显卡就会显示几个GPU编号

##### 异常情况处理：
- 如果执行命令提示「command not found」：说明服务器没有安装NVIDIA驱动，或者没有给你挂载GPU
- 如果输出「No devices were found」：说明服务器没有识别到显卡，GPU没有挂载成功
- 以上两种情况，直接去云平台后台，提交工单联系客服，让技术人员帮你挂载GPU、安装驱动，这是平台的问题，你自己无法修复。

---

### 2. 确认系统Python版本
#### 执行命令：
```bash
python3 --version
# 备选命令（如果上一条无响应）
python --version
```
#### 命令解释：
- `python3`：调用系统预装的Python 3解释器
- `--version`：参数，让Python输出自己的版本号
#### 正常输出示例：
```
Python 3.12.3
```
#### 异常情况处理：
- 如果提示「command not found」：说明系统没有安装Python，执行以下命令安装：
  ```bash
  apt update
  apt install -y python3 python3-pip
  ```

---

## 第三部分：RTX 4090D PyTorch GPU环境配置（完美适配版）
### 核心避坑原则（基于实际踩坑总结）
1.  必须彻底清理旧版本残留，否则pip会优先使用缓存的错误版本
2.  必须强制指定PyTorch版本和CUDA版本，禁止pip自动安装最新版
3.  CUDA版本必须 ≤ 驱动支持的最高版本（驱动支持12.4，实际兼容上限12.0.40，选cu120最稳定）
4.  Ubuntu 24.04开启了系统保护，必须加--break-system-packages参数才能用pip安装系统包

### 1. 彻底卸载所有旧版本torch相关包（执行2次，确保无残留）
执行命令：
```bash
# 第一次卸载
pip3 uninstall -y torch torchvision torchaudio
# 第二次卸载（清除残留）
pip3 uninstall -y torch torchvision torchaudio
```
命令解释：
- pip3 对应Python3的包管理器
- uninstall 卸载命令
- -y 参数，自动确认所有卸载操作，无需手动输入y确认
- torch torchvision torchaudio 要卸载的三个核心包（PyTorch核心、视觉库、音频库）

### 2. 清除pip缓存（彻底杜绝旧包干扰，必做！）
执行命令：
```bash
pip3 cache purge
```
命令解释：
- cache pip的缓存管理命令
- purge 参数，删除pip下载的所有缓存安装包，避免旧版本包干扰新安装

### 3. 强制安装完美匹配的GPU版PyTorch（指定版本，零兼容问题）
执行命令：
```bash
pip3 install torch==2.4.0 torchvision==0.19.0 torchaudio==2.4.0 --index-url https://download.pytorch.org/whl/cu120 --extra-index-url https://pypi.tuna.tsinghua.edu.cn/simple --break-system-packages
```
命令逐段解释：
- torch==2.4.0 强制安装PyTorch 2.4.0稳定版（==代表精确锁定版本号）
- torchvision==0.19.0 强制安装与PyTorch 2.4.0完全匹配的视觉库版本
- torchaudio==2.4.0 强制安装与PyTorch 2.4.0完全匹配的音频库版本
- --index-url https://download.pytorch.org/whl/cu120 核心参数，指定主下载源为PyTorch官方CUDA 12.0版本仓库（cu120=CUDA 12.0）
- --extra-index-url https://pypi.tuna.tsinghua.edu.cn/simple 指定清华镜像为额外加速源，解决海外网络下载慢的问题
- --break-system-packages 必加参数，绕过Ubuntu 24.04的PEP 668系统保护，允许pip直接安装包到系统Python

### 4. 最终验证（GPU可用必出True）
执行命令：
```bash
python3 -c "import torch; print('PyTorch版本:', torch.__version__); print('编译CUDA版本:', torch.version.cuda); print('GPU可用:', torch.cuda.is_available())"
```
命令解释：
- -c "..." 参数，让Python直接执行引号内的代码，无需创建.py文件
- import torch 导入PyTorch库
- torch.__version__ 输出PyTorch的版本号（双下划线__代表Python特殊属性）
- torch.version.cuda 输出PyTorch编译时使用的CUDA版本
- torch.cuda.is_available() 检测GPU是否可用，正常返回True/False

完美输出示例：
PyTorch版本: 2.4.0+cu120
编译CUDA版本: 12.0
GPU可用: True

### 5. 安装项目全量依赖
执行命令：
```bash
pip3 install transformers datasets pillow pandas numpy opencv-python accelerate scikit-learn --break-system-packages --extra-index-url https://pypi.tuna.tsinghua.edu.cn/simple
```
包用途说明：
- transformers Hugging Face模型库，加载XLM-RoBERTa等预训练语言模型
- datasets Hugging Face数据集库，加载和处理训练数据
- pillow Python基础图像处理库
- pandas 表格数据处理与分析库
- numpy Python科学计算基础库
- opencv-python 计算机视觉库，进阶图像处理
- accelerate PyTorch训练加速库，简化GPU/混合精度训练
- scikit-learn 机器学习库，用于模型评估指标计算

---

## 第四部分：VSCode中运行代码
环境配置完成后，按以下步骤运行代码：
1.  左侧文件资源管理器空白处右键 → New File（新建文件）
2.  命名为run.py（可自定义名称，后缀必须为.py）
3.  在文件中编写代码并保存
4.  下方终端中执行命令运行代码：
```bash
python3 run.py
```

---

## 总结与注意事项
1.  先确认硬件：永远先执行nvidia-smi确认显卡正常，再安装软件
2.  版本匹配是核心：PyTorch的CUDA版本必须≤驱动支持的最高CUDA版本
3.  清缓存是关键：更换版本前必须执行pip3 cache purge，避免旧包干扰
4.  强制锁定版本：不要让pip自动选择最新版，手动指定稳定版和对应CUDA版本
5.  系统保护参数：Ubuntu 24.04系统必须加--break-system-packages参数
