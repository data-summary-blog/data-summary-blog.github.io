---
layout: post
title: "Github Action의 편리함을 소개합니다"
categories: cicd
author: "Yongchan Hong"
---

# Github Action의 편리함을 소개합니다
필자는 Github Action의 덕을 많이 본 사람이다. Github Action 덕분에 쉽게 도커파일을 빌드하고 배포할 수 있었고, Lint도 쉽게 할 수 있었으며, S3에 Sync등도 할 수 있었다. 짧게 Github Action과 그 사용법에 대해 소개하고자 한다.

## Github Action이란
Github Action이란 Github에서 제공하는 CI/CD 툴이다. 깃헙에 Push, Pull Request 등의 이벤트가 생성되면 자동으로 Trigger되어 다양한 workflow를 자동화 할 수 있게 해준다. 이는 Github Action Runner로 가능한데, 이는 Workflow가 돌아가는 Virtual Machine이다. Enterprise에는 Github-hosted Runner를 지원하지않아 띄워야하는데, 우리 팀에서는 K8s에 [해당 구현체](https://github.com/actions-runner-controller/actions-runner-controller)를 통해 self-hosted runner를 만들었다 (해당 구현체는 Controller와 Runner로 나뉘어져 있다). Github Action의 Workflow를 정의하기 위해서는 repo의 `.github/workflows` 폴더 안에 yaml 파일로 저장해야한다. 

## Github Action 사용법
```
  name: Python application
	
  on:
    push:
      branches: [ master ]
    pull_request:
      branches: [ master ]
	
  jobs:
    build:
	
      runs-on: ubuntu-latest
	
      steps:
      - uses: actions/checkout@v2
      - name: Set up Python 3.7
        uses: actions/setup-python@v2
        with:
          python-version: 3.7
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install flake8 pytest
          if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
      - name: Lint with flake8
        run: |
          # stop the build if there are Python syntax errors or undefined names
          flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
          # exit-zero treats all errors as warnings. The GitHub editor is 127 chars wide
          flake8 . --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics
      - name: Test echo
        run: |
          echo "hi" 
      - name: Test python -h
        run: |
          python3 -h
```
[변성윤님의 자료](https://zzsza.github.io/development/2020/06/06/github-action/)를 빌려 쓰자면, 다음과 같이 name을 통해서 액션 명을, on을 통해서 어떤 액션에 trigger될지 (어떤 branch에서 trigger될지)를 정한다. workflow는 여러가지 job들로 나눠지게 되는데, 여러 job이 있는 경우 병렬 실행한다. 다음 코드에선 build라는 job하나 있고, build는 여러 step으로 나뉘게 된다. runs-on은 구동할 virtual machine을 정해주는데, 우리의 경우에는 self-hosted runner의 이름을 적었다. 그리고 steps의 시작은 actions/checkout@v2를 꼭 해줘야한다. 그래야 레포를 가져올 수 있다. 그 이후에 name으로 해당 step의 이름을 정해주고, uses를 통해 필요시 github action library에서 가져올 수 있다. run에서는 실제로 커맨드를 구동할 수 있다. 이렇게 깃헙 액션은 편하고 빠르게 사용할 수 있다.

### 개인 경험들..
> Action Secrets라는 걸 통해 Secret을 관리할 수 있지만 대부분 Vault를 통해 관리하였다.
> AWS Configure도 가능해 통신할 수 있다.
> aws-actions/amazon-ecr-login@v1을 통해 ECR Login이 가능하다
> COMMIT SHA를 받아와 Makefile에 넣어줘 docker build/push로 사용할 수 있다.

### Reference
https://ji5485.github.io/post/2021-06-06/build-ci-cd-pipeline-using-github-actions/
