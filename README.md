
# Install tools for geocomputation, research and fun on Ubuntu

Inspired by a post on [installing commonly needed GIS software on
Ubuntu](https://medium.com/@ramiroaznar/how-to-install-the-most-common-open-source-gis-applications-on-ubuntu-dbe9d612347b)
and having recently got a new computer (an Entroware Proteus, a bit
faster than the second hand [Lenovo
laptop](http://www.ebay.co.uk/sch/PC-Laptops-Netbooks/177/i.html?_from=R40&_nkw=lenovo&_dcat=177&rt=nc&_mPrRngCbx=1&_udlo=0&_udhi=200)
I was running when I created this repo!) with Ubuntu freshly installed,
I decided to make the process of installing GIS software on it as
reproducible as possible.

This is not intended to replace the excellent [OSGeo Live
distro](https://live.osgeo.org/en/index.html), which contains a ton of
amazing geo packages. Instead it’s for people who want lean, core GIS
functionality on their existing Ubuntu machine, with a focus on
geocomputation and data science.

Rather than providing automated scripts, it simply provides the bash
commands here in the README for you to copy, paste (or Ctrl+Shift+Enter
if in RStudio if you download this README, a great shortcut) Core
programs it shows how to install are:

- GitHub’s CLI tool for command-line collaboration and code sharing
- **R** with access to pre-compiled packages, a powerhouse for
  statistical computing
- R packages for working with spatial data
- **RStudio**, a space-aged editor for **R**
- **QGIS**, probably the most popular GUI-driven GIS in the world
- Recent versions of **GDAL** and **GEOS** C/C++ libraries
- Python packages for Geographic Data Science
- Rust, an upcoming systems language with impressive geo libs
- Docker, which gives ultimate power and flexibility

All this can be a pain to install manually. The commands below are
designed to make your life easier. Any comments/suggestions: welcome

## Prerequisites

- A working installation of Ubuntu 22.04, ‘Jammy Jellyfish’ on a
  computer you have access to

# Install key packages

Fire up a terminal, e.g. with `Ctl-Alt-T` after booting Ubuntu, then
enter the following.

## GitHub’s gh CLI

GitHub has developed a command line interface (CLI) tool for enabling
fast and intuitive interaction with the world’s premier code hosting
platform. It’s a good place to start because it’s small, self-contained,
and can be used to clone code repos like this one. Install it with the
following cryptic commands from the project’s [GitHub
page](https://github.com/cli/cli/blob/trunk/docs/install_linux.md):

``` bash
type -p curl >/dev/null || (sudo apt update && sudo apt install curl -y)
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg \
&& sudo chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg \
&& echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
&& sudo apt update \
&& sudo apt install gh -y
```

Test to see if it works as follows:

``` bash
# Log-in to GitHub from the command line
gh auth login
gh repo clone Robinlovelace/install-geocomp-ubuntu
cd install-geocomp-ubuntu 
less README.md
```

## R and RStudio

From
https://rtask.thinkr.fr/installation-of-r-4-2-on-ubuntu-22-04-lts-and-tips-for-spatial-packages/

``` bash
sudo apt update
sudo apt upgrade
sudo apt install -y --no-install-recommends software-properties-common dirmngr
# Add the keys
wget -qO- https://cloud.r-project.org/bin/linux/ubuntu/marutter_pubkey.asc | sudo tee -a /etc/apt/trusted.gpg.d/cran_ubuntu_key.asc
# add the R 4.0 repo from CRAN -- adjust 'focal' to 'groovy' or 'bionic' as needed
sudo add-apt-repository "deb https://cloud.r-project.org/bin/linux/ubuntu $(lsb_release -cs)-cran40/"

# You'll be asked to choose your geographical area
sudo apt install -y r-base r-base-core r-recommended r-base-dev
# Install PPA with pre-compiled packages
sudo add-apt-repository ppa:c2d4u.team/c2d4u4.0+
```

### Rapid install of R packages

``` bash
# My own selection among 5000...
sudo apt install -y r-cran-rgl r-cran-rjags r-cran-snow r-cran-ggplot2 r-cran-igraph r-cran-lme4 r-cran-rjava r-cran-devtools r-cran-roxygen2 r-cran-rjava
sudo apt install r-cran-tidyverse
sudo apt install r-cran-sf r-cran-tmap r-cran-osmextract r-cran-mapview
# System deps for cartography
sudo apt install -y libgdal-dev libproj-dev libgeos-dev libudunits2-dev libv8-dev libnode-dev libcairo2-dev libnetcdf-dev
sudo apt install -y libglu1-mesa-dev freeglut3-dev mesa-common-dev
# Extra packages for image manipulation
sudo apt install -y libmagick++-dev libjq-dev libv8-dev libprotobuf-dev protobuf-compiler libsodium-dev imagemagick libgit2-dev
# rspatial
sudo apt install r-cran-sf r-cran-terra r-cran-mapedit r-cran-tmap r-cran-mapdeck r-cran-shinyjs 
```

RStudio: https://www.entroware.com/store/proteus

``` bash
wget https://download1.rstudio.org/electron/jammy/amd64/rstudio-2023.03.0-386-amd64.deb
sudo dpkg -i rstudio*
rm -v rstudio*
```

## QGIS

From: https://github.com/geocompx/docker/blob/master/qgis/Dockerfile

``` bash
apt-get update && apt-get -y --with-new-pkgs upgrade
# how to install LTR, latest or nightly QGIS version, see:
# https://qgis.org/en/site/forusers/alldownloads.html#debian-ubuntu
sudo apt-get -y install gnupg software-properties-common python3-pip
sudo add-apt-repository -y ppa:ubuntugis/ubuntugis-unstable
sudo wget -O /etc/apt/keyrings/qgis-archive-keyring.gpg https://download.qgis.org/downloads/qgis-archive-keyring.gpg
# here, we install the QGIS-LTR, if you like the latest one, change in URIs from ubuntugis-ltr to ubuntugis
sudo -i
echo $'Types: deb deb-src \n\
URIs: https://qgis.org/ubuntugis-ltr\n\
Suites: jammy\n\
Architectures: amd64\n\
Components: main\n\
Signed-By: /etc/apt/keyrings/qgis-archive-keyring.gpg' >/etc/apt/sources.list.d/qgis.sources
# edit file manually
vim /etc/apt/sources.list.d/qgis.sources
apt-get update
# here using apt-get leads 10 packages kept back which in turn prevents the successful installation of qgis, ok, use --with-new-pkgs!
apt-get -y --with-new-pkgs upgrade && \
  apt-get -y autoremove && \
  apt-get -y install qgis qgis-plugin-grass saga

# for how to use the qgis-plugin-manager, see https://github.com/3liz/qgis-plugin-manager
pip3 install qgis-plugin-manager
# to enable the qgis-plugin-manager, add the corresponding path to PATH
# ENV PATH="/home/rstudio/.local/bin:$PATH"
echo "export PATH=\"/home/rstudio/.local/bin:\$PATH\"" >> /etc/profile
# from the next line onwards we have trouble with the rstudio server, therefore we switch to the rstudio user
mkdir -p /home/rstudio/.local/share/QGIS/QGIS3/profiles/default/python/plugins
ENV QGIS_PLUGINPATH=/home/rstudio/.local/share/QGIS/QGIS3/profiles/default/python/plugins
echo 'export QGIS_PLUGINPATH=/home/rstudio/.local/share/QGIS/QGIS3/profiles/default/python/plugins' >> /etc/profile
less /etc/profile
exit
qgis-plugin-manager init
qgis-plugin-manager update
# install SAGA next generation plugin
qgis-plugin-manager install 'Processing Saga NextGen Provider'
# You should see the following:
# QGIS version : 3.28.5
# Installation Processing Saga NextGen Provider latest
#         Ok processing_saga_nextgen.0.0.7.zip
# Installed with user 'robin'
# Please check file permissions and owner according to the user running QGIS Server.
# Tip : Do not forget to restart QGIS Server to reload plugins 😎

# the rest is nice to have but not really necessary
# qgis_process in a headless state
# ENV QT_QPA_PLATFORM=offscreen

# enable desired plugins
# qgis_process plugins enable processing_saga_nextgen
# qgis_process plugins enable grassprovider
# USER root
# install R package qgisprocess from Github (paused due to API limits)
Rscript -e "install.packages('remotes')" 
Rscript -e "remotes::install_github('r-spatial/qgisprocess')" 
```

## Rust

``` bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
# Make Rust available:
source "$HOME/.cargo/env"
# Try installing a crate:
cargo install geo
cargo install geozero
cargo install zonebuilder
```

## VS Code

VS Code is a versatile and future-proof IDE.

``` bash
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo install -o root -g root -m 644 packages.microsoft.gpg /etc/apt/trusted.gpg.d/
rm packages.microsoft.gpg
sudo add-apt-repository "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main"
sudo apt update
sudo apt-get install code
code --install-extension ms-vscode-remote.remote-containers
```

VS Code has a nice feature that enables you to develop inside a
‘devcontainer’. Devcontainers rely on Docker, which can be installed as
follows:

## Docker

Following instructions from
https://docs.docker.com/engine/install/ubuntu/, first install the
dependencies:

``` bash
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

Then follow these commands:

``` bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
# Use the following command to set up the repository:
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine

# Update the apt package index, and install the latest version of Docker Engine, containerd, and Docker Compose, or go to the next step to install a specific version:
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

### Post installation steps for Docker

The following steps enable you to run docker without `sudo`. As outlined
at https://docs.docker.com/engine/install/linux-postinstall/ this does
have security implications so it may be unwise to run these commands on
important production servers or critical infrastructure. For a personal
laptop that does not contain sensitive information the risks are low.

``` bash
sudo groupadd docker
# Add your user to the docker group:
sudo usermod -aG docker $USER

# Log out and log back in so that your group membership is re-evaluated.
# If you’re running Linux in a virtual machine, it may be necessary to restart the virtual machine for changes to take effect.
# You can also run the following command to activate the changes to groups:
newgrp docker

# Verify that you can run docker commands without sudo.
docker run hello-world

# If you initially ran Docker CLI commands using sudo before adding your user to the docker group, you may see the following error:
# WARNING: Error loading config file: /home/user/.docker/config.json -
# stat /home/user/.docker/config.json: permission denied

# This error indicates that the permission settings for the ~/.docker/ directory are incorrect, due to having used the sudo command earlier.

# To fix this problem, either remove the ~/.docker/ directory (it’s recreated automatically, but any custom settings are lost), or change its ownership and permissions using the following commands:

sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
sudo chmod g+rwx "$HOME/.docker" -R
```

### Running devcontainer with VS code

To check your `code` and `docker` installations worked you can try to
reproduce Geocomputation with Python:

``` bash
gh repo clone geocompx/geocompy
code geocompy
```

If you installed the `remote-containers` extension successfully, you
should see a button with “Reopen in Container” in the bottom right of VS
Code.

Click that button and you’ll see the devcontainer launch. If it works,
you can try reproducing the entirety of the book from the command line
with the following inside VS Code (you can launch the terminal by
pressing Ctrl+J\`):

``` bash
quarto preview
```

If you see something like this, congratulations, you can develop almost
anything in reproducible and easy-to-use devcontainers!

![](vscode-devcontainer.png)

## AppImage Launcher

Intuitive and time-saving management of AppImages.

``` bash
sudo add-apt-repository ppa:appimagelauncher-team/stable
sudo apt update
sudo apt install appimagelauncher
```

# Getting started with tools for geocomputation

Once you have the software installed, you can run it as follows:

- For QGIS, you can launch it with the usual launcher or from the
  command line

<!-- -->

    qgis # run QGIS

See the [QGIS manual](https://docs.qgis.org/latest/en/docs/index.html)

- For R, you can run R from the bash shell by entering `R`. For
  beginners, RStudio is recommended, which can be opened from the Ubuntu
  Dash launcher or from bash with:

<!-- -->

    rstudio

[This tutorial](https://github.com/Robinlovelace/Creating-maps-in-R)
provides a good starting point for working with spatial data in R.

- For Python, you need to activate the gds environment before use. Do
  this with from bash with:

<!-- -->

    source activate gds_test # activate the environment
    python
    >>>

from there you will be in the Python shell. Test if the geographic
packages work, e.g. with:

    import geopandas

You can also use the IPython notebook, as described in [Python’s
documentation](http://jupyter-notebook-beginner-guide.readthedocs.io/en/latest/execute.html).

For more, check out up-to-date tutorials, such as this one on
[GeoPandas](http://geopandas.org/).

# Other tools for boosting productivity and developer experience

## ddterm

ddterm is top down terminal:
https://github.com/ddterm/gnome-shell-extension-ddterm I have long been
a fan of [Guake](https://github.com/Guake/guake/), the popular terminal
emulator based roughly on Quake’s top down terminal. Unfortunately,
Guake does not have great support for Wayland, Ubuntu 22.04’s display
system, as discussed in this issue that is still open despite being a
priority that was opened in 2015:
https://github.com/Guake/guake/issues/492

Install it as follows:

``` bash
cd ~/Downloads/
wget https://github.com/ddterm/gnome-shell-extension-ddterm/releases/download/v43/ddterm@amezin.github.com.shell-extension.zip
gnome-extensions install -f ~/Downloads/ddterm@amezin.github.com.shell-extension.zip
sudo reboot # you may need to reboot the computer to use it
gnome-extensions enable ddterm@amezin.github.com 
```

# Alternative projects

- [ubuntu-post-install](https://github.com/snwh/ubuntu-post-install)
- [install-tl-ubuntu](https://github.com/scottkosty/install-tl-ubuntu)
