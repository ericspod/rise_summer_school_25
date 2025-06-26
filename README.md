# rise_summer_school_25
RISE Summer School 2025 Git and Docker

## Git Installation

This depends on your OS but all instructions can be found here: https://git-scm.com/downloads

A useful tool to have along side the command line interface is a GUI application such as [GitAhead](https://gitahead.github.io/gitahead.com/) or through [VSCode's inbuilt git integration](https://code.visualstudio.com/docs/sourcecontrol/overview).

## Docker Installation

Docker sometimes requires some specific steps to get installed. Docker can be [installed for Windows or macOS](https://www.docker.com/get-started/) but this will use a virtual machine to run containers. Installation instructions for Linux can be [found here](https://docs.docker.com/engine/install], and those for the Nvidia Container Toolkit can be [found here](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html). The toolkit is needed for accessing CUDA in a container.

The following is for installing in Ubuntu and is known to work for both Docker itself and the toolkit, but you should read the above documentation in case those are better instructions for your setup:

1. Install Nvidia drivers, try later versions than 550 if needed:

```sh
sudo apt-get install ubuntu-drivers-common
sudo ubuntu-drivers install nvidia:550
```

2. Remove conflicting packages:

```sh
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

3. Setup your apt repo and install:

```sh
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

4. Test with `sudo docker run --rm hello-world`

5. Install Nvidia Container Toolkit, this is needed to access CUDA from within a running container:

```sh
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
    
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
```

6. May need to restart docker daemon with `sudo systemctl restart docker`. Test with `sudo docker run --rm --gpus all ubuntu nvidia-smi`.

7. You can enable running docker as a non-root user by add that user to the `docker` group, then logging back in.

## Jupyter Installation

The slides are given here as Jupyter notebook slides. You can view the notebook using Jupyter or Jupyterlab, but can also view them in VSCode or PyCharm.
For VSCode you will need the Jupyter plugin at least, and others for slides if you want to view them that way.

To setup an environment for viewing the slides in Jupyter, this requires the RISE package:

```sh
pip install jupyterlab RISE jupyterlab-rise
```

The slides can be viewed as a continuous notebook or converted to HTML. One way of serving the slides on the command line:

```sh
jupyter nbconvert --to slides git_docker_slides.ipynb --post serve
```

While this is running in the background, the slides can be converted to pdf using `Chromium` (or `google-chrome` if installed):

```sh
chromium --headless --print-to-pdf=git_docker_slides.pdf http://127.0.0.1:8000/git_docker_slides.slides.html?print-pdf
```

## Useful Git Resources

* https://education.github.com/git-cheat-sheet-education.pdf
* https://about.gitlab.com/images/press/git-cheat-sheet.pdf
* https://learngitbranching.js.org/ (very useful exercise)
* https://git-scm.com/book/en/v2 (get the book)

## Useful Docker Resources

* Main docs: https://docs.docker.com/
* Docker command line cheatsheet: https://docs.docker.com/get-started/docker_cheatsheet.pdf
* Dockerfile reference: https://docs.docker.com/reference/dockerfile/
  