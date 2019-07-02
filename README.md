# python-monobank

![python-monobank](https://raw.githubusercontent.com/vitalik/python-monobank/master/docs/logo.png)

Python client for Monobank API (https://api.monobank.ua/docs/)

## Installation

```
pip install monobank
```


# Usage

## Personal api

1) Request your token at https://api.monobank.ua/

2) Use that token to initialize client:

```
  import monobank
  token = 'xxxxxxxxxxxxxxx'

  mono = monobank.Client(token)
  user_info = mono.personal_clientinfo()
  print(user_info)
```

### Methods

Get currencies

```
>>> mono.bank_currency()
[
 {'currencyCodeA': 840,
  'currencyCodeB': 980,
  'date': 1561686005,
  'rateBuy': 25.911,
  'rateSell': 26.2357},
 {'currencyCodeA': 978,
  'currencyCodeB': 980,
  'date': 1561686005,
  'rateBuy': 29.111,
  'rateSell': 29.7513},
  ...
```

Get client info

```
>>> mono.personal_clientinfo()
{
  'name': 'Dmitriy Dubilet'
  'accounts': [
    {
      'id': 'accidxxxxx'
      'balance': 100000000,
      'cashbackType': 'UAH',
      'creditLimit': 100000000,
      'currencyCode': 980,
      }
  ],
}

```


Get statements
```
>>> mono.personal_statement('accidxxxxx', date(2019,1,1), date(2019,1,30))
[
  {
    'id': 'iZDPhf8v32Qass',
    'amount': -127603,
    'balance': 99872397,
    'cashbackAmount': 2552,
    'commissionRate': 0,
    'currencyCode': 978,
    'description': 'Smartass club',
    'hold': True,
    'mcc': 5411,
    'operationAmount': 4289,
    'time': 1561658263
  },
  ...
]
```




## Corporatre API

Documentation is here - https://api.monobank.ua/docs/corporate.html

Corporate API have the same methods as Public API, but it does not have rate limitation, and it is a recomended way if you are handling data for commercial use (or just storing lot of personal data).

### Getting access

#### 1) Generate private key

```
openssl ecparam -genkey -name secp256k1 -rand /dev/urandom -out priv.key
```

This will output file **priv.key** 

**Warning**: do not share it with anyone, do not store it in public git repositories

#### 2) Generate public key

```
openssl ec -in priv.key  -pubout -out pub.key
```

This will output file **pub.key** 

#### 3) Request API access 
Send an email to api@monobank.ua - describe your project, and attach **pub.key** (!!! NOT priv.key !!!)


### Requesting permission from monobank user

Once your app got approved by Monobank team you can start using corporate API:


#### 1) Create monobank user access request

```
private_key = '/path/to/your/priv.key'
request = monobank.access_request('p', private_key)
```
If all fine you should recive the following:
```
print(request)
{'tokenRequestId': 'abcdefg_Wg', 'acceptUrl': 'https://mbnk.app/auth/abcdefg_Wg'}
```

You should save tokenRequestId to database, and then give user the link acceptUrl


#### 2) Check if user accepted

You can check if user accepted access request like this:


```
request_token = 'abcdefg_Wg'  # the token from access_request result
private_key = '/path/to/your/priv.key'

mono = monobank.CorporateClient(request_token, private_key)


mono.check()  # returns True if user accepted, False if not

```


#### 3) Use methods

Once user accepts your access-request, you can start using all the methods same ways as Public API

```
mono.personal_statement(....)
```

## Handling Errors

If you use Personal API you may encounter "Too Many Requests" error. To properly catch in and retry use monobank.TooManyRequests exception

```
try:
    mono.personal_statement(....)
except monobank.TooManyRequest:
    time.sleep(1)
    # try again:
    mono.personal_statement(....)
```

You can use ratelimiter library (like https://pypi.org/project/ratelimiter/ ) to download all transactions
