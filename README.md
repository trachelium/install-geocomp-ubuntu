
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

- **R** with access to pre-compiled packages, a powerhouse for
  statistical computing
- **RStudio**, a space-aged editor for **R**
- **QGIS**, probably the most popular GUI-driven GIS in the world
- Recent versions of **GDAL** and **GEOS** C/C++ libraries
- R packages for working with spatial data
- A Python stack for Geographic Data Science
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

## Getting started

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

## Note

This is just a starter for 10. If you find it useful, please fork it and
push any useful updates back to this repo or make and issue if anything
goes wrong.

## Alternative projects

- [ubuntu-post-install](https://github.com/snwh/ubuntu-post-install)
- [install-tl-ubuntu](https://github.com/scottkosty/install-tl-ubuntu)
