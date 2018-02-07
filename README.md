Production server URL：

<https://dream.lucky.fun/>

Test server URL：

<https://test.dream.lucky.fun/>

Any applications should comply the following constraints:

1. Use HTTP POST method to send request（Content-Type: application/x-www-form-urlencoded）；
2. Server will response in JSON format（Content-Type: application/json;charset=utf-8）；
3. To simplify development, request url shouldn't contain any param,  all params should submit in form（application/x-www-form-urlencoded）；
4. Param's name in request suppose to be unique. 

All APIs require each request to be signed.  After Authentication, we will send you **app_id **and **app_secret**.  You should take great care to store your app_secret. If someone obtains your api_secret, they will able to do whatever he( or she) wants. 

For security purpose, you should inform us your production server IP and we will add it in our white list. Any request whose ip beyonds white list will be rejected.

Request Parameters 

| parameter name | type   | required | description                                                  |
| -------------- | ------ | -------- | ------------------------------------------------------------ |
| `app_id`       | string | Y        | unique string we dispatch to your application                |
| action         | string | Y        | api name                                                     |
| args           | json   | Y        | json string contains all params required                     |
| timestamp      | string | Y        | timestamp when sending a request, unit: **millisecond. **Note: any request larger than 30 seconds will be discard. |
| sign           | string | Y        | request signature, see details instruction in **\*API*** **Authentication** section |

sample：

```
app_id: "123456",
action: "recharge",
args: {"uid":"124d","amount":"44.938","out_trade_no":"5a4f9798971fc03796b7bacc"},
sign: "1f70971ea7a8814c1735676d25adeb7456007c6510e8bfa0ba8a21432a9cd14c",
timestamp: "1512114606000"
```

### API Authentication

All request parameters (**sign** excluded) should be sorted by parameter name ascending,  use = to bind the corresponding parameter value,   and then use & to bind each parameter string together.

*sample*:

```
action=recharge&args={"uid":"124d","amount":"44.938","out_trade_no":"5a4f9798971fc03796b7bacc"}&app_id=123456&timestamp=1512114606000
```

finally, use  **HmacSHA256 **algorithm whose key is app_secret  to create hash, and then use base64 to encode the hash, and the result is *sign*. For more details about **HmacSHA256** algorithm, please access this website: <http://blog.csdn.net/js_sky/article/details/49024959>

When our server receive a request, it will obtain the corresponding app_secret via *app_id* param, and calculate *sign* using **HmacSHA256** algorithm mentioned above. If the *sign* param in request is different from the server one, the request will be rejected.

Note: For some certain API,  our server will callback your application. In this case,  the callback request will also contains the *sign* param, and we strongly recommend you to verify the sign param using HmacSHA256 algorithm.

### Response parameters

| parameter name | type   | required | description                                                  |
| -------------- | ------ | -------- | ------------------------------------------------------------ |
| error_code     | int    | Y        | Positive value means errors occur.Error_code will always be 1 if error occurs. We won't give distinct error_code in v1 version, but error_msg will give out details |
| error_msg      | string | N        | If error_code is positive, this parameter will give out error details. Otherwise, it won't be returned. |
| data           | json   | Y        | Response for API, in json format.                            |

*sample*：

```
// error
{
  "error_code": 1,
  "error_msg": "something went wrong"
}
```

```
// correct
{
  "error_code": 0,
  "data": {
    "balance": "23.138"
  }
}
```

# API

# 1、withdraw LUCKY 

## api name

> withdraw

## request parameter

| parameter name | type   | required | description                               |
| -------------- | ------ | -------- | ----------------------------------------- |
| out_trade_no   | string | Y        | unique transaction id of your application |
| amount         | string | Y        | withdraw amount                           |
| address        | string | Y        | withdraw address                          |
| uid            | string | Y        | user id of your application               |
| fee            | string | Y        | withdraw fee                              |

## response data

none

# 2、query LUCKY withdraw fee  

## api name

> estimate_fee

## request parameter

none

## response data

| parameter name | type   | required | description                             |
| -------------- | ------ | -------- | --------------------------------------- |
| fee            | string | Y        | withdraw fee, 0.0002 LUCKY for example. |
| amount_min     | string | Y        | minimun withdraw amount                 |

# 3、query user balance

## api name

> query_balance

## request parameter

| parameter name | type   | required | description                      |
| -------------- | ------ | -------- | -------------------------------- |
| type           | int    | N        | 1: LUCKY; 2: BTC; 3: BCH; 4: ETH |
| uid            | string | Y        | user id of your application      |

## response data

| parameter name | type    | required | description                           |
| -------------- | ------- | -------- | ------------------------------------- |
| balances       | array[] | Y        | balance for different crypto currency |

detail for the *balances* array item :

| parameter name    | type   | required | description                      |
| ----------------- | ------ | -------- | -------------------------------- |
| type              | string | Y        | 1: LUCKY; 2: BTC; 3: BCH; 4: ETH |
| balance_total     | string | Y        | total balance                    |
| balance_available | string | Y        | available balance                |
| balance_frozen    | string | Y        | frozen balance                   |

# 4、bind callback URL

## api name

> bind_url

## request parameter

| parameter name | type   | required | description                                                  |
| -------------- | ------ | -------- | ------------------------------------------------------------ |
| notify_url     | string | Y        | Callback url. Our server will inform your application of the recharge or withdraw result asynchronously.  The callback request will also contains the *sign* param, and we strongly recommend you to verify the sign param using HmacSHA256 algorithm.*sample:*<https://test.dream.lucky.fun/> valid [https://test.dream.lucky.fun](https://test.dream.lucky.fun/)?a=b invalid, callback url shouldn't contain any params. |

## response data

none

# 5、create recharge address

## api name

> create_addr

## request parameter

| parameter name | type   | required | description                      |
| -------------- | ------ | -------- | -------------------------------- |
| type           | int    | Y        | 1: LUCKY; 2: BTC; 3: BCH; 4: ETH |
| uid            | string | Y        | user id of your application      |

## response data

| parameter name | type   | required | description      |
| -------------- | ------ | -------- | ---------------- |
| address        | string | Y        | recharge address |

## 6、callback asynchronously for recharge result

request parameter

| parameter name | type   | required | description                                 |
| -------------- | ------ | -------- | ------------------------------------------- |
| action         | string | Y        | API name. In this case, action is recharge  |
| args           | json   | Y        | json format and its details are shown below |
| timestamp      | string | Y        | unit: millisecond                           |
| sign           | string | Y        | see above                                   |

*args* detail

| parameter name | type   | required | description               |
| -------------- | ------ | -------- | ------------------------- |
| balance        | string | Y        | balance after withdraw    |
| amount         | string | Y        | recharge amount           |
| address        | string | Y        | recharge address          |
| status         | int    | Y        | 1: success; 2: failure    |
| txid           | string | Y        | blockchain transaction id |

Note: If your application process the callback request successfully, please just send back success, otherwise return fail 

Our server will callback periodically if received fail result, until receive success string, or timeout after 24 hours.

## 7、callback asynchronously for withdraw result

request parameter

| parameter name | type   | required | description                                 |
| -------------- | ------ | -------- | ------------------------------------------- |
| action         | string | Y        | API name. In this case, action is withdraw  |
| args           | json   | Y        | json format and its details are shown below |
| timestamp      | string | Y        | unit: millisecond                           |
| sign           | string | Y        | see above                                   |

*args* detail

| parameter name | type   | required | description                                 |
| -------------- | ------ | -------- | ------------------------------------------- |
| out_trade_no   | string | Y        | unique transaction id of your application   |
| status         | int    | Y        | 1: pending; 2: sent; 3: success; 4: failure |
| trade_no       | string | Y        | unique transaction id of our server         |

*sample:*

```
action: "withdraw",
args: {"out_trade_no":"5a4f9798971fc03796b7bad0","status":3,"trade_no":"5a4f9798971fc03796b7bacc"},
sign: "1f70971ea7a8814c1735676d25adeb7456007c6510e8bfa0ba8a21432a9cd14c",
timestamp: "1512114606000"
```

Note: If your application process the callback request successfully, please just send back success, otherwise return fail 

Our server will callback periodically if received fail result, until receive success string, or timeout after 24 hours.

8、changes about user balance account 

Any changes about user balance account should inform our server, except withdraw and crypto currency recharge from third party wallet.

## api name

> money_change

## request parameter

| parameter name | type   | required | description                               |
| -------------- | ------ | -------- | ----------------------------------------- |
| coin_type      | int    | Y        | 1: LUCKY; 2: BTC; 3: BCH; 4: ETH          |
| op_type        | int    | Y        | 1: add; 2; minus                          |
| out_trade_no   | string | Y        | unique transaction id of your application |
| uid            | string | Y        | user id of your application               |
| amount         | string | Y        | amount delta                              |

## response data

| parameter name | type   | required | description                               |
| -------------- | ------ | -------- | ----------------------------------------- |
| balance        | string | Y        | balance after changed                     |
| amount         | string | Y        | amount delta                              |
| out_trade_no   | string | Y        | unique transaction id of your application |
| trade_no       | string | Y        | unique transaction id of our server       |
| status         | int    | Y        | 1: success; 2: failure                    |

\* This document will be updated regularly, so stay tuned please.