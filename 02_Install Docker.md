# Install Docker (Rocky Linux 9)

### [**Set up the repository**](https://docs.docker.com/engine/install/rhel/#set-up-the-repository)

Install the **`dnf-plugins-core`** package (which provides the commands to manage your DNF repositories) and set up the repository.

```bash
 sudo dnf -y install dnf-plugins-core
 sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
```

### [**Install Docker Engine**](https://docs.docker.com/engine/install/rhel/#install-docker-engine)

1. Install the Docker packages. Latest Specific version
    
    To install the latest version, run:
    
    ```bash
    sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```
    
2. Start Docker Engine.
    
    ```bash
    sudo systemctl enable --now docker
    ```
    
3. Verify
    
    ```bash
    docker version
    Client: Docker Engine - Community
     Version:           29.3.0
     API version:       1.54
     Go version:        go1.25.7
     Git commit:        5927d80
     Built:             Thu Mar  5 14:28:57 2026
     OS/Arch:           linux/amd64
     Context:           default
    permission denied while trying to connect to the docker API at unix:///var/run/docker.sock
    
    ```
