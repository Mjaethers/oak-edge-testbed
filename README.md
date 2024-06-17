# Oakestra Edge Testbed

## Cluster Orchestrator Setup
1. Export the requried environmental variables
    ```bash
    ## Choose a unique name for your cluster
    export CLUSTER_NAME=Edge_Testbed
    ## Set the location to the Forschungszentrum Garching
    export CLUSTER_LOCATION=48.26, 11.66
    ## Tell the NetManager where to find the system manager
    export SYSTEM_MANAGER_URL=<IP of device>
    ```
2. Clone the repository and move into it
    ```bash
    git clone https://github.com/oakestra/oakestra.git && cd oakestra
    ```
3. Run the cluster orchestrator
    ```bash
    sudo -E docker-compose -f run-a-cluster/1-DOC.yaml up
    ```

## RPi (Node) Setup
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
    # Copy and set permissions for node engine service
    sudo cp unit/NodeEngine.service /etc/systemd/system/
    sudo chmod 644 /etc/systemd/system/NodeEngine.service

    # Copy and set permissions for net manager service
    sudo cp unit/NetManager.service /etc/systemd/system/
    sudo chmod 644 /etc/systemd/system/NetManager.service
    ```
6. Enable the services
    ```bash
    sudo systemctl enable NodeEngine
    sudo systemctl enable NetManager
    ```
7. Reboot the system
