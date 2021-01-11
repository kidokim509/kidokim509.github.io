---
title: installing jupyter lab on the ubuntu 16.04
date: 2018-10-16 20:10:00 +0900
categories: [dev, jupyter]
tags: [jupyter, setting]
---

## Install Anaconda
[https://www.anaconda.com/download/#linux](https://www.anaconda.com/download/#linux)
```shell
$ wget https://repo.anaconda.com/archive/Anaconda3-5.3.0-Linux-x86_64.sh
$ ./Anaconda3-5.3.0-Linux-x86_64.sh
$ source ~/.bashrc
```

## Create a virtual environment
```shell
$ conda create -n jupyter
$ vim ~/.bashrc
conda activate jupyter
$ source ~/.bashrc
```

## Install python packages you need
```shell
$ conda install pandas
$ conda install scikit-learn
$ ...
```

## Jupyter Configuration
[https://jupyter-notebook.readthedocs.io/en/stable/public_server.html#running-a-public-notebook-server](https://jupyter-notebook.readthedocs.io/en/stable/public_server.html#running-a-public-notebook-server)
```shell
$ jupyter notebook --generate-config
c.NotebookApp.ip = '0.0.0.0'
```

## Install Jupyter Lab
[https://jupyterlab.readthedocs.io/en/stable/getting_started/installation.html](https://jupyterlab.readthedocs.io/en/stable/getting_started/installation.html)
[https://jupyterlab.readthedocs.io/en/stable/getting_started/starting.html](https://jupyterlab.readthedocs.io/en/stable/getting_started/starting.html)
[https://bradford.la/2017/jupyter-lab-service](https://bradford.la/2017/jupyter-lab-service)
```shell
$ conda install -c conda-forge jupyterlab
$ jupyter lab
```
