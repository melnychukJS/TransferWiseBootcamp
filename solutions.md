# task 1
```python
import requests
def get_url(action):
    return 'http://bootcamp-api.transferwise.com/{}/?token=06112afab9e912338169b5617e7c04a00193ff8f'.format(action)

r = requests.get(get_url(""))
print (r.text)
r = requests.get(get_url("task/1"))
print (r.text)
r = requests.post(get_url("task/1/start"))
print (r.text)
r = requests.post(get_url("name/Julia%20Melnychuk"))
r = requests.post(get_url('task/finish'))
print (r.text)
```

# task 2
```python
import requests
def get_url(action):
    return 'http://bootcamp-api.transferwise.com/{}/?token=06112afab9e912338169b5617e7c04a00193ff8f'.format(action)

r = requests.get(get_url("task/2"))
print(r.content)
r = requests.post(get_url("task/2/start"))
print(r.content)
a = [1] * 100
i = 0
removed = 0
while removed < 99:
    current = 0
    while current != removed:
        if a[i] == 1:
            current += 1
        i = (i + 1) % 100
    while a[i] != 1:
        i = (i + 1) % 100
    a[i] = 0
    removed += 1
print (a.index(1) + 1)
r = requests.post(get_url("survivor/{}".format(a.index(1) + 1)))
print(r.content)
r = requests.post(get_url("task/finish"))
print(r.content)
```

# task 3
```python
import requests

def get_url(action):
    return 'http://bootcamp-api.transferwise.com/{}/?token=06112afab9e912338169b5617e7c04a00193ff8f'.format(action)

requests.post(get_url('task/3/start'))

banks = requests.get(get_url('bank')).json()
accounts = requests.get(get_url('bankAccount')).json()
payments = requests.get(get_url('payment')).json()


def lookup(param_name, param_value, list_of_dicts):
    return filter(lambda x: x[param_name] == param_value, list_of_dicts)

get_bank = lambda x: next(lookup('id', x, banks))
get_account = lambda name, curr: next(lookup('currency', curr, lookup('accountName', name, accounts)))

for payment in payments:
    source_account = get_account("TransferWise Ltd", payment['sourceCurrency'])
    source_bank = get_bank(source_account['bankId'])
    requests.post(get_url('bank/{bank_name}/transfer/{source_account}/{target_bank}/{target_account}/{amount}'.format(
        bank_name=source_bank['name'],
        source_account=source_account['accountNumber'],
        target_bank=get_bank(payment['recipientBankId'])['name'],
        target_account=payment['iban'],
        amount=payment['amount']
    )))

r = requests.post(get_url('task/finish'))
print(r.content)
```

# task 4
```python
import requests, itertools
from math import floor

def get_url(action):
    return 'http://bootcamp-api.transferwise.com/{}/?token=06112afab9e912338169b5617e7c04a00193ff8f'.format(action)

r = requests.get(get_url('task/4'))
print(r.content)

r = requests.post(get_url('task/4/start'))
print(r.content)

currencies = requests.get(get_url('currency')).json()
companies = [c.strip() for c in requests.get(get_url('company')).json()["companies"].replace("[", "").replace("]", "").split(",")]

cur_list = list(itertools.combinations(currencies, 2))

for couple in cur_list:
    for amount in [100, 1000, 10000]:
        MMR = requests.get(get_url('rate/midMarket/{}/{}'.format(couple[0],couple[1]))).json()
        print('MMR ',MMR)
        OR = requests.get(get_url('quote/{}/{}/{}'.format(amount,couple[0],couple[1]))).json()
        print('OR', OR)
        for company in companies:
            print(company)
            OR_company = OR[company]
            print('OR_company ',OR_company)
            print('mmr-rate ',MMR['rate'])
            print('offer rate ',OR_company['offerRate'])
            fee = (MMR['rate'] - OR_company['offerRate'])/OR_company['offerRate'] * 100.0
            print('fee ',floor(fee))
            r = requests.post(get_url('hiddenFee/forCompany/{}/{}/{}/{}/{}'.format(
                company,
                amount,
                couple[0],
                couple[1],
                floor(fee)
            )))
            print(r.content)

r = requests.post(get_url('task/finish'))
print(r.content)
```

# task 5
```python
import requests

def get_url(action):
    return 'http://bootcamp-api.transferwise.com/{}/?token=06112afab9e912338169b5617e7c04a00193ff8f'.format(action)

r = requests.get(get_url('task/5')).json()

k = requests.post(get_url('task/5/start'))
print(k.content)

payments = requests.get(get_url('payment')).json()
people = r['peps']
peps = set()
for person in people:
    peps.add(person.split("-")[0].strip())

for payment in payments:
    if payment['recipientName'] in peps:
        print(requests.put(get_url('payment/{}/aml'.format(payment['id']))))
    else:
        print(requests.delete(get_url('payment/{}/aml'.format(payment['id']))))

r = requests.post(get_url('task/finish'))
print(r.content)
```

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
