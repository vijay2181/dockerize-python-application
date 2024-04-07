# dockerize-python-application

1.Create a directory for the project:
```
mkdir composetest
cd composetest
```

2.Create a file called app.py in your project directory and paste the following code in:
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

In this example, redis is the hostname of the redis container on the application's network and the default port, 6379 is used.

3.Create another file called requirements.txt in your project directory and paste the following code in:
```
flask
redis
```

4.Create a Dockerfile and paste the following code in:
```
# syntax=docker/dockerfile:1
FROM python:3.10-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run", "--debug"]
```

5.Create a file called compose.yaml in your project directory:

- Compose simplifies the control of your entire application stack, making it easy to manage services, networks, and volumes in a single, comprehensible YAML configuration file.

```
services:
  web:
    build: .
    ports:
      - "8000:5000"
  redis:
    image: "redis:alpine"
```

- This Compose file defines two services: web and redis.

- The web service uses an image that's built from the Dockerfile in the current directory. It then binds the container and the host machine to the exposed port, 8000. This example service uses the default port for the Flask web server, 5000.

- The redis service uses a public Redis image pulled from the Docker Hub registry.

6.Build and run your app with Compose:

```
 docker compose up

 docker compose up -d
```

- Enter http://<public-ip>:8000/ in a browser to see the application running.

![image](https://github.com/vijay2181/dockerize-python-application/assets/66196388/f92f6745-91b9-4329-a569-35c1a297c971)

- Refresh the page.

![image](https://github.com/vijay2181/dockerize-python-application/assets/66196388/7697e679-90bc-4b47-b63f-01c82bf50021)

7.Switch to another terminal window, and type docker image ls to list local images.

- Listing images at this point should return redis and web.

```
docker image ls

REPOSITORY        TAG           IMAGE ID      CREATED        SIZE
composetest_web   latest        e2c21aa48cc1  4 minutes ago  93.8MB
python            3.4-alpine    84e6077c7ab6  7 days ago     82.5MB
redis             alpine        9d8fa9aa0e5b  3 weeks ago    27.5MB
```

8.Edit the Compose file to use Compose Watch

- Edit the compose.yaml file in your project directory to use watch so you can preview your running Compose services which are automatically updated as you edit and save your code:

```
services:
  web:
    build: .
    ports:
      - "8000:5000"
    develop:
      watch:
        - action: sync
          path: .
          target: /code
  redis:
    image: "redis:alpine"
```

- Whenever a file is changed, Compose syncs the file to the corresponding location under /code inside the container. Once copied, the bundler updates the running application without a restart.

```
For this example to work, the --debug option is added to the Dockerfile. The --debug option in Flask enables automatic code reload, making it possible to work on the backend API without the need to restart or rebuild the container. After changing the .py file, subsequent API calls will use the new code, but the browser UI will not automatically refresh in this small example. Most frontend development servers include native live reload support that works with Compose.
```

9.Re-build and run the app with Compose

- From your project directory, type docker compose watch or docker compose up --watch to build and launch the app and start the file watch mode.

```
docker compose watch
[+] Running 2/2
 â Container docs-redis-1 Created                                                                                                                                                                                                        0.0s
 â Container docs-web-1    Recreated                                                                                                                                                                                                      0.1s
Attaching to redis-1, web-1
         â¦¿ watch enabled
...
```

10.Update the application

- Change the greeting in app.py and save it. For example, change the Hello World! message to Hello from Docker!:

```
return 'Hello from Docker! I have been seen {} times.\n'.format(count)
```

Refresh the app in your browser. The greeting should be updated, and the counter should still be incrementing.

![image](https://github.com/vijay2181/dockerize-python-application/assets/66196388/5376c7c4-21e3-453e-93b4-e556d4822096)


11.Split up your services

- Using multiple Compose files lets you customize a Compose application for different environments or workflows. This is useful for large applications that may use dozens of containers, with ownership distributed across multiple teams.
- In your project folder, create a new Compose file called infra.yaml.
- Cut the Redis service from your compose.yaml file and paste it into your new infra.yaml file. Make sure you add the services top-level attribute at the top of your file. Your infra.yaml file should now look like this:

```
services:
  redis:
    image: "redis:alpine"
```

- In your compose.yaml file, add the include top-level attribute along with the path to the infra.yaml file.

```
include:
   - infra.yaml
services:
  web:
    build: .
    ports:
      - "8000:5000"
    develop:
      watch:
        - action: sync
          path: .
          target: /code
```

Run docker compose up to build the app with the updated Compose files, and run it. You should see the Hello world message in your browser.

This is a simplified example, but it demonstrates the basic principle of include and how it can make it easier to modularize complex applications into sub-Compose files

Reference: https://docs.docker.com/compose/gettingstarted/



















