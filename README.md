# kobo-docker

## Local Only

`kobo-docker` is used to run a copy of the [KoBo Toolbox](http://www.kobotoolbox.org) survey data collection platform on a machine of your choosing. It relies [Docker](https://docker.com) to separate the different parts of KoBo into different containers (which can be thought of as lighter-weight virtual machines) and [Docker Compose](https://docs.docker.com/compose/) to configure, run, and connect those containers. Below is a diagram (made with [Lucidchart](lucidchart.com)) of the containers that make up a running `kobo-docker` system and their connections:
![Container diagram](./doc/Container_diagram.png)

# Setup procedure:

1. Clone this repository, retaining the directory name `kobo-docker`.

2. [Install Docker Compose for Linux on x86-64](https://docs.docker.com/compose/install/). Power users on Mac OS X and Windows can try [the new Docker beta for those platforms](https://blog.docker.com/2016/03/docker-for-mac-windows-beta/), but there are known issues with filesystem syncing on those platforms.

3. Pull the latest images from Docker Hub: `docker-compose pull`. **Note:** Pulling updated images doesn't remove the old ones, so if your drive is filling up, try removing outdated images with e.g. `docker rmi`.

4. Edit the appropriate environment file for your instance type, [`envfile.local.txt`](./envfile.local.txt) filling in **all** mandatory variables. Host address is usually just 172.17.0.1. ENKETO_API_TOKEN can be anything, DJANGO_SECRET_KEY has to be 50 random characters while the rest of the admin details can be set to whatever you want.

5. Optionally enable additional settings for your Google Analytics token, S3 bucket, e-mail settings, etc. by editing the files in [`envfiles/`](./envfiles).


6. Build any images you've chosen to manually override: `docker-compose build`.

7. Start the server: `docker-compose up -d` (or without the `-d` option to run in the foreground).

8. Container output can be followed with `docker-compose logs -f`. For an individual container, logs can be followed by using the container name from your `docker-compose.yml` with e.g. `docker-compose logs -f enketo_express`.

"Local" setup users can now reach KoBo Toolbox at `http://${HOST_ADDRESS}:${KPI_PUBLIC_PORT}` (substituting in the values entered in [`envfile.local.txt`](./envfile.local.txt)).

# Troubleshooting

## Basic troubleshooting
You can confirm that your containers are running with `docker ps`. To inspect the log output from the containers, execute `docker-compose logs -f` or for a specific container use e.g. `docker-compose logs -f redis_main`.

The documentation for Docker can be found at https://docs.docker.com.

## Django debugging
Developers can use [PyDev](http://www.pydev.org/)'s [remote, graphical Python debugger](http://www.pydev.org/manual_adv_remote_debugger.html) to debug Python/Django code. To enable for the `kpi` container:

1. Specify the mapping(s) between target Python source/library paths on the debugging machine to the locations of those files/directories inside the container by customizing and uncommenting the `KPI_PATH_FROM_ECLIPSE_TO_PYTHON_PAIRS` variable in [`envfiles/kpi.txt`](./envfiles/kpi.txt).
2. Share the source directory of the PyDev remote debugger plugin into the container by customizing (taking care to note the actual location of the version-numbered directory) and uncommenting the relevant `volumes` entry in your `docker-compose.yml`.
3. To ensure PyDev shows you the same version of the code as is being run in the container, share your live version of any target Python source/library files/directories into the container by customizing and uncommenting the relevant `volumes` entry in your `docker-compose.yml`.
4. Start the PyDev remote debugger server and ensure that no firewall or other settings will prevent the containers from connecting to your debugging machine at the reported port.
5. Breakpoints can be inserted with e.g. `import pydevd; pydevd.settrace('${DEBUGGING_MACHINE_IP}')`.

Remote debugging in the `kobocat` container can be accomplished in a similar manner.
