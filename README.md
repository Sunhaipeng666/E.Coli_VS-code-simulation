# Vivarium *E. coli*

![vivarium](doc/_static/ecoli_master_topology.png)

## Background

Vivarium *E. coli* (vEcoli) is a port of the Covert Lab's 
[E. coli Whole Cell Model](https://github.com/CovertLab/wcEcoli) (wcEcoli)
to the [Vivarium framework](https://github.com/vivarium-collective/vivarium-core).
Its main benefits over the original model are:

1. **Modular processes:** easily add/remove processes that interact with
    existing or new simulation state
2. **Unified configuration:** all configuration happens through JSON files,
    making it easy to run simulations/analyses with different options
3. **Parquet output:** simulation output is in a widely-supported columnar
    file format that enables fast, larger-than-RAM analytics with DuckDB
4. **Google Cloud support:** workflows too large to run on a local machine
    can be easily run on Google Cloud

As in wcEcoli, [raw experimental data](reconstruction/ecoli/flat) is first processed
by the parameter calculator or [ParCa](reconstruction/ecoli/fit_sim_data_1.py) to calculate 
model parameters (e.g. transcription probabilities). These parameters are used to configure
[processes](ecoli/processes) that are linked together into a
[complete simulation](ecoli/experiments/ecoli_master_sim.py).

## Setup

**本地运行主要有三个部分：安装 Windows Subsystem for Linux (WSL)、克隆仓库到指定路径，以及复现仓库的运行流程。**

**（成功运行之后会遇到一个环境清理问题，而不是程序运行问题。错误分析：**

**RuntimeError: Output directory already exists: /home/sunhaipeng/projects/vEcoli/out/test\_installation. Please use a different experiment ID or output directory. Alternatively, move, delete, or rename the existing directory.**

**这是因为您之前运行模拟时，Nextflow 已经创建了输出目录 /home/sunhaipeng/projects/vEcoli/out/test\_installation。**

**现在您重新运行，脚本为了保证结果的完整性和不混淆历史数据，拒绝覆盖已存在的目录。**

**解决方案：删除旧的输出目录您只需要删除或重命名这个旧的输出文件夹，就可以再次运行模拟。**

**在您的 WSL (Ubuntu) 终端中，运行以下命令来删除旧的输出目录：**

**Bash**

**#导航回工作目录cd ~/projects/vEcoli**

**rm -rf out/test\_installation**

**#rm -rf: 强制递归删除文件夹及其所有内容，out/test\_installation: 需要删除的目标文件夹。**

**最终删除旧的输出目录后，请再次运行您的工作流命令**)



**1. 在 Windows 11 上安装 Windows Subsystem for Linux (WSL)**

以管理员身份打开 PowerShell 或 Windows Terminal：

点击“开始”按钮，输入 Terminal 或 PowerShell。

右键点击搜索结果，选择\*\*“以管理员身份运行”\*\*。

运行 WSL 安装命令：

在打开的终端窗口中，运行以下命令：

PowerShell



wsl --install

此命令将自动启用所需的 WSL 功能、下载并安装最新的 WSL 2 架构，以及默认的 Ubuntu Linux 发行版。



重启电脑：安装完成后，系统会提示您重启电脑以完成安装。

首次启动 Ubuntu 并设置用户：

电脑重启后，Ubuntu 终端窗口会自动弹出。

您将被要求创建一个 Linux 用户名和密码。请记下这些信息，它们将用于您在 Linux 环境中执行管理任务（如 sudo 命令）。

更新 Linux 系统（可选但推荐）：

进入 Ubuntu 终端后，运行以下命令更新软件包列表：

Bash



sudo apt update \&\& sudo apt upgrade -y

-------------------------------------------------------------

**2. 准备环境与克隆仓库**



\*\*\*\*准备环境

Vivarium E. coli 仓库的运行依赖 Nextflow，而 Nextflow 需要 Java 环境，同时仓库推荐使用 uv 作为 Python 包管理器。

安装 Git 和 Java：

在您的 WSL (Ubuntu) 终端中运行以下命令：

Bash



\# 安装 Git (用于克隆仓库) 和 Java 17 (Nextflow 依赖)

sudo apt install git openjdk-17-jdk -y

安装 uv Python 包管理器：在 WSL 终端中运行以下命令，使用官方推荐的独立安装程序：

Bash



curl -LsSf https://astral.sh/uv/install.sh | sh

安装程序可能会提示您将 uv 的安装路径添加到您的 PATH 环境变量中。

通常您需要按照提示将类似 export PATH="$HOME/.cargo/bin:$PATH" 的行添加到您的 ~/.bashrc 或 ~/.zshrc 文件中，然后运行 source ~/.bashrc 使其生效。

您正在使用 WSL 中的 Ubuntu，默认的 Shell 是 Bash 或 Zsh，所以您应该使用第一种方法。

以下是完整的步骤，用于将 $HOME/.local/bin 添加到您的 PATH 环境变量中，使其永久生效：

永久生效（推荐）

为了确保您每次打开新的 WSL 终端时，uv 都能被识别，您需要将这条配置添加到您的 Shell 启动文件（通常是 ~/.bashrc 或 ~/.zshrc）中。

打开您的 Shell 配置文件：

如果您使用的是 Bash (默认：首选)，运行：

Bash



nano ~/.bashrc

如果您使用的是 Zsh，运行：

Bash



nano ~/.zshrc

添加配置行：使用方向键将光标移动到文件末尾。在文件末尾添加以下行（如果您在第 1 步使用的是 ~/.bashrc，则添加到 ~/.bashrc 中）：

Bash



\# uv installer adds the $HOME/.local/bin directory to PATH

source $HOME/.local/bin/env

保存并退出：在 nano 编辑器中，按 Ctrl + O (写入) → 按 Enter 确认文件名 → 按 Ctrl + X (退出)。

重新加载配置：

在终端中运行以下命令使更改永久生效：

Bash



source ~/.bashrc  # 如果您编辑的是 ~/.zshrc，则改为 source ~/.zshrc



验证 PATH 路径是否加入成功

在您的 WSL (Ubuntu) 终端中，完成以下任一操作即可：

1\. 检查 uv 版本 (最简单的方法)

如果 uv 命令可以直接运行，则说明它的路径已成功添加到 PATH 中。

Bash



uv --version

如果成功： 您会看到类似这样的输出（版本号可能会不同），这表示 uv 已被系统识别：



uv 0.8.22

如果失败： 您会看到类似这样的错误信息：

Bash



uv: command not found



\*\*\*\*\*\*克隆仓库

---	-----------------------------------	-----------------------------------	----------------------------------	------------------------------

（（（您需要在 WSL 环境中安装必要的工具，并最好将文件系统克隆到linux文件目录下边。（（（文件系统隔离： WSL 2 的 Linux 环境是一个完整的虚拟机。它将自己的 Linux 文件系统存储在 Windows 上的一个大型虚拟硬盘文件（VHDX）中，这个文件默认路径在类似 C:\\Users\\<UserName>\\AppData\\Local\\Packages 下。性能差异：当您在 Linux 文件系统内部 (/home/sunhaipeng/... 或 /) 执行 I/O 密集型操作（如 git clone、npm install、运行复杂的模拟或大量读写文件）时，性能是极佳的，接近原生 Linux。当您在 Windows 文件系统 (/mnt/c/、/mnt/d/ 等) 上执行这些操作时，WSL 必须通过一个网络层在 Linux 和 Windows 之间转换文件操作。这会导致显著的性能下降，尤其是在处理大量小文件或进行 I/O 密集型操作时。您的操作： git clone 就是一个典型的 I/O 密集型操作，它会创建数千个文件。因此，WSL 警告您在这个 Windows 挂载点 (/mnt/d/...) 上进行操作会导致性能不佳。））如何访问 Linux 文件系统中的文件？您可能会担心文件转移到 Linux 后，如何用 Windows 上的文件浏览器或 VS Code 打开它们。您可以使用 Windows 10/11 提供的 WSL 文件系统集成：在 Windows 文件资源管理器地址栏中输入：



\\\\wsl$

在弹出的文件夹中，您会看到一个名为 Ubuntu（或其他您安装的发行版名称）的文件夹。

进入 Ubuntu -> home -> sunhaipeng -> projects -> vEcoli，即可像访问任何 Windows 文件夹一样访问您的项目文件。）

------------------------	---------------------------------------	---------------------------------------	-----------------------------------------	-----------

切换到您的 Linux 主目录：

这是 WSL 默认推荐的高性能工作区。

Bash



cd ~  # 切换到 /home/sunhaipeng/

在 Linux 文件系统下创建并进入项目文件夹：

我们重新创建一个项目文件夹，并命名为 projects (它将位于 Linux 内部，与 Windows 的 D 盘无关)。

Bash



mkdir -p projects/

cd projects/

重新克隆仓库：

这次克隆操作将在 Linux 内部完成，性能会更快，运行模拟时也不会有 I/O 瓶颈。

Bash



git clone (https://github.com/Sunhaipeng666/E.Coli_VS-code-simulation.git)

cd vEcoli
3. 运行 Vivarium E. coli 仓库

进入仓库文件夹后，请按照 README.md 文档的指示，使用 uv 来设置环境和运行 Nextflow 工作流。

初始化项目并创建虚拟环境：

uv 会读取项目配置文件并创建一个隔离的虚拟环境。

Bash



uv init

\*\*\*（（（sunhaipeng@legion:~/projects/vEcoli$ uv init

error: Project is already initialized in `/home/sunhaipeng/projects/vEcoli` (`pyproject.toml` file exists)）

这是一个很好的信号！这个错误提示实际上告诉您，您的项目环境已经部分准备好了，您不需要再运行 uv init 了。为什么会出现这个提示？

当您运行 git clone 将仓库克隆下来时，Vivarium E. coli 仓库（和其他许多 Python 项目一样）很可能已经包含了项目配置文件 pyproject.toml。

uv init 的作用就是在当前文件夹创建这个文件。由于它已经存在，uv 报告了错误，防止覆盖您现有的配置。

接下来要做的事情

既然项目已经被识别，您只需要直接跳过初始化步骤，进行下一步：安装 Nextflow 依赖并运行工作流。））

安装 Nextflow 依赖：

将 Nextflow 添加到您的项目依赖中。

Bash



uv add nextflow

运行安装测试工作流：

运行 uv run 命令来执行 Nextflow 工作流，并使用 -profile test\_installation 运行完整的测试流程。

Bash



uv run python runscripts/workflow.py --config configs/test\_installation.json

--------------------------------------------------------------------------------------

程序可以正常运行，但您的 WSL 环境没有提供足够的内存来满足模拟对内存的最低要求。

错误信息如下：

ERROR ~ Error executing process > 'runParca'

Caused by:

&nbsp; Process requirement exceeds available memory -- req: 8 GB; avail: 6.1 GB

所需内存 (req): runParca 进程（运行参数计算器，这是模拟的第一步）要求 8 GB 内存。可用内存 (avail): 您的 WSL 2 环境默认配置的可用内存只有 6.1 GB。

因此，Nextflow 拒绝启动这个进程，因为它知道进程会因内存不足而崩溃。

解决方案：增加 WSL 2 的内存限制

您需要修改 WSL 2 的配置文件 .wslconfig 来增加它可用的内存。在 Windows 11 中，WSL 默认限制了可以分配给 Linux 环境的内存量，通常是您物理内存的 50% 或 8GB（取较小值）。

步骤 1：创建或修改 .wslconfig 文件。

打开您的 Windows 文件资源管理器，导航到您的 Windows 用户目录。这个目录通常是：C:\\Users\\你的用户名

在该目录下，您需要创建一个名为 .wslconfig 的文件（C:\\Users\\legion\\.wslconfig）。(注意： 文件名是 .wslconfig，前面有一个点。如果您看不到文件扩展名，请在“查看”选项卡中勾选“文件扩展名”。)

用任何文本编辑器（如记事本、VS Code 或 Notepad++）打开或创建 .wslconfig 文件。

在文件中添加以下内容：

Ini, TOML



\[wsl2]

\# 将此值设置为您想要分配给 WSL 的内存量，例如 12GB (12288MB)

\# 您的物理内存应大于此值。

memory=12GB 

\# 也可以设置分配的虚拟 CPU 核心数

\# processors=4 



重要提示： 请将 memory 的值设置为 8GB 以上，例如 12GB。请确保这个值不超过您 Windows 电脑的物理内存总量（通常建议留出 2-4GB 给 Windows 系统本身）。如果您的电脑总内存只有 8GB，您可能需要将该值设置为 6GB，但这可能仍然不够运行 ParCa。

步骤 2：关闭 WSL 虚拟机。

打开一个新的 Windows Terminal 窗口：

在您的 Windows 11 电脑上，点击“开始”按钮。输入 Terminal 或 PowerShell。无需以管理员身份运行。

在 Windows Terminal 中运行关闭命令：

在打开的 PowerShell 或 Command Prompt 标签页中，输入以下命令：

PowerShell



wsl --shutdown

这将强制关闭所有正在运行的 WSL 实例，使您对 .wslconfig 文件的更改生效。



2\. 验证内存并重新运行模拟

重新打开您的 WSL (Ubuntu) 终端。

验证内存是否已增加：

在 Ubuntu 终端中，运行 free -h。确认 Mem 一行中的总量（total）已增加到您在 .wslconfig 文件中设置的值（例如 12G 或 16G）。

再次运行工作流命令：

回到您克隆的仓库目录 

Bash



cd ~/projects/vEcoli。

运行工作流启动命令：

Bash



uv run python runscripts/workflow.py --config configs/test\_installation.json

这次，有了足够的内存（大于 8 GB），runParca 进程应该能成功启动，并继续执行您的生物模拟。

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

运行流程概述： 这个测试流程将执行以下步骤，并将所有输出保存到 out/test\_installation 文件夹中：

1\.运行参数计算器（ParCa）以生成仿真数据。

2..运行仿真模拟（单一代）。

3\.通过创建质量分数图来分析仿真输出。



验证运行结果

如果 Nextflow 工作流没有错误地完成，显示：

Specified generations: initial lineage seed 0, 1 initial sims

&nbsp;N E X T F L O W   ~  version 25.04.7

Launching `/home/sunhaipeng/projects/vEcoli/nextflow\_temp/test\_installation/main.nf` \[agitated\_maxwell] DSL2 - revision: 209e33af87

executor >  local (4)

\[6b/d3d17b] runParca                                                          | 1 of 1 ✔

\[48/df9e75] createVariants                                                    | 1 of 1 ✔

\[e0/e619cd] sim\_gen\_1 (variant=0/seed=0/generation=1/agent\_id=0)              | 1 of 1 ✔

\[97/2d1fef] analysisSingle (variant=0/lineage\_seed=0/generation=1/agent\_id=0) | 1 of 1 ✔

Completed at: 25-Sep-2025 13:10:20

Duration    : 18m 10s

CPU hours   : 0.8

Succeeded   : 4

结果文件查看指南 (Windows Explorer 方式)

由于您将项目安置到 Linux 文件系统 (/home/sunhaipeng/projects/vEcoli)，Windows 提供了一个特殊的网络路径来访问这些文件。

步骤一：通过 \\\\wsl$ 访问 Linux 文件系统

打开您的 Windows 文件资源管理器（文件浏览器）。

将以下路径粘贴到地址栏中（并按回车键）：



\\\\wsl$

您将看到一个或多个 WSL 发行版文件夹（通常是 Ubuntu）。

步骤二：导航到结果文件

根据 README.md 的说明，测试安装的结果文件位于一个特定的子目录下。您需要沿着以下路径层级导航：

完整的路径结构为：



\\\\wsl$\\Ubuntu\\home\\sunhaipeng\\projects\\vEcoli\\out\\test\_installation\\analyses\\variant=0\\lineage\_seed=0\\generation=1\\agent\_id=0\\plots



双击进入 Ubuntu 文件夹。

双击进入 home 文件夹。

双击进入 sunhaipeng 文件夹。

依次进入 projects -> vEcoli -> out -> test\_installation -> analyses -> variant=0 -> lineage\_seed=0 -> generation=1 -> agent\_id=0 -> plots。



步骤三：打开结果文件

在 plots 文件夹中，您会找到用于验证安装成功的关键文件：



mass\_fraction\_summary.html

双击打开这个 HTML 文件，它将在您的默认浏览器中显示一个交互式仿真模拟的质量分数总结图表，总结了模拟中关键大分子组分的质量分数，这证明您的 Vivarium E. coli 模拟已成功运行。

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////





**在 VS Code 中运行 WSL 项目非常简单，因为 VS Code 具有出色的 Remote - WSL 扩展支持。**



以下是完整的设置和运行步骤：



1\. 安装 VS Code 扩展

确保您的 Windows 上的 VS Code 已经安装了必要的扩展，以便连接到您的 WSL 环境：



打开 VS Code (在 Windows 上)。



点击左侧的\*\*“扩展”\*\*图标（或按 Ctrl+Shift+X）。



搜索并安装 “ WSL” 扩展 (由 Microsoft 提供)。



2\. 在 VS Code 中打开 WSL 仓库

这一步会将 VS Code 客户端连接到您的 Ubuntu 环境，并确保所有终端和 Python 运行时都在 WSL 内部。

从 WSL 终端打开 VS Code：

回到您的 WSL (Ubuntu) 终端。

确保您在仓库根目录 (cd ~/projects/vEcoli)。

运行以下命令：

Bash



code .

这将启动 Windows 上的 VS Code，并提示您它正在连接到 WSL。

第一次运行时，VS Code 会在 WSL 环境中安装一个小型服务器。

验证连接：



VS Code 窗口左下角应该显示一个按钮，上面写着 WSL: Ubuntu。这确认您已成功连接到 WSL 环境。



3\. 配置 Python 虚拟环境

您需要告诉 VS Code 使用您之前用 uv 创建的虚拟环境 (.venv)。

按 Ctrl+Shift+P (或 F1) 打开命令面板。

输入 Python: Select Interpreter 并选择它。

(VS Code 应该会自动检测到您在仓库根目录下的 .venv 虚拟环境。选择路径类似于以下内容的解释器：



./.venv/bin/python

重要： 确保您选择的是项目目录下的 .venv 路径，而不是系统的 Python。)



4\. 创建运行配置 (launch.json)

为了简化每次运行的命令，我们将创建一个运行配置 (launch.json)，将复杂的 uv run python ... 命令自动化。



点击左侧的\*\*“RUN”\*\*图标（或按 Ctrl+Shift+D）。



点击 “创建 launch.json 文件” 链接。



选择 “Python”，然后选择 “Python File”。



VS Code 会在您的 .vscode 文件夹中创建一个 launch.json 文件。



用以下内容替换 launch.json 的内容。这个配置会使用您选择的 Python 解释器，并传递正确的参数：



JSON



{

&nbsp;   "version": "0.2.0",

&nbsp;   "configurations": \[

&nbsp;       {

&nbsp;           "name": "Launch vEcoli Test Simulation",

&nbsp;           "type": "python",

&nbsp;           "request": "launch",

&nbsp;           "program": "${workspaceFolder}/runscripts/workflow.py",

&nbsp;           "args": \[

&nbsp;               "--config",

&nbsp;               "configs/test\_installation.json"

&nbsp;           ],

&nbsp;           // important: this tells VS Code to use 'uv run'

&nbsp;           "console": "integratedTerminal",

&nbsp;           "justMyCode": true,

&nbsp;           "env": {

&nbsp;               "PATH": "${workspaceFolder}/.venv/bin:${env:PATH}"

&nbsp;           }

&nbsp;       }

&nbsp;   ]

}

5\. 在 VS Code 中运行模拟

回到\*\*“运行和调试”\*\*视图 (Ctrl+Shift+D)。



在顶部的下拉菜单中，选择 “Launch vEcoli Test Simulation”。



点击绿色的\*\*“RUN”\*\*按钮（或按 F5）。


VS Code 将在集成终端中启动脚本，并执行以下操作：
使用 .venv 中的 Python 解释器。
运行 runscripts/workflow.py 脚本。
传入 --config configs/test\_installation.json 参数。


这会像您在终端中一样启动模拟。运行成功后，您也可以在 VS Code 中直接浏览 out/test\_installation 文件夹查看结果。

> [Windows Subsystem for Linux](https://learn.microsoft.com/en-us/windows/wsl/install). Refer to the following pages for non-local setups:
> [Sherlock](https://covertlab.github.io/vEcoli/hpc.html#sherlock),
> [other HPC cluster](https://covertlab.github.io/vEcoli/hpc.html#other-clusters),
> [Google Cloud](https://covertlab.github.io/vEcoli/gcloud.html).

## Documentation
The documentation is located at [https://covertlab.github.io/vEcoli/](https://covertlab.github.io/vEcoli/)
and contains more information about the model architecture, output,
and workflow configuration.

If you encounter an issue not addressed by the docs, feel free to create a GitHub issue, and we will
get back to you as soon as we can.
