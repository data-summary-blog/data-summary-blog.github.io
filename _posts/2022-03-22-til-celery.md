---
layout: post
title: "TIL: Celery"
categories: til
author: "Yongchan Hong"
---

# Celery
Superset은 Celery를 쓴다. 그래서 대충 Celery의 용도는 알고 있지만 (분산을 위한 친구라는 것...) 정확히 어떤 친구 인지가 궁금해졌다. 이 친구 뭐하는 친구일까? 

`Celery`란 Python으로 작성된 `Asynchronous Task Queue`이다. 

> Task Queue란?  
> Thread나 머신에 작업을 분산시키 위한 메커니즘.  

![](https://media.vlpt.us/images/sms8377/post/258643d3-7422-40b8-901c-aa36a3ae0644/image.png)
Celery를 이해하기 위해선 `Broker`와 `Worker`를 이해해야한다. Celery내 `worker`는 `broker`를 통해 task들을 Client로부터 전달 받게되고, 작업을 처리하게 된다. 여기서 `Broker`는 보통 Messaging Queue인 Redis나 RabbitMQ를 자주 쓰곤 한다.

## Celery 사용하기
`pip install celery`로 먼저 설치하고, Broker를 정해주고 만들어둔다. 그 이후 다음과 같은 코드를 통해 구동할 수 있다.
```
from celery import Celery

app = Celery('tasks', broker='url_for_message_queue', backend='some_backend') # backend는 result 저장할곳

@app.task
def add(x, y):
    return x + y
```
다음과 같은 방식으로 Celery를 지정하고, task를 생성할 수 있다. 해당 Task를 Celery Task로서 호출하려면, delay를 사용하면 된다. 
```
from tasks import add
add.delay(1, 2)
```
또한 subtask와 chord (subtask들의 집합으로 묶여있는 작업들과 끝났을때 부를 Callback)을 사용하여 deadlock을 방지할 수 있다 ([Spoqa 예시 참조](https://spoqa.github.io/2012/05/29/distribute-task-with-celery.html  )). 

Celery는 간단하게 만들 수 있고, Highly Available하며 (connection이 실패하면 retry함), 빠르다!

## Celery Monitoring
[Flower](https://flower.readthedocs.io/en/latest/)라는 친구를 통해서 Monitoring도 할 수 있다! Worker와 브로커의 상태를 리얼 타임으로 보여준다. Task가 성공하는지 실패하는지도 보여준다.

## Superset에서 사용
Superset에서는 Async Queries를 실행하기 위해 Celery가 사용된다. 특히나 대형DB를 사용하는 팀으로서는 Async Queries는 필수적이다. 다음과 같이 Superset에서는 Celery관련 Config를 지정할 수 있다.
```
class CeleryConfig(object):
    BROKER_URL = 'redis://localhost:6379/0'
    CELERY_IMPORTS = (
        'superset.sql_lab',
        'superset.tasks',
    )
    CELERY_RESULT_BACKEND = 'redis://localhost:6379/0'
    CELERYD_LOG_LEVEL = 'DEBUG'
    CELERYD_PREFETCH_MULTIPLIER = 10
    CELERY_ACKS_LATE = True
    CELERY_ANNOTATIONS = {
        'sql_lab.get_sql_results': {
            'rate_limit': '100/s',
        },
        'email_reports.send': {
            'rate_limit': '1/s',
            'time_limit': 120,
            'soft_time_limit': 150,
            'ignore_result': True,
        },
    }
    CELERYBEAT_SCHEDULE = {
        'email_reports.schedule_hourly': {
            'task': 'email_reports.schedule_hourly',
            'schedule': crontab(minute=1, hour='*'),
        },
    }

CELERY_CONFIG = CeleryConfig
```
그리고 다음과 같이 구동한다.
```
celery --app=superset.tasks.celery_app:app worker
```

## 이모저모
`--max-tasks-per-child`를 통해 리소스 누수를 방지 할 수 있다. 이건 task를 일정 숫자 이상 execute하면 자동으로 worker를 restart하는 것이다. 까먹고 db connection을 닫지않거나 하는걸 강제로 리스타트 하니까 유용하다. 

## 결론
Python에서 비동기 분산처리로 필수적인 Celery... 다양한 configuration에 대해서 더 알아봐야겠다. Superset에서 최적화 시킬 부분이 어느정도 있을거같기도하다.

### Reference
https://spoqa.github.io/2012/05/29/distribute-task-with-celery.html  
https://velog.io/@sms8377/Celery-Python-Celery%EB%9E%80  
https://jonnung.dev/python/2018/12/22/celery-distributed-task-queue/  