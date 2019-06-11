# Get started with Docker Compose

This is extracted from [official Docker documentation](https://docs.docker.com/compose/gettingstarted/)
### Prerequisites
Make sure you have already installed both Docker Engine and Docker Compose.
You don’t need to install Python or Redis, as both are provided by Docker images.
[Docker for Windows documentation including installation ](https://docs.docker.com/docker-for-windows/)

I personally prefer to run docker from Ubuntu on Windows10 even though using Windows Docker Engine.
If you want to install Ubuntu (or any other Linux distro) on windows 10, please follow instruction on this [link](https://docs.microsoft.com/en-us/windows/wsl/install-win10)
You may need to expose Docker daemon on `tcp://localhost:2375`
In the bash shell you may need to add following line in the `.profile` file.
```sh
export DOCKER_HOST=localhost:2375
```

In case you don't install docker-compose:
```sudo apt install docker-compose```

### Step 1: Setup
Define the application dependencies.

1. Create a directory for the project:
```
$ mkdir composetest
$ cd composetest
```
2. Create a file called `app.py` in your project directory and paste this in:
```
import time
import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)

```
3. Create another file called `requirements.txt` in your project directory and paste this in:
```
flask
redis
```

### Step 2: Create a Dockerfile
Create a file named `Dockerfile` and paste the following:
```
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP app.py
ENV FLASK_RUN_HOST 0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
COPY . .
CMD ["flask", "run"]
```

### Step 3: Define services in a Compose file
Create a file called `docker-compose.yml` in your project directory and paste the following:
```
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
```
This Compose file defines two services: `web` and `redis`.
#### Web service
The `web` service uses an image that’s built from the `Dockerfile` in the current directory. 
It then binds the container and the host machine to the exposed port, `5000`.
This example service uses the default port for the Flask web server, `5000`.

#### Redis service
The `redis` service uses a public Redis image pulled from the Docker Hub registry.

### Step 4: Build and run your app with Compose
1. From your project directory, start up your application by running 
`docker-compose up`
```sh
$ docker-compose up
Creating network "composetest_default" with the default driver
Creating composetest_web_1 ...
Creating composetest_redis_1 ...
Creating composetest_web_1
Creating composetest_redis_1 ... done
Attaching to composetest_web_1, composetest_redis_1
web_1    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
redis_1  | 1:C 17 Aug 22:11:10.480 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
redis_1  | 1:C 17 Aug 22:11:10.480 # Redis version=4.0.1, bits=64, commit=00000000, modified=0, pid=1, just started
redis_1  | 1:C 17 Aug 22:11:10.480 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
web_1    |  * Restarting with stat
redis_1  | 1:M 17 Aug 22:11:10.483 * Running mode=standalone, port=6379.
redis_1  | 1:M 17 Aug 22:11:10.483 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
web_1    |  * Debugger is active!
redis_1  | 1:M 17 Aug 22:11:10.483 # Server initialized
redis_1  | 1:M 17 Aug 22:11:10.483 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
web_1    |  * Debugger PIN: 330-787-903
redis_1  | 1:M 17 Aug 22:11:10.483 * Ready to accept connections
```
2. Enter http://localhost:5000/ in a browser to see the application running.
