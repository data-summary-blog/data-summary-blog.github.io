---
layout: post
title: "TIL: Jupyter Hub Embedding하기"
categories: til
author: "Yongchan Hong"
---

# TIL: Jupyter Hub Embedding하기

JupyterHub를 iframe에 Embedding해야하는 이슈가 있었으나, iframe에 src를 넣고 구동하였을때 `Refuse to display xxx in a frame because an ancestor violates the following Content Security Policy directive: "frame-ancestors 'self'".` 이라는 오류 문을 확인할 수 있었다. 해결 방법은 `jupyterhub_config.py`와 `jupyter_notebook_config.py`라는 config에 다음과 같은 configuration을 넣어주어야 했다.
```
# jupyterhub_config.py
c.JupyterHub.tornado_settings = {'headers': {
    "Access-Control-Allow-Origin": "임베딩하는곳주소",
    "Content-Security-Policy": "frame-ancestors 'self' 임베딩하는곳주소"
  }}

# jupyter_notebook_config.py
c.NotebookApp.tornado_settings = {'headers': {
    "Access-Control-Allow-Origin": "임베딩하는곳주소",
    "Content-Security-Policy": "frame-ancestors 'self' 임베딩하는곳주소"
  }}
```
전체 JupyterHub말고 각 서버도 같은 config를 가지고 있어야하기에 `jupyter_notebook_config.py`에 넣어주는건 필수적이다. 이렇게 하면 다음과 같이 src에 넣어줄수 있다. 
```
<iframe src="JupyterHub주소/user/유저명?token=토큰값">
```
토큰 값은 각 유저가 받을 수 있다.