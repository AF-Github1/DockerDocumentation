# DockerDocumentation
Self Learning Docker Documentation


# Docker Documentation

https://docs.docker.com/get-started/

# Glossary

**Container**

Isolated process that is sandboxed away from every single other process in the machine.

**Container Image**

The isolated filesystem of a container is provided by the container image, which itself contains all configurations, settings and information needed to run the application. 



# Docker installation

sudo apt update && sudo apt upgrade -y

sudo apt install ca-certificates curl gnupg lsb-release

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

docker -v

systemctl status docker --no-pager -l


sudo usermod -aG docker $USER

id $USER

newgrp docker

And if everything is working properly, this should run without issues

docker run hello-world


![image](https://github.com/AF-Github1/DockerDocumentation/assets/133685290/5f9bc3ff-dabd-472e-baf8-5bec058db31c)


# Useful commands

**docker ps**

Shows active containers

**docker logs -f CONTAINERID**

Shows logs for any given container

**docker rm -f CONTAINERID**

Forcefully removes a container

**-it**

Flag that enables running an interactive terminal

**docker image history IMAGE_NAME**

Enables you to see the history for any given image

![image](https://github.com/AF-Github1/DockerDocumentation/assets/133685290/65dbbe8b-c3aa-4d98-a80b-19bfbc8ba9da)

# Getting started with docker

https://docs.docker.com/get-started/02_our_app/

git clone https://github.com/docker/getting-started.git

cd /home/ubuntu/getting-started/app

create Dockerfile and add the following
````
# syntax=docker/dockerfile:1
 
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
EXPOSE 3000
````
Then run these commands, always in the getting-started/app directory

docker build -t getting-started .
docker run -dp 127.0.0.1:3000:3000 getting-started

And check

http://localhost:3000/ on your machines browser. The application should show up.


If you are unable to access localhost, try restarting the docker service and rerunning the docker build and docker run commands, then reload the localhost webpage.

# Updating a container

To update a container, one must first remove it

Use docker ps to check your container list

![image](https://github.com/AF-Github1/DockerDocumentation/assets/133685290/3abe24ef-ae4d-4da7-8fc1-5637f348d4f9)


Stop and remove the container you will update

docker stop CONTAINERID
docker rm CONTAINERID

If you don’t remove it before we run it again it will not let you do so, as the port 3000 is already allocated to the previous version of the container

**What to do if unable to delete container: Permission denied error**

Execute container in bash with either its ID or name

docker exec -it CONTAINERID sh

Kill the process directly

kill 1 

When you go back to check with docker ps, the container should be gone


-----------------------------------

In /home/ubuntu/getting-started/app/src/static/js

nano app.js

Find this line 

<p className="text-center">No items yet! Add one above!</p>

Replace the text with what you want

<p className="text-center">This is a documentation test</p>

Of course if you know javascript, change what you want in the code, as long as the changes are clear on the webpage later to check if you did the update process properly


Rerun these commands in the app directory

docker build -t getting-started .
docker run -dp 127.0.0.1:3000:3000 getting-started

Now when you go ahead and check the site the new text should show up.

![image](https://github.com/AF-Github1/DockerDocumentation/assets/133685290/f12da2bf-f1fb-4dc4-9234-3d488678a43c)

# Persisting changes

When you remove and reactivate then container, the data contained within is deleted. Heres how to make sure it persists across changes to the site using a database

docker volume create todo-db 


Remove previous to do container

docker rm -f CONTAINERID

Run this command (in the app directory). In this command you are identifying the database you will store the site information in. In this case it will be the list entries you add.

docker run -dp 127.0.0.1:3000:3000 --mount type=volume,src=todo-db,target=/etc/todos getting-started


Check local host and add something to the list

Delete and then recreate the container.

The same list entries you added before should now still be present on your webpage.


# Bind mounts/nodemon

Bind mount is the second type of mount available which lets you share a directory between your host and container


nodemon enables you to change the container without having to manually remove and restart it.

Run this command

docker run -dp 127.0.0.1:3000:3000 \
    -w /app --mount type=bind,src="$(pwd)",target=/app \
    node:18-alpine \
    sh -c "yarn install && yarn run dev"

The   -w /app --mount type=bind,src="$(pwd)",target=/app \ specifies where the bind will be mounted. In this case to /app 

The ‘yarn install && yarn run dev’ part of the command is the one that includes the script to start nodemon, that automatically reloads the container whenever it detects any change.

—-----------------------

In app.js, change one of the lines containing text to something else, like was done before

src/static/js/app.js

Refresh the page

The changes should show up after the page refresh, without needing to redo the whole container process. You can do this any number of times until you decide to destroy the container.


# Multi-container networking


This command creates the network

docker network create todo-app


Run this 

docker run -d \
     --network todo-app --network-alias mysql \
     -v todo-mysql-data:/var/lib/mysql \
     -e MYSQL_ROOT_PASSWORD=secret \
     -e MYSQL_DATABASE=todos \
     mysql:8.0

Don’t actually change the password at MYSQL_ROOT_PASSWORD as it’s a 50/50 of not working or bricking the container and forcing you to kill the process.

Password should always be secret when it asks you for it

Afterwards check processes with docker ps 

Then run this

docker exec -it CONTAINERID mysql -u root -p

Where CONTAINERID is your mysql instance

Inside the mysql shell you can now run queries
Like the one to check your database

SHOW DATABASES;

Use exit to leave said shell

# Connecting to mysql

Run this command. Uses a container specifically made for troubleshooting

docker run -it --network todo-app nicolaka/netshoot

Now inside the container run this command

dig mysql

Your output should look something like this

![image](https://github.com/AF-Github1/DockerDocumentation/assets/133685290/a5620341-c9b9-4a3e-b354-aa09e474433f)

What one can see here, is that Docker can use the name of network to find a host


# Running mysql with the to do app container

Inside the app directory, run this command. Names will depend on what you called previously to your own containers/networks and where you put your files. Change this accordingly

docker run -dp 127.0.0.1:3000:3000 \
   -w /app -v "$(pwd):/app" \
   --network todo-app \
   -e MYSQL_HOST=mysql \
   -e MYSQL_USER=root \
   -e MYSQL_PASSWORD=secret \
   -e MYSQL_DB=todos \
   node:18-alpine \
   sh -c "yarn install && yarn run dev"

And check your own mysql container as you add things in your localhost to do list. They should show up as you add more entries to your list

docker exec -it CONTAINERID mysql -p todos

![image](https://github.com/AF-Github1/DockerDocumentation/assets/133685290/17302139-a093-46bb-8025-cd3c43f86129)















































































