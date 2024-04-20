# FastAPI EasyEasyAuth #

## What is this? ##
This library quickly and easily creates authentication using JWT and Cookies. You can also use easyauth to identify an active user




### Using ###

First, we need to import the EasyAuth and Jwt classes from easyauth

    from fastapi_easyauth import EasyAuth, Jwt, ALGORITHM

After that, we create instances of classes
```python
jwt = Jwt(
    secret = "SECRET", # The secret key for generating tokens and decoding them. Keep the token secret
    algorithm = ALGORITHM.HS256 # The encryption algorithm
    model = MyModel # Your Pydantic model. When decoding the token, the result will be converted to this model
)

auth = EasyAuth(
    cookie_name = "user" # The Name Of The Cookie File
    jwt = jwt
    expires = exp.EXPIRES_30_DAYS # Cookie lifetime
    )
```

Great, everything is ready. Now we can create tokens and decode them. Also check if the user is active. 
```python
from fastapi import FastAPI, Request, Response, Depends
from fastapi_easyauth import EasyAuth, Jwt, ALGORITHM, exp

from pydantic import BaseModel

class User(BaseModel):
    name: str
    password: str

app = FastAPI()

jwt = Jwt(
    secret = "SECRET",
    algorithm = ALGORITHM.HS256
)

auth = EasyAuth(
    cookie_name = "user"
    jwt = jwt
    expires = exp.EXPIRES_30_DAYS
)

@app.post('/log')
def log(user: User, response: Response):
    token = jwt.create_token(user)
    auth.save_token_in_cookie(response, token)
    return {'status': 200}


@app.get('/active')
def active(user: User = Depends(auth.active_user))
    return user
```
----------

# What was added and changed in version 0.1.0 #

+ Now, using only an instance of the EasyAuth class, you can create a token and immediately save it in a cookie.

```python
@app.post('/login')
def login(user: User, response: Response):
    token = auth.create_token(subject = user, response = response)
    return token

@app.get('/getToken')
def get_token(request: Request):
    data = auth.decode_token(request)
    return data
```

+ The name of the Auth class has also been changed to EasyAuth.
+ All functions have been documented
+ Now, when initializing the Jwt class, you can specify the model. Each time you decode the token, the result will be returned in the form of the model that you specified

```python
class User(BaseModel):
    id: int
    username: str

jwt = Jwt(
    secret = "SECRET",
    model = User
)


def get_user(token: str) -> User:
    # if you did not specify the model during initialization, the result will be returned as a dictionary
    
    user = jwt.decode_token(token)
    return user


def get_user_in_model(token: str) -> User:
    # If you are not going to add models when initializing the Jwt class, then you can use the decode_taken_in_model function.
    # It accepts a token and a model in which
    
    user = jwt.decode_token_in_model(token, User)
    return user
```

----------
### There are also two simple functions in this library so far. The first is password hashing. The second is an error about an unauthorized user. ###

```python
def hash_password(password: str) -> str:
    """
    hash_password: hashes the password

    Args:
        password (str): User password

    Returns:
        str: hashed password
    """

    return hashlib.sha256(password.encode()).hexdigest()


def not_authorized() -> HTTPException:
    """
    not_authorized: a function that returns an error when an unauthorized user
    """

    return HTTPException(
        status_code=401,
        detail='Unauthorized'
    )
```









