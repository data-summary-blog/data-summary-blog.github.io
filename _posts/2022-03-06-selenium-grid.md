---
layout: post
title: "What is Selenium Grid and How to Use It"
categories: selenium_grid
author: "Yongchan Hong"
---

# Selenium Grid

Selenium Grid는 무엇일까요? 일반 Selenium과는 어떻게 다를까요? 그리고, 언제 사용하는 것이 좋을까요? 오늘 포스트에 대해서는 이에 대해 알아봅시다.

## What is Selenium Grid? When should we use Selenium Grid?
Selenium Grid는 remote machine에서 Selenium script를 구동하기 위해 존재하는 것으로, client로부터 오는 커맨드를 각 remote instance에 라우팅해줍니다. 따라서, 같은 테스트를 여러 개의 remote machine에서 parallel하게 구동하기 매우 유용합니다.  
그렇다면 일반 Selenium Webdriver와는 어떻게 다를까요? 사실 Selenium Grid는 Selenium Webdriver를 원격으로 구동하는 stand-alone server일 뿐입니다. Selenium Webdriver는 구체적인 OS와 브라우저에서 구동이 되지만, Selenium Grid에선 다양한 OS와 브라우저 환경을 세팅하고 구동할 수 있습니다.  
Selenium Grid는 언제 사용하는게 좋고, 언제 사용하지 않는게 좋을까요? 먼저, 가장 일반적인 사용케이스는 일반적인 API에서 정보를 가져올 수 없고 브라우저를 통해서만 가져올 수 있는 경우입니다. 쿠버네티스 환경으로 구축 되어 있는 경우, Selenium Grid를 띄운후 Script를 구동하고 결과값을 받아올 수 있습니다. 또한, 여러 가지 브라우저/OS 환경에서 구동하고 실험하고 싶은 경우 사용합니다. 이 경우, 크롬에서는 괜찮은데 IE에선 문제가 생기는 경우 등을 확인해볼 수 있습니다. 또한, 간단한 반복 Test를 진행하고 싶은 경우 사용가능합니다. 너무 무거운 test의 경우, Selenium Grid를 사용하는 것을 비추천한다고 하지만, 가벼운 실험의 경우 parallel하게 구동할 수 있습니다.    
최근에 Selenium Grid 4의 정식 버전이 나왔습니다. 이전 Selenium Grid 3의 경우에는 Client와 Server(Browser Driver)사이에 JSON Wire Protocol Over HTTP라는게 있어서 사이에서 전달하는 역할을 하였다면, Selenium Grid 4에서는 Client와 Server끼리 직접 오갈 수 있게 변경이 되었습니다. 따라서 이전보다 더 Stable하게 브라우저에서 구동이 됩니다. 그리고 또 UI가 이쁘게 변경이 되었습니다.

![](https://miro.medium.com/max/700/1*3ZqqGJWULaSLX02rOvGQrw.png)

## Architecture of Selenium Grid

![](https://www.selenium.dev/images/documentation/grid/components.png)

Selenium Grid는 다음과 같이 Router, Distributor, Node, Session Map, Session Queue, Event Bus로 이루어져 있습니다. 양이 많지만, 생각보다 복잡하지 않으므로 빠르게 훑어 보겠습니다.  
- Router는 Client에서 온 request를 가장 처음으로 받고 전달하는 역할을 합니다. Session이 처음 들어오게 되면, Session Queue로 해당 Request를 전달하게 되고, 이미 존재하는 세션이라면 Session Map을 통해 해당 request를 구동중인 노드를 받아옵니다.
- Distributor는 구동할 세션과 그 정보를 받으면 적절한 node를 찾아서 배정해줍니다. 배정이 되면, 이후 session_id와 node를 매핑해서 Session Map에 저장합니다.
- Node는 test machine이라 보면 됩니다. 여기서 Test가 구동됩니다. Node는 Event Bus를 통해 Distributor에 자기 존재를 register합니다.
- Session Map은 이전에도 언급하듯 Session Id와 Node를 매핑합니다. 
- New Session Queue는 client단에서 온 request를 queue에 넣고 보관해주는 역할을 합니다. Router는 request를 여기에 채워넣고, Distributor는 정기적으로 와서 확인하고 Node가 비어있으면 request를 Queue에서 빼내고 새 session을 만들어줍니다. New Session Queue에서 parameter를 설정하여 request timeout와 request retry interval을 설정해줄 수 있습니다.
- Event Bus는 Sesion Map, Distributor, Nodes간의 메세지를 전달하는 역할을 해줍니다.  
보통 Selenium Grid를 설치하면 Hub와 Node로 나뉘게 되는데, Hub는 Node를 제외한 나머지 component들을 의미합니다.


## How I use Selenium Grid
저는 일단 Selenium Grid를 Kubernetes환경에 띄웠었습니다. [해당링크](https://github.com/pedrodotmc/docker-selenium/tree/selenium-grid-helm-chart/chart/selenium-grid)를 재구성하여 Selenium Grid를 Helm Chart화하여 띄웠었습니다. 이 경우, Hub와 Node가 뜨게 됩니다.  
제가 Selenium Grid를 사용했던 경우는 크게 2가지입니다.  
먼저, API로 가져올 수 없고 브라우저 상에서만 가져올 수 있는 데이터를 정기적으로 가져오는 일입니다. Airflow의 Script내 Selenium Grid와 연결하고, 구동하는 방법을 사용하였습니다. 다음은 Selenium Grid내 Chrome Node와 연결하는 방법입니다.
```
from selenium import webdriver

options = webdriver.ChromeOptions()
driver = webdriver.Remote(
    command_executor='http://{selenium_host}:4444/wd/hub',
    desired_capabilities=options.to_capabilities()
)
```
options에서는 no-sandbox나 window size등의 flag를 전달할 수 있습니다.  
Selenium Grid를 사용한 두번째 케이스는 바로 사용하던 앱을 테스트 하는 경우입니다. 저 같은 경우에는 Superset이라는 앱의 안정성을 테스트할 필요성이 있었습니다. Locust라는 툴을 통해서 기본적인 테스트를 하였지만, 특정 버튼을 클릭한다던가 하는 테스트가 필요했습니다. 따라서, Script를 Docker화 하고 이후 Kubernetes Job을 만들어 200개의 node에서 parallel하게 구동시키는 test를 진행하였습니다. 이를 통해 Superset의 Cache사용 등을 발전시키고 안정성을 크게 향상시킬 수 있었습니다.  

## So what?
Selenium Grid는 확실히 매력적인 툴이지만 가끔 안정성면에서는 의문이 가는 경우가 있습니다. 특히, 너무 많은 노드를 갖게 되면 더 불안정해집니다. 또한, Selenium Grid 4 Stable 버전이 나오기전에는 chrome node가 시간이 지나면 unregister되는 현상이 자주 있었습니다. 하지만 그럼에도 Selenium으로 밖에 하지 못하는 일이 있거나, 간단한 테스트가 필요한 경우에는 매우 추천합니다.

### Reference
https://blog.aerokube.com/selenium-grid-4-do-you-really-need-it-ab03366625b0
https://www.selenium.dev/documentation/grid/
