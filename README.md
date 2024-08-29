# Oakestra Edge Testbed

## Device Setup
1. Install Raspberry Os
2. Install the LCD Driver:
```bash
sudo rm -rf LCD-show
git clone https://github.com/goodtft/LCD-show.git
chmod -R 755 LCD-show
cd LCD-show
sudo ./MHS35-show   
```
3. Install [docker](https://docs.docker.com/engine/install/debian/)

## Cluster Orchestrator Setup


1. Export the requried environmental variables
    ```bash
    ## Choose a unique name for your cluster
    export CLUSTER_NAME=Edge_Testbed
    ## Set the location to the Forschungszentrum Garching
    export CLUSTER_LOCATION=48.26,11.66
    ## Tell the NetManager where to find the system manager
    export SYSTEM_MANAGER_URL=<IP of device>
    ```
2. Clone the repository and move into it
    ```bash
    git clone https://github.com/oakestra/oakestra.git && cd oakestra
    ```
3. Run the root orchestrator with ipv6 overrides
    ```bash
    cd root_orchestrator
    sudo -E docker compose -f docker-compose.yml -f override-ipv6-enabled.yml up
    ```
4. Run the cluster orchestrator with ipv6 overrides
    ```bash
    cd cluster_orchestrator
    sudo -E docker compose -f docker-compose.yml -f override-ipv6-enabled.yml up
    ```

## RPi (Node) Setup
1. Install go
    ```bash
    wget https://go.dev/dl/go1.23.0.linux-arm64.tar.gz
    sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.23.0.linux-arm64.tar.gz
    echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.profile
    source ~/.profile
    ## Test if installation was successful
    go version
    ```
2. Download and unpack the NodeEngine

    ```bash
    wget -c https://github.com/oakestra/oakestra/releases/download/v0.4.202/NodeEngine_$(dpkg --print-architecture).tar.gz && tar -xzf NodeEngine_$(dpkg --print-architecture).tar.gz && chmod +x install.sh && mv NodeEngine NodeEngine_$(dpkg --print-architecture) && ./install.sh $(dpkg --print-architecture)
    ```

    and the NetManager

    ```bash
    git clone https://github.com/oakestra/oakestra-net.git
    cd oakestra-net/node-net-manager/build
    ./build.sh
    cp bin/arm-7-NetManager .
    cp ../config/* .
    chmod u+x ./install.sh
    ./install.sh
    ```

3. Edit `/etc/netmanager/netcfg.json` accordingly:

    ```json
    {
      "NodePublicAddress": "[<IPv6 ADDRESS OF THIS RPi>]",
      "NodePublicPort": "<PORT REACHABLE FROM OUTSIDE>",
      "ClusterUrl": "[<IPv6 ADDRESS OF THE CLUSTER ORCHESTRATOR>]",
      "ClusterMqttPort": "10003"
    }
    ```

4. Create `/etc/NodeEngine/.config` to provide the NodeEngine service with the cluster orchestrator IP

    ```bash
    sudo mkdir /etc/NodeEngine
    sudo sh -c 'echo "CLUSTER_IP=[<IPv6 ADDRESS OF THE CLUSTER ORCHESTRATOR>]" > /etc/NodeEngine/.config'
    ```

5. Copy unit files into `/etc/systemd/system/` and grant permissions
    ```bash
    # Clone unit files
    cd && git clone https://github.com/Mjaethers/oak-edge-testbed.git && cd oak-edge-testbed

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
