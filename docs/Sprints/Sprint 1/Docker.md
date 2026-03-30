## Docker Installation

Remote server connection from Termius. The commands for the installation depend on the server's OS. Run this command into the Termius terminal to identify the server's OS.

``` bash
cat /etc/os-release
```

The server OS is Ubuntu, so the from now on the commands to install Docker are meant for Ubuntu only.

1. Update the package list

``` bash
sudo apt update
```

2. Install dependencies

``` bash
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
```

3. Add Docker's official GPG key

``` bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

4. Add Docker's repository

This step is important so the latest official Docker version is installed. 

``` bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

5. Update the package list again

``` bash
sudo apt update
```

6. Start Docker

``` bash
sudo systemctl start docker
```

7. Enable Docker

``` bash
sudo systemctl enable docker
```

8. Verify Docker Version

``` bash
docker --version
```

9. Verify Docker Installation

``` bash
docker run hello-world
```

If you see "Hello from Docker!" The installation is all done!