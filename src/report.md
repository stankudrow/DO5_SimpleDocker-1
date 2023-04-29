# DO5_SimpleDocker Report

## Preface

* Please refer to the ["Docker useful info"](https://21-school-by-students.notion.site/Docker-install-on-school-iMacs-9354ef106a8a40c6b46a69cea0a11bf8) guide before you go further. If you run docker at the School 21, you may encounter the error `docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?`. Consider launching **Docker Desktop** application and try again.

* It is a good idea to install Dockle for linting docker images.

## Part 1

01. `docker pull nginx`  
![pull_nginx](part1/screenshots/part1_step01_docker_pull_nginx.png)

02. `docker images`  
![images](part1/screenshots/part1_step02_docker_images.png)

03. `docker run -d --name nginxer nginx`  
![run_nginx](part1/screenshots/part1_step03_docker_run_nginx.png)

04. `docker ps` (ps like "Process Status")  
![ps](part1/screenshots/part1_step04_docker_ps.png)

05. The info on the nginx container:

    01. IP = 172.17.0.2 (`docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' nginxer`)  
    ![nginx_ip](part1/screenshots/part1_step05_docker_inspect_ip.png)

    02. SIZE = 1095 bytes (`docker inspect --size -f 'RW = {{ .SizeRw }}; ROOTFS = {{ .SizeRootFs }}' nginxer`)  
    ![nginx_ip](part1/screenshots/part1_step05_docker_inspect_size.png)  
    Also, `docker ps -s`:  
    ![docker_ps_s](part1/screenshots/part1_step05_docker_ps_size.png)

    03. List of mapped ports (`docker inspect --type=container -f '{{.NetworkSettings.Ports}}' nginxer`)  
    ![nginx_mapped_ports](part1/screenshots/part1_step05_docker_inspect_ports.png)

06. `docker stop`  
![stop](part1/screenshots/part1_step06_docker_stop.png)

07. `docker ps` after having the container stopped  
![ps_after_stop](part1/screenshots/part1_step07_docker_ps_after_docker_stop.png)

08. `docker run -d -p 80:80 -p 443:443 --name nginxer2 nginx`  
![with_ports](part1/screenshots/part1_step08_docker_run_with_ports.png)

09. `docker ps`  
![nginxer2](part1/screenshots/part1_step09_nginx_ps_ports.png)

10. Nginx is working at the port 80  
![nginx_80](part1/screenshots/part1_step09_nginx_80.png)

11. Nginx is restarted and works as well, but the name has changed  
![restarted](part1/screenshots/part1_step10_nginx_restarted.png)

## Part 2

A [course](https://www.youtube.com/watch?v=dHsv8EnhSgQ&list=PLhgRAQ8BwWFa7ulOkX0qi5UfVizGD_-Rc) about NGINX for a quick start.

0. Assuming we are in the **part2** directory.

1. `docker exec nginxer2 cat /etc/nginx/nginx.conf`.  
![origin_nginx_conf](part2/screenshots/part2_step01_nginx_conf.png)

2. Copying the basic config file, pasting it in a local config file (see **part2/nginx_origin.conf**) and then enrich it with the \[local\] server configuration. Please refer to the **./part2/nginx.conf** file for details. This file should replace the NGINX configuration in the container. When it is done, reloading the server  
![new_nginx_conf](part2/screenshots/part2_step02_nginx_new_conf.png)

3. Reloading (`docker exec nginxer2 nginx -s reload`) and checking the **localhost:80/status** (`curl localhost:80/status`):  
![new_nginx_results](part2/screenshots/part2_step03_nginx_new_status.png)

4. Exporting the container into a **.tar** archive (`docker export nginxer2 > nginxer.tar`)  
![container.tar](part2/screenshots/part2_step04_nginxer2_tar.png)

5. Stopping containers, removing images and containers - cleansing routine.
![stop_and_remove](part2/screenshots/part2_step05_stopping_removing.png)

6. Importing with `docker import -c 'CMD ["nginx", "-g", "daemon off;"]' ./nginxer2.tar re_nginx` and launching the image via `docker run --rm -d -p 80:80 -p 443:443 --name nginxer3 re_nginx`:
![import_relaunch](part2/screenshots/part2_step06_import_relaunch.png)

## Part 3

The part is to write a "Hello, world!" mini server in C using the FastCGI module.

0. Assuming we are in the **part3** directory and consider to run `bash launch_part3.sh` script.

1. Cleaning up the ground.  
![cleaning](part3/screenshots/part3_step00_cleaning.png)

2. `docker pull nginx` - yes, again if needed.

3. Running the nginx container (`docker run --rm -d -p 81:81 --name server nginx`) and then executing the following commands:

    1. `docker exec server apt update` - checking the system for updates;

    2. `docker exec server apt upgrade -y` if you want to upgrade your container;

    3. `docker exec server apt install -y gcc spawn-fcgi libfcgi-dev` - only necessary tools for compiling a FastCGI server;

    4. commiting the changes: `docker commit $CONTAINER_ID [new_image]`  
    ![commited](part3/screenshots/part3_step01_commiting.png)

    5. copying the files from the **part3** directory:

        1. `docker cp nginx.conf server:/etc/nginx/`;

        2. `docker cp fastcgi_server.c server:/`

    6. `docker exec server gcc ./fastcgi_server.c -l fcgi -o fcgi_server` - compiling the FastCGI server.

    7. `docker exec server spawn-fcgi -p 8080 fcgi_server` - launching;

    8. `docker exec server nginx -s reload` - a new nginx configuration is in effect;

4. Executing locally: `curl http://localhost:81/` to behold our server alive.  
![fcgi_alive](part3/screenshots/part3_step02_fcgi_alive.png)

Note: **part3/fastcgi_server.c** file has some useful references.

## Part 4

0. Assuming we are in the **part4** directory and consider to run `bash launch_part4.sh` script.

1. The **part4** containes necessary files for building a container.

2. `docker build . -t server:v2` - building the image;

3. `docker images` - to be sure that the building has been done successfully;

4. running `docker run --rm -d -p 80:81 --name server2 server:v2` instance;

5. `curl localhost:80` -> "Hello World!", `curl localhost:80/status` -> nginx status.

Results:  
![dockefile_server](part4/screenshots/part4_results.png)

## Part 5

I found no `brew` on my MAc, so first installing it via `curl -fsSL https://rawgit.com/kube/42homebrew/master/install.sh | zsh` (got from [school21_faq](https://github.com/daniiomir/faq_for_school_21)). Then, installing the Dockle tool with `brew install goodwithtech/r/dockle` command (see [Dockle](https://github.com/goodwithtech/dockle) project page). For other platforms, please refer to [this post](https://habr.com/ru/companies/timeweb/articles/561378/).

0. Assuming we are in the **part5** directory and consider to run `bash launch_part5.sh` script.

1. `dockle [YOUR_IMAGE_NAME]` - here you may witness problems;

2. Time for the third (3) version of the server container:

    0. stop and clean what you need (`docker system prune -a`);

    1. in the shell: `export DOCKER_CONTENT_TRUST=1`;

    2. `docker build . -t server:v3`;

    3. `dockle -ak NGINX_GPGKEY -ak NGINX_GPGKEY_PATH server:v3`;

    4. in the shell: `export DOCKER_CONTENT_TRUST=0` (see the screenshot with the results below);

    5. `docker run --rm -it -d -p 80:81 --name server3 server:v3` (for -it option, refer to this [answer](https://stackoverflow.com/questions/29957653/docker-container-not-starting-docker-start));

    6. `curl localhost:80` - hello, world;

    7. `curl localhost:80/status` - nginx status;

Results:  
![dockled_results](part5/screenshots/part5_step00_dockle_report_after.png)

## Part 6

0. Assuming we are in the **part6** directory, prepare at least two (2) terminals.

1. In a terminal: `docker-compose up --build`

2. In another terminal: `curl` commands and then `docker-compose stop`

Result:  
![composed_results](part6/screenshots/part6_results.png)

That is it.
