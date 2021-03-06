# [`app.py`](https://github.com/ineshbose/boyd_bot_messenger/blob/master/app.py)

This script is the Flask app. It is the only script to have access to the keys and enables webhook.


## Packages Used

* [flask](https://github.com/pallets/flask)



## Setup

The app is initialized using the following lines. All environment variables are loaded onto the scripts.

```python
# Flask App Properties
app = Flask(__name__)
## Link views.py to app
app.register_blueprint(pages)
## Enable logging despite debug = False
app.logger.setLevel(logging.DEBUG)
## Disable logging for POST / GET status (Not Recommended)
logging.getLogger("werkzeug").setLevel(logging.ERROR)

# App URL as variable to easily change it
app_url = os.environ["APP_URL"]

# Flask Secret Key to enable FlaskForm
app.config["SECRET_KEY"] = os.environ["FLASK_KEY"]

# Webhook Token for GET request
webhook_token = os.environ["VERIFY_TOKEN"]

# Webhook Argument Name
wb_arg_name = os.environ["WB_ARG_NAME"]

# Custom Platform API
platform = Platform(access_token=os.environ["PAGE_ACCESS_TOKEN"])

# Custom Database API
db = Database(
    cluster=os.environ["MONGO_TOKEN"],
    db=os.environ["FIRST_CLUSTER"],
    collection=os.environ["COLLECTION_NAME"],
)

# Custom Parsing API
parser = Parser()

# Custom Hasher/Cryptographer API
guard = Guard(key=os.environ["FERNET_KEY"])
```



## `webhook()`

Enables webhook with the platform/service (like Dialogflow or Facebook Messenger). The request data is fetched (in JSON) and used by other functions. The `GET` method does not have any use but rather redirects users that navigate to `url+"/webhook"`.

```python
>>> import requests
>>> requests.post(app_url+"webhook/", headers={wb_arg_name: webhook_token})
<Response [200]>

>>> requests.get(app_url+"webhook/").url
%APP_URL%
```

|   Returns   |
|-------------|
| **`dict`**  |



## `new_user_registration()`

Registers users to the application by verifying login credentials and adding them in the database. In-registration users are distinguished using a `reg_id` attribute in their data.

```python
>>> import requests
>>> requests.post(app_url+"/register?id=123", 
                  data={"uni_id": "random_id", "uni_pw": "random_pass"}).url
%APP_URL%
```

|   Returns   |
|-------------|
| **`None`**  |



## `handle_intent(request_data, uid)`

Generates response for message if an intent is found. If the response is not handled by the app, the parser takes care of it. This function features a dictionary to map intents to functions in order to avoid a `if-elif-...-else` ladder.

```python
>>> handle_intent({"keys-to-intent": "delete data"}, "1234567890", "123456Z")
Something went wrong. :(
```

|                                           Parameters                                           |                                Returns                                   |
|------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------|
| **`request_data`:** the POST request dictionary<br>**`uid`:** the unique sender ID of the user | **`str`:** the response<br>**`None`:** if response is not handled by app |



## `user_gateway(request_data, uid)`

Acts as a gateway for all messages before understanding intents and user attributes by enabling login-integrity and fetching data from the database.

```python
>>> user_gateway({"queryText": "hi there"}, "1234567890")
APP: 1234567890 logging in again.
Hey there!
```

|                                       Parameters                                               |                                  Returns                                 |
|------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------|
| **`request_data`:** the POST request dictionary<br>**`uid`:** the unique sender ID of the user | **`str`:** the response<br>**`None`:** if response is not handled by app |



## `log(message)`

Prints essential information on the console of the server. This helps in easily changing the process of keeping logs.

```python
>>> log("lorem ipsum")
APP: lorem ipsum
```

|                 Parameters              |    Returns  |
|-----------------------------------------|-------------|
| **`message`:** the message to log       | **`None`**  |