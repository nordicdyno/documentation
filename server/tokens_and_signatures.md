# Tokens and signatures

Centrifugo uses HMAC with SHA-256 digest algorithm to create connection token and to sign
various data when communicating with Centrifugo via server or client API.

In this chapter we will see how to create tokens and signatures for different
actions. If you use Python all functions available in `Cent` library and you
don't need to implement them. This chapter can be useful for developers building
their own library (in other language for example) to communicate with Centrifugo.

Lets start with connection token.

### Client connection token

When client connects to Centrifuge from browser it should provide several connection
parameters: `user`, `timestamp`, `info` (optional) and `token`.

We discussed the meaning of parameters in other chapters - here we will see
how to generate a proper token for them.

What you should do to create client token is take hex digest of HMAC initialized
with Centrifugo server `secret` and feeding it `user`, `timestamp` and optionally `info`
(preserve this order). Optional `info` means that if you don't need it then just omit it
in token generation.

Let's look at Python code example for this:

```python
import six
import hmac
from hashlib import sha256

def generate_token(secret, user, timestamp, info=""):
    sign = hmac.new(six.b(secret), digestmod=sha256)
    sign.update(six.b(user))
    sign.update(six.b(timestamp))
    sign.update(six.b(info))
    return sign.hexdigest()
```

We initialize HMAC with secret key and ``sha256`` digest mode and then update
it with user ID, timestamp and info (order is important). Info is an optional arguments and if
no info provided empty string is used by default - so it does not affect resulting token
value.

**Note that our API clients already have functions to generate client token - you don't
have to implement this yourself until you want to implement it for another language for
which we don't have API client yet**


### Private channel subscription sign

When client wants to subscribe on private channel Centrifuge js client sends AJAX POST
request to your web application. This request contains `client` ID string and one or multiple
private `channels`. In response you should return an object where channels are keys.

For example you received request with channels `$one` and `$two`. Then you should return
JSON with something like this in response:

```python
{
    "$one": {
        "info": "{}",
        "sign": "ssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssss"
    },
    "$two": {
        "info": "{}",
        "sign": "ssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssss"
    }
}
```

Where `info` is additional information about connection for this channel and `sign` is
properly constructed HMAC based on client ID, channel name and info. Lets look at Python code
generating this sign:

```python
import six
import hmac
from hashlib import sha256

def generate_channel_sign(secret, client, channel, info=""):
    auth = hmac.new(six.b(secret), digestmod=sha256)
    auth.update(six.b(str(client)))
    auth.update(six.b(str(channel)))
    auth.update(six.b(info))
    return auth.hexdigest()
```

Not so different from generating client token. Note that as with client token - info is already JSON
encoded string.

**Note that our API clients already have functions to generate private channel sign - you don't
have to implement this yourself until you want to implement it for another language for
which we don't have API client yet**


### HTTP API request sign

When you use Centrifugo server API you should also provide sign in each request.

Again, Python code for this:

```python
import six
import hmac
from hashlib import sha256

def generate_api_sign(self, secret, encoded_data):
    sign = hmac.new(six.b(secret), digestmod=sha256)
    sign.update(six.b(encoded_data))
    return sign.hexdigest()
```

`encoded_data` is already a JSON string with your API commands. See available commands
in server API chapter.

**Note that our API clients already have functions to generate API sign - you don't
have to implement this yourself until you want to implement it for another language for
which we don't have API client yet**
