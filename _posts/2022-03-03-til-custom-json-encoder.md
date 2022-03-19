---
layout: post
title: "TIL: Flask Custom Json Encoder"
categories: til
author: "Yongchan Hong"
---

# TIL: Flask Custom Json Encoder

Flask에서 JSON을 Encoding할때 (jsonify를 사용하여) date가 'yyyy-mm-dd'형태가 아닌 다른 형태로 만들어줘 곤란한 경험이 있었다. 그런 경우 아래와 같은 custom encoder를 제작해주면 된다.
```
class CustomJSONEncoder(JSONEncoder):
    def default(self, obj):
        try:
            if isinstance(obj, date):
                return obj.isoformat()
            iterable = iter(obj)
        except TypeError:
            pass
        else:
            return list(iterable)
        return JSONEncoder.default(self, obj)
```
이렇게 만든 후, app에다 적용시켜주면 편리하게 완료 가능하다.
```
app = Flask(__name__)
app.json_encoder = CustomJSONEncoder
```

### Reference 
https://stackoverflow.com/questions/43663552/keep-a-datetime-date-in-yyyy-mm-dd-format-when-using-flasks-jsonify