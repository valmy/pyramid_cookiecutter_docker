pyramid-cookiecutter-docker
===========================

## Bootstraping docker pipenv pyramid system

Resources:
- Read the first part in https://github.com/valmy/pyramid-docker
- https://docs.pylonsproject.org/projects/pyramid/en/latest/quick_tour.html

### Execute cookiecutter

Since cookiecutter's purpose is to generate the directory template for the actual application
we run it on the host OS's pipenv without docker, and then use setup the environment for result
in docker.

1. Setup env for cookiecutter

   ```
   $ mkdir cookiecutter
   $ cd cookiecutter
   $ pipenv --three
   $ pipenv install cookiecutter
   ```
2. Run cookiecutter

   ```
   $ pipenv run cookiecutter gh:Pylons/pyramid-cookiecutter-starter --checkout 1.10-branch
   ```
   See https://docs.pylonsproject.org/projects/pyramid/en/latest/quick_tour.html#quick-project-startup-with-cookiecutters

3. Use the generated folder as the root folder for the next section

Alternatively we can use docker CLI to run this without Python on the host OS:

```
$ docker run -it -v `pwd`:/app kennethreitz/pipenv
# pipenv --three
# pipenv install cookiecutter
# pipenv run cookiecutter gh:Pylons/pyramid-cookiecutter-starter --checkout 1.10-branch
```

### Prepare docker environment with pipenv

1. Start with simple `docker-compose.yml` file:
   ```
   version: '3'
   services:

     webapp:
       image: kennethreitz/pipenv
       volumes:
         - .:/app
         - ./.local:/root/.local
       command: tail -f /dev/null
   ```
   We mount our code folder to /app and `./.local` to store the virtualenv of our app


2. Get the container up:

   ```
   $ docker-compose up
   ```

3. From another shell, get into the container:

   ```
   $ docker-compose exec webapp bash
   ```

4. Setup an env and install pyramid

   ```
   # pipenv --three
   # pipenv install -e ".[testing]"
   ```

5. Get to the environment

   ```
   # pipenv shell
   ```

6. Edit docker-compose.yml

   ```
   version: '3'
   services:

     webapp:
       image: kennethreitz/pipenv
       volumes:
         - .:/app
         - ./.local:/root/.local
       ports:
         - 6543:6543
       command: pipenv run pserve development.ini --reload
   ```

    Edit in development.ini:
    `listen = localhost:6543` to `listen = 0.0.0.0:6543`

    Activate debugtoolbar:
    `debugtoolbar.hosts = 0.0.0.0/0`


7. Run pytest:
```
docker-compose exec webapp pipenv run pytest --cov --cov-report=term-missing
```
