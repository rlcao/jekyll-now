# 1. Prerequiste Knowledge
##### 1.1 What is a Session?
- A context in which a client communicates with a server over multiple http requests
- http protocol is stateless, while session is here to save
- Session is helping keep a set of data over multiple http requests
##### 1.2 Http Cookie Header Explained
Usually, session is built upon cookie, hence we need to understand how cookie works.
By using Cookies we can exchange information between the server and the browser to provide a way to customize a user session, and for servers to recognize the user between requests.
- Http Request without Cookie(first time request)
  - Client send request without cookie
  - Server generate a cookie, and send respose with header 'set-cookie', example:
  ```
  set-cookie: PHPSESSID=3fl0***********ug1k4uj7; path=/; expires=Wed,10-Oct-19 07:12:20 GMT
  ```
- Subequent Request Workflow
  - Client send request with cookie:
  ```
  Cookie: PHPSESSID=3fl0***********ug1k4uj7
  ```
Cookies are essentially used to store a session id.
In the past cookies were used to store various types of data, since there was no alternative. But nowadays with the Web Storage API (Local Storage and Session Storage) and IndexedDB, we have much better alternatives.
Especially because cookies have a very low limit in the data they can hold, since they are sent back-and-forth for every HTTP request to our server - including requests for assets like images or CSS/JavaScript files.
**Restrictions of cookies:**
- Cookies can only store 4KB of data
- Cookies are private to the domain. A site can only read the cookies it set, not other domains cookies
- You can have up to 20 limits of cookies per domain (but the exact number depends on the specific browser implementation)
- Cookies are limited in their total number (but the exact number depends on the specific browser implementation). If this number is exceeded, new cookies replace the older ones.
**Cookies can be set or read server side, or client side.**
##### 1.3 PHP Session Explained
- default cookie key name: PHPSESSID
- session_start() will start a session in sever, and create a session file in temp dir with name: sess_<session id>
- session_destroy() will close the session, and delete the session file on temp dir
- the session id is stored in cookie
- session will time out if there is no update on session data in a period of time(configurable)
- session id can be regenerated for purpose of security
- php server wide configuration reside in /etc/php.ini file, where you can find the session file save path.
- session data can be stored in database where you can share session between server nodes
- storing data using $_SESSION["name"] = "rongcao", unset $_SESSION["name"] will delete the key from session data
##### 1.4 Python Flask-HttpBasicAuth Package
Here are the notes for document: https://flask-httpauth.readthedocs.io/en/latest/
- Create HttpBasicAuth instance, which will hold decorator callback data registerred
```
from flask import Flask
from flask_httpauth import HTTPBasicAuth

app = Flask(__name__)
auth = HTTPBasicAuth()

users = {
    "john": "hello",
    "susan": "bye"
}

@auth.get_password
def get_pw(username):
    if username in users:
        return users.get(username)
    return None

@app.route('/')
@auth.login_required
def index():
    return "Hello, %s!" % auth.username()

if __name__ == '__main__':
    app.run()
```
- Register verify_password function:
```
@auth.verify_password
def verify_pw(username, password):
    return call_custom_verify_function(username, password)
```
- Check Authorization by decorator auth_inst.login_reqired
```
@app.route('/')
@auth.login_required
def index():
    return "Hello, %s!" % auth.username()
```
Here is simpler version of Flask-HttpBasicAuth:
```
from functools import wraps
from flask import request
def login_required(f):
    @wraps(f)
    def wrapped_view(**kwargs):
        auth = request.authorization
        if not (auth and check_auth(auth.username, auth.password)):
            return ('Unauthorized', 401, {
                'WWW-Authenticate': 'WWW-Authenticate': 'Basic realm="Login Required"'
            })

        return f(**kwargs)
    return wrapped_view

@app.route('/secret')
@login_required
def secret():
    return f'Logged in as {request.authorization.username}.'
import requests
response = requests.get('http://127.0.0.1:5000/secret', auth=('world', 'hello'))
print(response.text)
```
##### 1.5 Python itsdangerous Package
Package Home Page: https://pythonhosted.org/itsdangerous/
1. code to generate token
```python
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer
from itsdangerous import SignatureExpired, BadSignature
from config import config

def gen_token(user, expiration=1440*31*60):  # 单位为秒, 设定31天过期
    s = Serializer(config.SECRET_KEY, expires_in=expiration)
    return s.dumps({'id': user.id})  # user为model中封装过的对象
```
2. code to verify token
```
from functools import wraps

def token_required(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        token = request.form['token']
        s = Serializer(config.SECRET_KEY)
        try:
            data = s.loads(token)
        except SignatureExpired:
            return jsonify({'status': 'fail', 'data': {'msg': 'expired token'}})
        except BadSignature:
            return jsonify({'status': 'fail', 'data': {'msg': 'useless token'}})
        kwargs['user_id'] = data['id']
        return func(*args, **kwargs)
    return wrapper
    
@user.route('/follow', methods=['POST'])
@token_required
def follow_user(user_id):
    user_to_follow_id = request.form['user_id']
    user_rel = UserRel(user_id, user_to_follow_id)
    model = Model()
    if model.get_user_rel(user_id, user_to_follow_id):
        return to_json('already follow')
    user_to_follow = model.session.query(UserInfo).filter_by(user_id=user_to_follow_id).first()
    data = user_to_follow.data()
    model.session.add(user_rel)
    model.session.commit()
    return to_json(data, success=True)  # 将flask中的jsonify封装了一层
```
# Put it all Together
**Architecture:**
![image](https://user-images.githubusercontent.com/27807070/54362100-50c1c300-46a3-11e9-840c-852a22e17190.png)
**Workflow:**
![image](https://user-images.githubusercontent.com/27807070/54362172-6f27be80-46a3-11e9-9252-e7ce90391b39.png)
**Session Setup in Steps:**
1. Client send username/password data via https connection to server
```
#post following data to ui login
username=<username>
password=<password>
a=login
```
2. Server authorize the username, and generate token, save the token into session data. It then send session id as PHPSESSID value in set-cookie response header
- generate token
```php
curl_setopt_array($curlobj, array(
             CURLOPT_URL => $login_url,
             CURLOPT_USERAGENT => 'Request',
             CURLOPT_USERPWD => "$username:$password",
             CURLOPT_RETURNTRANSFER => 1,
             CURLOPT_SSL_VERIFYPEER, false,
             //curl_setopt($curlobj,CURLOPT_HTTPHEADER, array('Content-Type:application/json')),
             CURLOPT_POST => 0));
     # execute request
     $res = curl_exec($curlobj);
```
```python
    def generate_auth_token(self, expiration = 86400):
        """
        Helper function to create token
        """
        s = Serializer(app.config['SECRET_KEY'], expires_in = expiration)
        return s.dumps({'id': self.id})
......
# user/login
@mod.route('/login', methods=['GET'])
@ma.login_required
def user_login():
    """
    LOGIN method
    This view/function returns an auth cookie with expire time 1 hour
    """
    user = mk.User.query.filter_by(username=request.authorization.username).first()
    if user is None:
        app.logger.info('%s %r', "Exited, unable to find in database:", request.authorization.username, extra=d)
        return seasoned_response([], "404", "Not Found: unable to find in database" + request.authorization.username)

    cookie = user.generate_auth_token()
    role = mk.User.query.filter_by(username=request.authorization.username).first().role
    result = { "role": role, "cookie": cookie }
    app.logger.info('%s %r', "Output:", result, extra=d)
    return seasoned_response(result, "200")
```
- save the token in session
```php
$_SESSION[sess_coookie] = $rs['cookie'];
$_SESSION[sess_userauthorized] = true;
```
- set the session id by
```php
session_start()
```
3. Client send request with cookie set in request headers
```http
Cookie: PHPSESSID=3fl06lbf8aamkq8p0cug1k4uj7;
```
4. Server figure out the token base on the PHPSESSID in cookie in request header, and verify if the token is valid.
```php
 # the login block
 if (isset($_SESSION[sess_userauthorized]) && $_SESSION[sess_userauthorized])
 {
     $user = $_SESSION[sess_username];
     $email = $_SESSION[sess_useremail];
     $fullname = $_SESSION[sess_userfullname];
     ......
```
# Reference
Cookie and Session: https://zhuanlan.zhihu.com/p/27669892
Cookie Explained: https://flaviocopes.com/cookies/
Flask HttpAuth: https://flask-httpauth.readthedocs.io/en/latest/

# NOTE
There is a technology that recently got popular, called JSON Web Tokens (JWT), which is a Token-based Authentication.

# Enhancements
- Store session data in database
- HAProxy/KeepAlived
- Message Queue?
- Microservice
- CI/CD(deployment)
- DNS Based Load Balancer
- Centralize Log Using ELK/Splunk
