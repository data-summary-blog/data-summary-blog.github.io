---
layout: post
title: "Querybook - Introduction (2)"
categories: querybook
author: "Yongchan Hong"
---

# Querybook Functions

저번까진 Querybook의 구조도에 알아봤다면, 이번에는 본격적으로 Querybook을 어떻게 사용할 수 있는지 알아보도록 합시다. 

## Querybook DataDocs
![](https://www.querybook.org/img/key_features/collab.gif)  
Querybook DataDocs는 Querybook의 가장 꽃같은 부분입니다. DataDoc은 구글닥스와 같이 공유 및 동시 수정이 가능한 Notebook느낌의 Document입니다. 아키텍쳐에서 설명하였듯이, Socket.IO를 이용하여 변경사항이 스트리밍됩니다.  
Querybook DataDocs에는 총 3가지 종류의 input이 가능합니다.  
첫번째는 **Text**로, 다른 document와 비슷하게 title, bold, slanted 등 다양한 형식을 즐길 수 있습니다.  
다음은 **Query**입니다. 필요한 Query를 입력하고 옆에 run (>) 버튼을 누르게 되면, 해당 엔진을 이용하여 구동을 하게 됩니다. 결과는 바로 나오게 되면, Export를 누르면 csv나 tsv형태로 다운 받을 수 있고, share execution을 통해 해당 주소를 공유 할 수도 있습니다.  
Query상의 매력적인 부분은 바로 Template입니다.  
![](https://www.querybook.org/img/key_features/templating.gif)  
다음과 같은 형식의 jinja template으로 variable이름을 넣어주고, 우측 하단의 template에서 template이름과 값을 넣어주게 되면 이 변수의 값이 자동으로 들어가 구동할 수 있습니다.
마지막으로 **Chart**입니다.  
![](https://www.querybook.org/img/key_features/visualization.gif)  
Graph는 이전 Query문을 바탕으로 자동으로 Graph를 생성해줍니다.  
Config Chart를 누르게 된다면, Graph를 변경할 수 있으며, 현재 지원되는 Graph형식은 Line, Stacked Area, Bar, Pie, Scatter, Doughnut, Bubble등이 있습니다.  

이러한 Document들은 다 작성한 이후 Private이나 Public으로 접근권한을 설정할 수 있습니다.  
Public으로 설정된 경우, environment내 모든 유저들에게 공개가 됩니다.  
Private으로 추가된 경우, 현재는 손으로 특정 유저들을 username을 통해 검색을 하고 추가해야합니다.    
권한은 read only, edit, owner 3가지 종류가 있습니다.


## Querybook Table
Querybook Table 탭은 Catalog의 기능을 지원하는 창입니다.  
![](https://www.querybook.org/assets/images/board3-b1b02316306a34b0e41ff2015c63c11a.gif)  
특정 테이블을 누르고 View Table을 보게 되면, 다음과 같은 화면이 나옵니다.  
Overview, Columns, Row Sample (간단한 where/order by문이 있는 쿼리구동), lineage 등을 확인할 수 있습니다.  

## Querybook Adhoc  
쉽고 빠르게 Query를 날릴 수 있는 곳입니다. 남들과 공유가 되지 않습니다.  

## Querybook Snippet  
조인 등을 한 쿼리들을 특정 이름으로 저장할 수 있는 곳입니다.  

## Querybook Admin  

![](https://www.querybook.org/img/plugin_features/engine.png)
Querybook의 Admin 권한이 있는 경우 다음과 같이 Admin App에 접속할 수 있습니다.   
접속방법으론 /admin을 뒤에 붙이거나, 웹사이트 왼쪽 아래에서 이름을 누르고 Admin Tools를 누르면 접속이 가능합니다.  
Querybook Admin 앱에선 많은 것을 관리할 수 있습니다.  
특히, **Query Engine**, **Metastore**, **Environment**를 관리할 수 있습니다.  
**Query Engine**에서는 Query를 구동하는 Engine입니다. Query Engine에서는 Metastore를 지정해줌으로써 필요한 Metastore를 불러올 수 있고, Status Checker를 통해 현재 그 엔진이 이상이 없이 잘 연동 되는지 확인이 가능합니다. Query Engine에서는 한개의 Metastore과 연결이 가능합니다.  
**Metastore**는 실질적인 Table과 Schema에 관한 정보를 불러오는 곳입니다. ACL Control을 통해 필요한 Table들만 불러올 수 있고, Scheduler가 있어 Metastore를 주기적으로 업데이트 시킬 수 있습니다. 이러한 업데이트 된 Metastore는 **Querybook Table**에 나타납니다.    
**Environment**를 통해서 Querybook을 자유롭게 사람들에게 공유할 수 있습니다. Environment이름을 정하고, public/hidden/shareable등의 설정으로 access를 관리할 수 있습니다. 또한, Access Control을 통하여 어떤 사람들이 접근할 수 있는지 explicit하게 정의가 가능합니다.  
Admin에서는 그 외에도 다른 Admin을 정할 수 있는 User Role, API에 관한 설정을 볼 수 있는 API Access Token, Announcment를 만들 수 있는 곳 까지 있습니다.

## Others  
![](https://www.querybook.org/img/key_features/scheduling.png)  
Querybook에서는 이외에도 스케줄링을 통해 이메일이나 Slack을 통해서 export하거나 하는 등의 기능을 제공합니다.

### Reference
https://www.querybook.org/