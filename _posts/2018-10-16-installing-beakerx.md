---
title: installing beakerx
date: 2018-10-16 16:00:00
categories: [dev, jupyter]
tags: [jupyter, beakerx, setting]
---

## ipywidgets, QGrid, BeakerX 설치
* [ipywidgets: notebook에서 slider 같이 user input 받을 수 있는 widget](https://ipywidgets.readthedocs.io/en/stable/user_guide.html)
* [QGrid: interactive한 grid view](https://github.com/quantopian/qgrid)
* [BeakerX: interactive한 grid view](http://beakerx.com/documentation)

```shell
$ conda config --add channels conda-forge
$ conda install ipywidgets
$ conda install qgrid

$ conda install nodejs
$ jupyter labextension install @jupyter-widgets/jupyterlab-manager
$ jupyter labextension install qgrid

$ conda install -c conda-forge beakerx ipywidgets
$ jupyter labextension install @jupyter-widgets/jupyterlab-manager
$ jupyter labextension install beakerx-jupyterlab
```
