
# Memium API Documentation

This guide serves as the REST API documentation for integration with the Memium Bitcoin SV wallet platform.  Any questions or support 
required for integration of this API can be directed towards the developer at matt.dizak@gmail.com.

1. <a href="#requests">Requests</a>
2. <a href="#authentication">Authentication</a>
    1. <a href="#users_login">POST /users/login</a>
    2. <a href="#users_refresh_token">POST /users/refresh_token</a>
3. <a href="#users">User Management</a>
    1. <a href="#users_register">POST /users/register</a>
    2. <a href="#users_update">POST /users/update</a>
    3. <a href="#users_change_password">POST /users/change_password</a>
    4. <a href="#users_recover_wallet">POST /users/recover_wallet</a>
    5. <a href="#users_delete">DELETE /users/delete</a>
    6. <a href="#users_logout">GET /users/logout</a>
3. <a href="#coins">Wallet Management</a>
    1. <a href="#coins_get_rates">GET /coins/get_rates</a>
    2. <a href="#coins_generate_address">GET /coins/generate_address</a>
    3. <a href="#coins_get_balance">GET /coins/get_balance</a>
    4. <a href="#coins_get_transaction_history">GET /coins/get_transaction_history</a>
    5. <a href="#coins_get_transaction">GET /coins/get_transaction</a>
    6. <a href="#transaction_types">Transaction Types</a>
    7. <a href="#coins_send_funds">POST /coins/send_funds</a>
4. <a href="#memium">Memium Transactions</a>
    1. <a href="#memium_create_tx">POST /memium/create_tx</a>
5. <a href="#ws_notifications">Web Socket Notifications</a>
    1. <a href="#ws_message_exchange_rate">exchange_rate</a>
    2. <a href="#ws_message_deposit">deposit</a>


<a name="requests">,/a>
## Requests

All requests will be sent to https://wallet.memium.app/api/URI where URI is the specific method you're performing as defined below.  Proper use 
of HTTP request methods will be used, meaning:

- GET = Retrieve a resource.
- POST = Create or update a resource.
- DELETE = Delete a resource.
- HEAD = Retrieve META data on a resource.

All parameters required will be sent via standard form POST, and all responses will be JSON encoded.  Every response will contain a "status" 
variable which will have one of the following values:

- ok
- error -- Will also contain a "errmsg" variable giving a message regarding the error.
- expired - The access token has expired, and needs to be refreshed via the <a href="#users_refresh_token">/users/refresh_token</a> method.


<a name="authentication"></a>
## Authentication

You will generally need to initially execute the <a href="#users_login">/users/login</a> method to obtain an access token for the 
user account.  All requests must then include the following two HTTP headers.

HTTP Header | Required | Description
------------- |------------- |------------- 
API-Provider-Key | Yes | The API key you received as the software provider.
API-Access-Token | Yes | The access token as returned by the <a href="#users_login">/users/login</a> method.

Both HTTP headers are required for all requests, except the following methods which only require the "API_Provider_Key" HTTP header.

- <a href="#users_login">/users/login</a>
- <a href="#users_refresh_token">/users/refresh_token</a>
- <a href="#users_register">/users/register</a>


<a name="users_login"></a>
### POST /users/login

**URL:** POST https://wallet.memium.com/api/users/login

**Description:** Logs in a user, and returns an access and refresh token to be used for later requests.

**Parameters**

Variable | Required | Description
------------- |------------- |------------- 
username | Yes | The account username.
password | Yes | The account password in plain text.

**Return Values**

Variable | Description
------------- | -------------
access_token | The access token to use in all future requests to access this account.
refresh_token | The refresh token to use in order to obtain another access token once it expires.
expire_secs | The number of seconds the access_token is valid for.
expire_time | The time in seconds since the epoch when the access_token will expire.


<a name="users_refresh_token"></a>
### POST /users/refresh_token

**URL:** POST https://wallet.memium.app/api/users/refresh_token

**Description:** Refreshes an access token, and returns a new access token to be used within future requests.

**Parameters**

Variable | Required | Description
------------- |------------- |------------- 
refresh_token | Yes | The refresh token that was returned by the <a href="#users_login">/users/login</a> method.

**Return Values**

Variable | Description
------------- |------------- 
access_token | The new access token to use
expire_secs | The number of seconds the access_token is valid for.
expire_time | The time in seconds since the epoch when the access_token will expire.


<a name="users"></a>
## User Management

The below methods are used for overall account management including wallet creation, 
recovering a wallet, and others.

<a name="users_register"></a>
### POST /users/register

**URL:** POST /users/register

**Description:** Registers a new user, and creates a wallet.

**Parameters**

Variable | Key | Description
------------- |------------- |-------------
username | Yes | The desired account username.
email | Yes | The e-mail address of the user.
full_name | No | Optional full name of the user.
currency | No | The user's preferred fiat currency.  Defaults to USD.
password | Yes | Desired account password of the user.
wallet_password | Yes | The wallet password.  Must be minimum of 10 characters, and include at least one digit, one special character, and one uppercase character.  Must be hashed via MD5 before being sent.

**Return Values**

Variable | Description
------------- |------------- 
userid | The ID# of the newly created user.
mnemonic_phrase | The mnemonic phrase that must be displayed to the user, which they must be prompted to write down.  This will be used to recover their wallet in case of lost password.



<a name="users_update"></a>
### POST /users/update

**URL:** POST https://wallet.memium.app/api/users/update

**Description:** Updates a user's profile as necessary.

**Parameters**
Variable | Required | Description
------------- |------------- |------------- 
email | No | The e-mail address of the user.
full_name | No | Optional full name of the user.
currency | No | The user's preferred fiat currency.  Defaults to USD.



<a name="users_change_password"></a>
### POST /users/change_password

**URL:** POST https://wallet.memium.app/api/users/change_password

**Description:** Will change the user's account password.  Please note, this changes their user password only, and NOT their wallet password.

**Parameters**

Variable | Required | Description
------------- |------------- |------------- 
old_password | Yes | Their old / current account password.
new_password | Yes | The new account password to change to.


<a name="users_recover_wallet"></a>
### POST /users/recover_wallet

**URL:** POST https://wallet.memium.app/api/users/recover_wallet

**Description:** Used if the user has lost their wallet password.  This requires the mnemonic phrase they were given upon initial account creation.

**Parameters**

Variable | Key | Description
------------- |------------- |------------- 
mnemonic_phrase | Yes | The mnemonic phrase that was provided during their initial account creation.
wallet_password | Yes | TThe new wallet password they will be using.  Must be at least 10 characters in length, and contain at least one number and epcial character.

**Return Values**

Key | Description
------------- |------------- 
balance_coin | The current balance of their wallet in BSV
currency | Their preferred fia currency
balance_fiat | The total balance of their wallet in their preferred fiat currency.


<a name="users_delete"></a>
### DELETE /users/delete

**URL:** DELETE https://wallet.memium.app/api/users/delete

**Description:** Will permanently remove a user's account from the system, including all previous transaction information.  Please note, this operation can not be conducted if thare any unspent coins within the user's wallet.

**Parameters**

Variable | Required | Description
------------- |------------- |-------------
wallet_password | Yes | The user's wallet password, as defined during account creation.  Must be hashed via MD5 before being sent.


<a name="users_logout"></a>
### GET /users/logout

**URL:** GET https://wallet.mimium.app/api/users/logout

**Description:** Will logout the user, and delete their authenticated session from the system.  Recommended when a user closes the app / their phone.



<a name="coins"></a>
## Wallet Management

The below methods allow for various wallet management functions such as generate address, send funds, get balance, and others.

<a name="coins_get_rates"></a>
### GET /coins/get_rates

**URL:** GET https://wallet.memium.com/api/coins/get_rates

**Description:** Gets the current exchange rates of BSV into various fiat currencies.  Returns an array, the keys being the fiat currency symbol (eg. USD, EUR, GBP), and the 
value being the current exchange rate into BSV.


<a name="coins_generate_address">Generate Address</a>
### GET /coins/generate_address

**URL:** GET https://wallet.memium.com/api/coins/generate_address

**Description:** Generates a new payment address for the user.  Can take an optional query string parameter of "currency", but not needed as of this writing and will 
always default to BSV.  Functionality is simply there if future coins are ever supported.

**Return Values**

Variable | Description
------------- |------------- 
address | The newly generated payment address.


<a name="coins_get_balance"></a>
### GET /coins/get_balance

**Description:** Returns the current balance of the user's wallet.

**Return Values**

Variable | Description
------------- |------------- 
balance_coins | The balance in BSV
balance_usd | Balance of the wallet in USD.
currency | The user's chosen preferred fiat currency.
balance_fiat | Balance of the wallet in user's preferred fiat currency.


<a name="coins_get_transaction_history"></a>
### POST /coins/get_transaction_history

**URL:** GET https://wallet.memium.com/coins/get_transaction_history

**Description:** Retrives the user's transaction gistory, which can be filtered using the below parameters within the query string.

**Parameters**

Variable | Required | Description
------------- |------------- |-------------
type | No | Optional type of transaction.  Supported values are, *memium, coin, send, receive*
date_start | No | Optional start date to filter by, formatted in *YYYY-MM-DD*
date_end | No | Optional end date to filter by, formatted in *YYYY-MM-DD*
start | No | Used for pagination, and the result number to start at.  Defaults to 0.
num_results | No | The maximum number of results to return.  Defaults to 25.

**Return Values**

Returns an element named "transactions", which is an array of associative arrays.  Each element contains the following keys:

Key | Description
------------- |------------- 
id | The unique ID# of the transaction.
type | The type of transaction.  See <a href="#transaction_types">below table</a> for list of supported types.
amount_coin | The amount in BSV.
currency | The user's preferred currency.
amount_fiat | The amount of the transaction in the user's preferred currency.
status | The status of the transaction, can be either: *approved, declined, pending, refunded, cancelled*
date_added | The full date and time the transaction was dded, formatted in *YYYY-MM-DD HH:II:SS*
payment_address | Payment address if applicable (ie. tx is a send).
txid | Unique txid of the transaction on the blockchain, if applicable.
meme_id | Unique id# of the meme for the transaction, if applicable.
name | The name / one-lime description of the transaction.
note | Optional note included with the transaction, if available.


<a name="coins_get_transaction"></a>
### POST /coins/get_transaction

**URL:** POST /coins/get_transaction

**Description:** Gets details of the individual transaction.

**Parameters**

Variable | Required | Description
 |------------- |------------- 
transaction_id | Yes | The id# of the transaction to retrieve.

**Return Values**

Key | Description
------------- |------------- 
id | The unique ID# of the transaction.
type | The type of transaction.  See <a href="#transaction_types">below table</a> for list of supported types.
amount_coin | The amount in BSV.
currency | The user's preferred currency.
amount_fiat | The amount of the transaction in the user's preferred currency.
status | The status of the transaction, can be either: *approved, declined, pending, refunded, cancelled*
date_added | The full date and time the transaction was dded, formatted in *YYYY-MM-DD HH:II:SS*
payment_address | Payment address if applicable (ie. tx is a send).
txid | Unique txid of the transaction on the blockchain, if applicable.
meme_id | Unique id# of the meme for the transaction, if applicable.
name | The name / one-lime description of the transaction.
note | Optional note included with the transaction, if available.



<a name="transaction_types"></a>
### Transaction Types

The below table lists the supported transaction types, which are the "type" variable when retrieving transaction details using the above two methods.

Type | Description
------------- |------------- 
receive | Funds were sent directly to their wallet.
send | Funds were sent directly out of their wallet.
memium_clone_purchase | Purchase of a mime clone.
memium_clone_receive | Compensation received from someone cloning their mime.
memium_share_purchase | Purchase of a mim share.
memium_share_receive | Compensation received from someone sharing their meme.
memium_sticker_purchase | Purchase of usage of a sticker.
memium_sticker_receive | Compensation received for someone purchasing a sticker usage. 
memium_tip_send | Sent a meme tip.
memium_tip_receive | Received a tip on a mime.
memium_fee | Fees collected by administrator during a mime is cloned / shred, or a tip is sent.



<a name="coins_send_funds"></a>
### POST /coins/send_funds

**URL:** POST https://wallet.memium.com/api/coins/send_funds

**Description:** Sends funds from the user's wallet to another Memium user, a PayMail address, or a Bitcoin SV address.

**Parameters**

Variable | Required | Description
------------- |------------- |------------- 
currency | No | Optional currency to send.  Defaults to BSV (Bitcoin SV).  If fiat currency, current exchange rate will be applied.
amount | Yes | The amount to send.
recipient_types | Yes | The type of recipient.  Supported values are: *user, paymail, address*
recipient | Yes | Depends on the value of "recipient_type", and is either the Memium username, PayMail address, or Bitcoin SV payment address to send funds to.
wallet_password | Yes | The wallet password that was defined during user account creation.  Must be hashed via MD5 before being sent.
note | No | Optional user-defined note for the transaction.


** Return Values**

Key | Description
------------- |------------- 
amount_coin | The amount in BSV that was sent.
currency | The user's preferred currency.
amount_fiat | The amount sent in the user's defined fiat currency.
transaction_id | The unique id# of the internal transaction within Memium.
txid | The txid within the blockchain.
date_sent | The date and time the transaction was sent, formatted in *YYYY-MM-DD HH:II:SS*



<a name="memium"></a>
## Memium Transactions

Below lists all the Memium specific methods available.


<a name="memium_create_tx"></a>
### POST /memium/create_tx

**URL:** https://wallet.memium.com/api/memium/create_tx

**Description:** Used to create a Memium specific transaction when a user wishes to clone / share a meme, or provide a tip.

**Parameters**

Variable | Required | Description
------------- |------------- |------------- 
type | Yes | The type of transaction.  Supported types are: *clone, share, sticker, tip*
amount_type | Yes | The type of amount, either *coin* or *fiat*.  Defaults to *coin*.
amount | Yes | The amount in BSV or USD to charge for this transaction.
username | Yes | The username of the other Memium user this transaction is referring to (ie. if cloning a meme, the username of the user who owns the meme).  This can optionally be a comma delimited list of usernames, if more than one user is receiving compensation for the transaction.
meme_id | Yes The unique id# of the meme the transaction is for.
wallet_password | Yes | The wallet password as defined during account creation.  Must be hashed via MD5 before being sent.
note | No | Optional note.


**Return Values**

Key | Description
------------- |------------- 
amount_coin | The amount in BSV that was sent.
currency | The user's preferred currency.
amount_fiat | The amount sent in the user's defined fiat currency.
transaction_id | The unique id# of the internal transaction within Memium.
txid | The txid within the blockchain.
date_sent | The date and time the transaction was sent, formatted in *YYYY-MM-DD HH:II:SS*



<a name="ws_notifications"></a>
## Web Socket Notifications

There are two web socket notifications that will be sent to a pre-defined server for both, when the currency exchange rate changes, and 
when new deposits hit a user's account.  These are small JSON encoded messages, each of which will contain an 
"action" element of either "exchange_rate" or "deposit" as explained below.


<a name="ws_message_exchange_rate"></a>
### `exchange_rate`

This message is sent when the exchange rate is updated, simply contains an "action" element with a value of "exchange_rate", and a "rates" associative array where the 
keys are the three letter fiat currency abbreviation, and the values of the current exchange rate into BSV.  

**Example**

~~~
{
    "action": "exchange_rate", 
    "rates": [
        "USD": 272.51, 
        "AUD": 197.44, 
    "RUB": 85.13, 
    "CNY": 35.86
    ]
}
~~~


<a name="ws_message_deposit"></a>
### action = `deposit`

This message is sent every time a new deposit hits a user's account, and contains the following elements:

Element | Description
------------- |------------- 
action | Will always be "deposit".
username | The username who received the deposit.
amount_coin | The amount in BSV received.
currency | The user's preferred fiat currency.
amount_fiat | The amount in fiat that was received.
transaction_id | The id# of the internal Memium transaction
txid | The txid of the transaction on the blockchain.
address | The address that funds were sent to.
name | The name of the transaction (eg. You've received money as jsmith cloned your meme id# 4623462").


**Example**

~~~
{
    "action": "deposit", 
    "username": "mike", 
    "amount_coin": 0.053, 
    "currency": "USD", 
    "amount_fiat": 138.55
    "transaction_id": 88371, 
    "txid": "dad6998531e37b2273bc514c89d00ddab289e957f8f0ab45453931945ecb4abd", 
    "address": "n18WhospQpmfNnKZYQe8PM7sRhM3Xm2mk9", 
    "name": "You received 0.053 BSV from the address n18WhospQpmfNnKZYQe8PM7sRhM3Xm2mk9"
}
~~~




