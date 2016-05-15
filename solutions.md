# task 6
```python

import requests

def get_url(action):
    return 'http://bootcamp-api.transferwise.com/{}/?token=06112afab9e912338169b5617e7c04a00193ff8f'.format(action)

r = requests.get(get_url('task/6'))
print(r.content)

r = requests.post(get_url('task/6/start'))
print(r.content)

#Get data
history = requests.get(get_url('payment/history')).json()
payments = requests.get(get_url('payment')).json()

for h in history:
    if h['fraud']:
        blocked_ip = h['ip'].split(".")[:-1]

offenders = []
for p in payments:
    if p['ip'].split(".")[:-1] == blocked_ip:
        offenders.append(p['id'])
        requests.put(get_url('payment/{}/fraud'.format(p['id'])))
    else:
        requests.delete(get_url('payment/{}/fraud'.format(p['id'])))

r = requests.post(get_url('task/finish'))
print(r.content)

```
