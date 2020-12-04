# Use VSCode to build and debug Apollo projects

The Apollo project is loved and praised by many developers for its excellent system architecture, complete module functions, good open source ecology and standardized code style. However, the project uses the command line to compile and debug, and cannot use IDE development, which is neither intuitive nor conducive to improving efficiency. I am pursuing perfection a bit, and I have to find a way to make the Apollo project compile and debug in the IDE.

Visual Studio Code (hereafter referred to as VSCode) is Microsoft's first lightweight code editor that supports Linux. Its function is between an editor and an IDE, but it prefers an editor. The advantage is that it runs fast, occupies less memory, and the usage is similar to Visual Stuio. The disadvantage is that compared with IDEs such as Visual Studio and QT, the functions are not powerful enough. 

In my opinion, the most powerful C++ IDE in Windows is Visual Studio, and the most powerful C++ IDE in Linux is QT. The Apollo project currently only supports the Ubuntu system (partially supported by the Mac OS system), and the Visual Studio in the Windows system is naturally excluded; in addition, the Apollo project uses Bazel to compile, and QT currently only supports QMake and CMake projects, so it can only bear the pain. Under the premise that the above two IDEs cannot be used, VSCode is the second choice. I have written several configuration files that allow the use of VSCode to compile and debug Apollo projects. I will elaborate on them in detail below, hoping to bring some help to developers.

## 1. Use VSCode to compile the Apollo project
First download the Apollo source code from [GitHub website](https://github.com/ApolloAuto/apollo), you can use the git command to download, or you can download the compressed package directly from the web page. After the source code is downloaded, place it in a suitable directory. 

A prerequisite for using VSCode to compile Apollo projects is that Docker has been successfully installed on your machine. The previous version of Apollo provided an "install_docker.sh" script file. Because many developers reported that there may be errors, the Apollo project team has removed the file. Now to install Docker, you can only refer to the help document of [Docker official website](https://www.docker.com/).

### 1.1 Compilation method
Open "Visual Studio Code", execute the menu command "File->Open Folder", in the pop-up dialog box, select the "Apollo" project source folder, and click "OK", as shown in the following figure:
![1](images/vscode/open_directory.png)
![2](images/vscode/choose_apollo_root_directory.png)

After that, execute the menu command "Task -> Run Build Task" or directly press the shortcut key "Ctrl+Shift+B" (consistent with the shortcut keys of Visual Studio and QT) to build the project. If Docker has not been started before, it will be compiled To start Docker, you need to enter the super user password in the bottom terminal window, as shown in the following figure:
![3](images/vscode/vscode_call_for_password.png)

After the command is executed, if the message "**Terminal will be reused by the task, press any key to close it.**" appears in the terminal window at the bottom (as shown in the figure below), it means the build is successful. The whole process must keep the network unblocked, otherwise the dependency package cannot be downloaded.
![4](images/vscode/vscode_build_complete.png)

### 1.2 Configuration file analysis
I configured a total of four common tasks in the `.vscode/tasks.json` file: `build the apollo project` (build the Apollo project), `run all unit tests for the apollo project` (run all units of the Apollo project) Test), `code style check for the apollo project` (code style check for the Apollo project), `clean the apollo project` (clean up the Apollo project). The first task is the default generated task, you can directly press the shortcut key "Ctr+Shift+B" to call, other tasks can be executed by executing the menu command: Task -> Run Task (R)..., in the pop-up window, select the corresponding Option is fine, as shown in the figure below:
![5](images/vscode/run_normal_build_task.png)
![6](images/vscode/selete_build_task.png)

The following is the specific configuration content, please refer to the comments inside to adjust the compilation task to meet your build requirements:

```
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "build the apollo project",
            "type": "shell",
            // The compilation task can be adjusted according to the options provided by "apollo.sh", for example: build_gpu
            "command": "bash apollo_docker.sh build",
            "group": {
                "kind": "build",
                "isDefault": true // default building task invoked by "Ctrl+Shift+B"
            },
            // Format error message
            "problemMatcher": {
                "owner": "cc",
                "fileLocation": [
                    "relative",
                    "${workspaceFolder}"
                ],
                "pattern": {
                    "regexp": "^(.*):(\\d+):(\\d+):\\s+(warning|error):\\s+(.*)$",
                    "file": 1,
                    "line": 2,
                    "column": 3,
                    "severity": 4,
                    "message": 5
                }
            }
        },
        {
            "label": "run all unit tests for the apollo project",
            "type": "shell",
            "command": "bash apollo_docker.sh test",
            "problemMatcher": {
                "owner": "cc",
                "fileLocation": [
                    "relative",
                    "${workspaceFolder}"
                ],
                "pattern": {
                    "regexp": "^(.*):(\\d+):(\\d+):\\s+(warning|error):\\s+(.*)$",
                    "file": 1,
                    "line": 2,
                    "column": 3,
                    "severity": 4,
                    "message": 5
                }
            }
        },
        {
            "label": "code style check for the apollo project",
            "type": "shell",
            "command": "bash apollo_docker.sh lint",
            "problemMatcher": {
                "owner": "cc",
                "fileLocation": [
                    "relative",
                    "${workspaceFolder}"
                ],
                "pattern": {
                    "regexp": "^(.*):(\\d+):(\\d+):\\s+(warning|error):\\s+(.*)$",
                    "file": 1,
                    "line": 2,
                    "column": 3,
                    "severity": 4,
                    "message": 5
                }
            }
        },
        {
            "label": "clean the apollo project",
            "type": "shell",
            "command": "bash apollo_docker.sh clean",
            "problemMatcher": {
                "owner": "cc",
                "fileLocation": [
                    "relative",
                    "${workspaceFolder}"
                ],
                "pattern": {
                    "regexp": "^(.*):(\\d+):(\\d+):\\s+(warning|error):\\s+(.*)$",
                    "file": 1,
                    "line": 2,
                    "column": 3,
                    "severity": 4,
                    "message": 5
                }
            }
        }
    ]
}
```

### 1.3 Possible problems and solutions
#### 1.3.1 "ERROR: query interrupted" error encountered during compilation
This is caused by the inconsistent internal cache of bazel.

Solution:

Press any key to exit the compilation process, and execute the following command in the VSCode command terminal window (if it is not open, press the shortcut key "Ctrl + `" to open) to enter the Docker environment:
``` bash
bash docker/scripts/dev_into.sh
```

Enter the following command in the Docker environment to execute bazel's cache cleanup task (**must keep the network unblocked** to successfully download the dependent package, otherwise the command will not work even if executed 10,000 times):
``` bash
bazel query //...
```

Finally, enter the exit command to exit the Docker environment and press the shortcut key "Ctrl+Shift+B" to execute the build task again.

#### 1.3.2 Staying on the "Building: no action running" interface for a long time during compilation
This is due to the existence of multiple different versions of Docker in the current system or the inconsistent internal cache of bazel.

Solution: 

Press the shortcut key "Ctrl+C" to terminate the current build process. In the command terminal window of VSCode (if not open, press the shortcut key "Ctrl + `" to open), use any of the following methods to stop the currently running Docker:
``` bash
# Method 1: Stop all current Apollo project Docker
docker stop $(docker ps -a | grep apollo | awk'{print $1}')
# Method 2: Stop all current Docker
docker stop $(docker ps -aq)
```

Execute the menu command of VSCode: Task -> Run Task (R)..., in the pop-up window, select "Clean the apollo project" (clean the Apollo project). After cleaning up, press the shortcut key "Ctrl+Shift+B" to rebuild the Apollo project.

#### 1.3.3 An error similar to "Another command (pid=2466) is running. Waiting for it to complete..." appears during compilation
This is caused by compiling in other command line terminals or by pressing the "Ctrl+C" key during the previous compiling to forcibly terminate but part of the compiling process remains.

Solution: 

Press the shortcut key "Ctrl+C" to terminate the current build process. In the command terminal window of VSCode (if not open, press the shortcut key "Ctrl + `" to open), use the following command to terminate the remaining compilation process:
``` bash
# 1. Enter Docker
bash docker/scripts/dev_into.sh
# 2. Kill the remaining compilation process in Docker
pkill bazel-real
# 3. Check whether the bazel-real process remains in Docker, if there is, press "q" to exit, and perform step 2 again.
# You can also use "ps aux | grep bazel-real" to view
top
# 4. Exit Docker
exit
```
Press the shortcut key "Ctrl+Shift+B" to execute the build task again.

## Second, use VSCode to debug Apollo projects locally
The Apollo project runs in Docker, and cannot be directly used in GDB debugging on the host (the so-called host is the host running Docker, because the Docker service is like hosting on the host, so it is called). It must be created in Docker with the help of GDBServer. Debug the service process, and then use GDB in the host to connect to the debug service process in Docker to complete. The following describes the specific operation method:
### 2.1 Prerequisites
#### 2.1.1 Compile Apollo project requires debugging information
When compiling Apollo projects, you need to use options with debugging information such as `build` or `build_gpu`, but not optimization options such as `build_opt` or `build_opt_gpu`.
#### 2.2.2 GDBServer is installed inside Docker
After entering Docker, you can use the following command to view:

``` bash
gdbserver --version
```

If prompted with information similar to the following:

```
GNU gdbserver (Ubuntu 7.7.1-0ubuntu5~14.04.3) 7.7.1
Copyright (C) 2014 Free Software Foundation, Inc.
gdbserver is free software, covered by the GNU General Public License.
This gdbserver was configured as "x86_64-linux-gnu"
```

It means that GDBServer has been installed inside Docker.
If the following information is prompted:

```
bash: gdbserver: command not found
```

It means that GDBServer is not installed inside Docker, you can use the following command to install:

``` bash
sudo apt-get install gdbserver
```

### 2.2 Docker internal operations
#### 2.2.1 Start Dreamview background service program
Enter Docker, start Dreamview, the command is as follows:

``` bash
cd your_apollo_project_root_dir
# If Docker is not started, start first, otherwise ignore this step
bash docker/scripts/dev_start.sh -C
# Enter Docker
bash docker/scripts/dev_into.sh
# Start Dreamview background service
bash scripts/bootstrap.sh
```

#### 2.2.2 Start the module to be debugged
Start the module to be debugged, either by using the command line or by using the Dreamview interface. I definitely like to use the Dreamview interface to operate. The following is an example of debugging the "planning" module.

Open the Chrome browser, enter the URL: [http://localhost:8888/](http://localhost:8888/), open the Dreamview interface, and open the "SimControl" option, as shown in the following figure:
![7](images/vscode/enable_simcontrol.png)
Click the "Module Controler" tab in the toolbar on the left, and select the "Routing" and "Planning" options, as shown below:
![8](images/vscode/start_routing_and_planning.png)

Click the "Default Routing" tab in the left toolbar, select "Route: Reverse Early Change Lane" or any of these options, and send a "Routing Request" request to generate a global navigation path, as shown in the following figure:
![9](images/vscode/check_route_reverse_early_change_lane.png)
#### 2.2.3 View "Planning" process ID

Use the following command to view the "Planning" process ID:

``` bash
ps aux | grep mainboard | grep planning
```

The result is similar to the figure below, you can see that the "Planning" process ID is 4147.
![11](images/vscode/planning_id_ps.png)

#### 2.2.4 Use GDBServer to debug the "Planning" process
Next, we need to perform our key operations. Use GDBServer to debug the "Planning" process. The command is as follows:

``` bash
sudo gdbserver :1111 --attach 4147
```

In the above command, **":1111" means to open the debugging service process with port "1111", and "4147" means the "Planning" process ID found in `step 2.2.3`**. If the result is as shown in the figure below, it means the operation was successful.
![12](images/vscode/gdbserver_attach_debug.png)

Reopen a command terminal, after entering Docker, use the following command to see that the "gdbserver" process is running normally:

``` bash
ps aux | grep gdbserver
```
![13](images/vscode/view_gdbserver_process.png)

#### 2.2.5 Start GDBServer with script file
I wrote two script files: `scripts/start_gdb_server.sh` and `docker/scripts/dev_start_gdb_server.sh`. The former is used to start GDBServer inside Docker, and the latter is used to start GDBServer directly on the host (outside Docker).
Assuming that the `planning` module is debugged, the port number is `1111`, and the usage of `scripts/start_gdb_server.sh` is:

``` bash
# Enter Docker
bash docker/scripts/dev_into.sh
# Start gdbserver
bash scripts/start_gdb_server.sh planning 1111
```

Assuming that the `planning` module is debugged and the port number is `1111`, the usage of `docker/scripts/dev_start_gdb_server.sh` is:

``` bash
# Start gdbserver directly in the host (outside Docker)
bash docker/scripts/dev_start_gdb_server.sh planning 1111
```

In **Section 2.3**, I will continue to configure the VSCode file so that you can directly press the shortcut key "F5" in VSCode to start debugging.
The content of `start_gdb_server.sh` is as follows:

``` bash
#!/usr/bin/env bash

function print_usage() {
  RED='\033[0;31m'
  BLUE='\033[0;34m'
  BOLD='\033[1m'
  NONE='\033[0m'

  echo -e "\n${RED}Usage${NONE}:
  .${BOLD}/start_gdb_server.sh${NONE} MODULE_NAME PORT_NUMBER"

  echo -e "${RED}MODULE_NAME${NONE}:
  ${BLUE}planning${NONE}: debug the planning module. 
  ${BLUE}control${NONE}: debug the control module.
  ${BLUE}routing${NONE}: debug the routing module.
  ..., and so on."

  echo -e "${RED}PORT_NUMBER${NONE}: 
  ${NONE}a port number, such as '1111'."
}

if [ $# -lt 2 ];then
    print_usage
    exit 1
fi

DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
source "${DIR}/apollo_base.sh"

MODULE_NAME=$1
PORT_NUM=$2
shift 2

# If there is a gdbserver process running, stop it first. 
GDBSERVER_NUMS=$(pgrep -c -x "gdbserver")
if [ ${GDBSERVER_NUMS} -ne 0 ]; then
  sudo pkill -SIGKILL -f "gdbserver"
fi

echo ${MODULE_NAME}
# Because the "grep ${MODULE_NAME}" always generates a process with the name of 
# "${MODULE_NAME}", I added another grep to remove grep itself from the output.
PROCESS_ID=$(ps -ef | grep "mainboard" | grep "${MODULE_NAME}" | grep -v "grep" | awk '{print $2}')
echo ${PROCESS_ID}

# If the moudle is not started, start it first. 
if [ -z ${PROCESS_ID} ]; then       
  #echo "The '${MODULE_NAME}' module is not started, please start it in the dreamview first. "  
  #exit 1 

  # run function from apollo_base.sh
  # run command_name module_name
  run ${MODULE_NAME} "$@"

  PROCESS_ID=$(ps -ef | grep "mainboard" | grep "${MODULE_NAME}" | grep -v "grep" | awk '{print $2}')
  echo ${PROCESS_ID}
fi 

sudo gdbserver :${PORT_NUM} --attach ${PROCESS_ID}
```
`dev_start_gdb_server.sh`的内容如下：
``` bash
#!/usr/bin/env bash

function check_docker_open() {
  docker ps --format "{{.Names}}" | grep apollo_dev 1>/dev/null 2>&1
  if [ $? != 0 ]; then       
    echo "The docker is not started, please start it first. "  
    exit 1  
  fi
}

function print_usage() {
  RED='\033[0;31m'
  BLUE='\033[0;34m'
  BOLD='\033[1m'
  NONE='\033[0m'

  echo -e "\n${RED}Usage${NONE}:
  .${BOLD}/dev_debug_server.sh${NONE} MODULE_NAME PORT_NUMBER"

  echo -e "${RED}MODULE_NAME${NONE}:
  ${BLUE}planning${NONE}: debug the planning module. 
  ${BLUE}control${NONE}: debug the control module.
  ${BLUE}routing${NONE}: debug the routing module.
  ..., and so on."

  echo -e "${RED}PORT_NUMBER${NONE}: 
  ${NONE}a port number, such as '1111'."
}

if [ $# -lt 2 ];then
    print_usage
    exit 1
fi

check_docker_open

DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
cd "${DIR}/../.."
# pwd

xhost +local:root 1>/dev/null 2>&1
#echo $@
docker exec \
    -u $USER \
    -it apollo_dev \
    /bin/bash scripts/start_gdb_server.sh $@
xhost -local:root 1>/dev/null 2>&1
```

### 2.3 Operations inside VSCode on the host
Use VSCode on the host to open the Apollo project (must be the version you just built), open the file that needs to be debugged, set a breakpoint at the specified location, and press the "F5" key to start debugging. Note: **Since VSCode is written in a scripting language, the startup process will be slow. If the internet speed is not fast enough, it may even be a one-minute wait**. The debugging method is similar to that of Visual Studio, so I won't repeat it here. As shown below:
![14](images/vscode/vscode_debug_interface.png)

### 2.4 Configuration file analysis
I configured the `.vscode/launch.json` file so that I can connect to the debugging service process in Docker in VSCode. In addition, in order to start GDBServer directly in VSCode, I added a pre-debugging start task in the `.vscode/launch.json` file: `"preLaunchTask": "start gdbserver"`, which corresponds to `.vscode/ A task in the tasks.json` file to start GDBServer, because GDBServer will always block the command line window after it is started, and it cannot be started in the background by adding `&` after the command. I can only configure it as a VSCode Run tasks in the background.

The configuration content of the `.vscode/launch.json` file is as follows:

```
{
    "version": "0.2.0",
    "configurations": [
    {
        "name": "C++ Launch",
        "type": "cppdbg",
        "request": "launch",
        "program": "${workspaceRoot}/bazel-bin/cyber/mainboard",
        // You can change "localhost:1111" to another "IP:port" name, but it 
        // should be same as those in gdbserver of the docker container.  
        "miDebuggerServerAddress": "localhost:1111",
        // You can set the name of the module to be debugged in the 
        // ".vscode/tasks.json" file, for example "planning".
        // Tips: search the label "start gdbserver" in ".vscode/tasks.json".
        // The port number should be consistent with this file.        
        "preLaunchTask": "start gdbserver",
        "args": [],
        "stopAtEntry": false,
        "cwd": "${workspaceRoot}",
        "environment": [],
        "externalConsole": true,
        "linux": {
            "MIMode": "gdb"
        },
        "osx": {
            "MIMode": "gdb"
        },
        "windows": {
            "MIMode": "gdb"
        }
    }
    ]
}
```

The task configuration used to start GDBServer in the `.vscode/tasks.json` file is as follows:

```
        {
            "label": "start gdbserver",
            "type": "shell",
            // you can change the "planning" module name to another one and 
            // change the "1111" to another port number. The port number should 
            // be same as that in the "launch.json" file. 
            "command": "bash docker/scripts/dev_start_gdb_server.sh planning 1111",            
            "isBackground": true,
            "problemMatcher": {
                "owner": "custom",
                "pattern": {
                    "regexp": "__________"
                },
                "background": { 
                    "activeOnStart": true,
                    // Don't change the following two lines, otherwise the 
                    // gdbserver can't run in the background.
                    "beginsPattern": "^Listening on port$",
                    "endsPattern": "^$"
                }
            }                           
        }
```

### 2.5 Possible problems and solutions
During the debugging process, the following problems may be encountered:
One is that the process to be debugged inside Docker crashes and cannot be debugged in VSCode (as shown in the figure below). The solution is: restart the debug process inside Docker;
![15](images/vscode/failed_to_connect_gdbserver.png)

The second is that the network connection is not smooth and cannot be debugged in VSCode. The solution is to ensure that the network is smooth and disable the proxy tool;

The third is that after closing debugging inside VSCode, the debugging service process inside Docker will be closed at the same time. There may be situations where the debugging service mileage cannot be started directly in VSCode. The solution is: restart the debugging service process inside Docker, and then in VSCode Press the "F5" key to start debugging.

## Third, use VSCode to remotely debug Apollo projects
During the development process, we also need to remotely debug the Apollo project on the industrial computer in the car, that is, connect to the industrial computer in the car through the SSH service on the debugging computer, start the relevant processes in the industrial computer, and then perform remote debugging on the debugging computer. The following takes the debugging of the `planning` module as an example for specific explanation:

### 3.1 View the IP address of the industrial computer in the car
On the industrial computer in the car, use the following command to view the local IP:

``` bash
ifconfig
```

As shown in the figure below, the white selected part: `192.168.3.137` is the **LAN IP address** of the industrial computer in the car.
![16](images/vscode/view_ipc_ip.png)

### 3.2 Open Dreamview in the browser of the debugging computer and start the module to be debugged
Assuming that the IP address of the industrial computer in the car is: `192.168.3.137`, open the Chrome or Firefox browser and enter the following URL: [http://192.168.3.137:8888/](http://192.168.3.137:8888/ ), according to the method in section **2.2.2**, start the `planning` to be debugged and its dependent `routing` module.
![17](images/vscode/remote_show_dreamview.png)

### 3.3 Use the SSH command to remotely log in to the industrial computer in the car and start the gdbserver service of the industrial computer
Assuming that the user name of the industrial computer in the car is `davidhopper`, and the IP address of the LAN is: `192.168.3.137`, use the following command to remotely log in to the industrial computer in the car:

``` bash
ssh davidhopper@192.168.3.137
```

![18](images/vscode/ssh_remote_login_192_168_3_137.png)
After successfully entering the industrial computer, assuming that the `planning` module needs to be debugged, the port number is `1111`, use the following command to start the gdbserver service of the industrial computer in the car:

``` bash
# Switch to the root directory of the Apollo project on the industrial computer
cd ~/code/apollo
# Start gdbserver service outside Docker
bash docker/scripts/dev_start_gdb_server.sh planning 1111
```

As shown in the figure below, if you see a prompt similar to `Listening on port 1111`, it means that the `gdbserver` service started successfully.
![19](images/vscode/remote_start_gdbserver.png)

### 3.4 Use VSCode on the debugging computer to remotely debug the `planning` module on the industrial computer
Use VSCode to open the Apollo project on the debugging computer. Note that the project version should be consistent with the version on the industrial computer, otherwise a lot of inconsistent information will be output during debugging. First, the pre-debugging loading task in the configuration file `.vscode/launch.json`: `"preLaunchTask": "start gdbserver",` comment out, and then modify the remote debugging service address to: `"miDebuggerServerAddress": "192.168 .3.137:1111",`, as shown in the figure below:
![20](images/vscode/config_launch_json_for_remote_debug.png)

Set a breakpoint at the desired location and press the `F5` key to start debugging. Because it is a remote connection, **starting waiting time will be longer, even more than 1 minute**. The figure below is the remote debugging interface. **Note: Every time you stop debugging in VSCode, when you need to start debugging again, you must perform the operation in section 3.3 again on the command line terminal to restart the gdbserver service process in the industrial computer. **During local debugging, because I configured a `preLaunchTask` task, this step can be omitted.
![21](images/vscode/remote_debug_planning_module.png)
