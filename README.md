# Oakestra Edge Testbed

## RPi Setup
1. Install Raspberry Pi OS
2. Download and unpack both the NodeEngine

    ```bash
    wget -c https://github.com/oakestra/oakestra/releases/download/v0.4.202/NodeEngine_$(dpkg --print-architecture).tar.gz && tar -xzf NodeEngine_$(dpkg --print-architecture).tar.gz && chmod +x install.sh && mv NodeEngine NodeEngine_$(dpkg --print-architecture) && ./install.sh $(dpkg --print-architecture)
    ```

    and the NetManager

    ```bash
    wget -c https://github.com/oakestra/oakestra-net/releases/download/v0.4.202/NetManager_$(dpkg --print-architecture).tar.gz && tar -xzf NetManager_$(dpkg --print-architecture).tar.gz && chmod +x install.sh && ./install.sh $(dpkg --print-architecture)
    ```

3. Edit `/etc/netmanager/netcfg.json` accordingly:

    ```json
    {
      "NodePublicAddress": "<IP ADDRESS OF THIS RPi>",
      "NodePublicPort": "<PORT REACHABLE FROM OUTSIDE>",
      "ClusterUrl": "<IP ADDRESS OF THE CLUSTER ORCHESTRATOR>",
      "ClusterMqttPort": "10003"
    }
    ```

4. Create `/etc/NodeEngine/.config` to provide the NodeEngine service with the cluster orchestrator IP

    ```bash
    CLUSTER_IP=<IP ADDRESS OF THE CLUSTER ORCHESTRATOR>
    ```

5. Copy unit files into `/etc/systemd/system/` and grant permissions
    ```bash
    cp unit/NodeEngine.service /etc/systemd/system/
    sudo chmod 644 /etc/systemd/system/NodeEngine.service
    cp unit/NetManager.service /etc/systemd/system/
    sudo chmod 644 /etc/systemd/system/NetManager.service
    ```
6. Enable the services
    ```bash
    sudo systemctl enable NodeEngine
    sudo systemctl enable NetManager
    ```
7. Reboot the system
