# iot-edge-file-and-dir-watcher

Azure IoT Edge module which monitors file/directory creation and puts the filename/dirname as messages on the IoT Edge routing 

This module is forked from [here](https://github.com/iot-edge-foundation/iot-edge-filewatcher), customized it for only filename/dirname monitoring.

## Installation

### Clone this repository
Clone this repository by following command (or download as zip)
```
git clone https://github.com/yahanda/iot-edge-file-and-dir-watcher.git
```

### Create an Azure Container Registry
1. Follow the instructions to [Quickstart: Create a private container registry using the Azure portal](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-portal)
1. After your container registry is created, browse to it, and from the left pane select **Access keys** from the menu located under **Settings**.
1. Enable Admin user and copy the values for **Login server**, **Username**, and **Password** and save them somewhere convenient.

### Build and Push IoT Edge Solution
1. Open git cloned folder with VS Code.
1. Update the `.env` with the values you made a note from Azure Container Registry.
1. Open the Visual Studio Code integrated terminal by selecting **View > Terminal**.
1. Sign in to Docker with the Azure container registry credentials that you saved after creating the registry.
    ```
    docker login -u <ACR username> -p <ACR password> <ACR login server>
    ```
1. Open the command palette and search for **Azure IoT Edge: Set Default Target Platform for Edge Solution**, or select the shortcut icon in the side bar at the bottom of the window.
1. In the command palette, select the target architecture from the list of options.
1. **Build and Push IoT Edge Solution** by right clicking on `deployment.template.json` file.
1. Verify `deployment.json` file is generated in the `config` folder.

### Deploy the IoT Edge module
1. **Create Deployment for Single Device** by right clicking on `deployment.json` file under `config` folder.
1. Select a targeted IoT Hub and IoT Edge device.
1. The results of your deployment are printed in the VS Code output. Successful deployments are applied within a few minutes if the target device is running and connected to the internet.

## Container create options

This module makes use of Docker volumes to detect and access files on disk.

Use these container create options:

```
{
  "HostConfig": {
    "Binds": [
      "[Host folder]:/app/exchange"
    ]
  }
}
```

An example is:

```
{
  "HostConfig": {
    "Binds": [
      "/var/iot-edge-filewatcher/exchange:/app/exchange"
    ]
  }
}
``` 

The module will automatically create the 'exchange' folder. Please give the folder elevated rights to allow renaming files after processing. 

This can be done with:

```
sudo chmod 666 exchange
```

## Module twin desired properties

The module check the 'exchange' folder every few seconds to detect new files using a certain search pattern.

After processing, the file is renamed using a new extension which is appended to the orginal filename.

With desired properties, we can change the behavior:

* interval, the interval in milliseconds - Default 10000
* searchPattern, the pattern mapped on filenames  - Default '*.txt'
* renameExtension, the appended extension to prevent a file being read twice - Default '.old'
* retentionTime, the retention time in milliseconds until file has been fully written - Default 60000

## Messages

The messages outputted are JSON messages that contains the filename (full path) and filesize (byte) of the incoming file.

Detecting files

```
10/05/2021 14:18:04 - Seen 0 directory objects with pattern '*.txt'
10/05/2021 14:18:14 - Seen 0 directory objects with pattern '*.txt'
10/05/2021 14:18:24 - Seen 1 directory objects with pattern '*.txt'
10/05/2021 14:18:24 - File found: '/app/exchange/a.txt' - Size: 30 bytes.
10/05/2021 14:18:25 - Sleep for 60000 msec
10/05/2021 14:19:25 - Sending message: {"filename":"/app/exchange/a.txt","filesize":30}
10/05/2021 14:19:25 - Renamed '/app/exchange/a.txt' to '/app/exchange/a.txt.old'
10/05/2021 14:19:35 - Seen 0 directory objects with pattern '*.txt'
10/05/2021 14:19:45 - Seen 0 directory objects with pattern '*.txt'
```

Detecting directories

```
10/05/2021 14:20:25 - Seen 0 directory objects with pattern '*dir'
10/05/2021 14:20:35 - Seen 0 directory objects with pattern '*dir'
10/05/2021 14:20:45 - Seen 1 directory objects with pattern '*dir'
10/05/2021 14:20:45 - Directory found: '/app/exchange/test01dir'
10/05/2021 14:20:45 - Sleep for 60000 msec
10/05/2021 14:21:45 - Sending message: {"directoryname":"/app/exchange/test01dir"}
10/05/2021 14:21:45 - Renamed '/app/exchange/test01dir' to '/app/exchange/test01dir.old'
10/05/2021 14:21:55 - Seen 0 directory objects with pattern '*dir'
10/05/2021 14:22:05 - Seen 0 directory objects with pattern '*dir'
```

## File access

Keep in mind the module needs access to the folder. Please check the console log output of the module to see if right are elevated correctly. 

A simple check for access to the file is built in.

## File location

This module is tested with files available on the local file system only.

## Contributions

Sourcecode is available at [GitHub](https://github.com/iot-edge-foundation/iot-edge-filewatcher).

An example container is available on [Docker Hub](https://hub.docker.com/repository/docker/svelde/iot-edge-filewatcher).

Our project accepts GitHub pull requests :-) 
