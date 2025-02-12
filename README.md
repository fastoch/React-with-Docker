# React-with-Docker

React + Docker Tutorial: https://www.youtube.com/watch?v=zi6cdUyZ_uQ&t=4s  

---

# Why use Docker?

Docker is one of the most critical skills to have in the current job market (2025).  
It gives you a massive advantage over developers who don't have this skill.  

# What we'll learn

- How to dockerize a React application using **dockerfile** along with **docker-compose**
- How to deploy this dockerized app on a private server (no Netlify, no Vercel)
- How the apps are actually deployed in the real world

# What is Docker?

Docker allows you to separate your application from your infrastructure so you can deliver software quickly.  
By encapsulating an application and its dependencies within a lightweight container, Docker empowers developers to create consistent, isolated environments  
that function seamlessly across various computing environments, from development to production.  

Imagine that you're building a React app, it probably requires various different things to run properly.  
For example, there might be certain dependencies who run at a specific Node version, let's say Node 19.  
And if you send this app to some other person who has Node 20 installed, the application could start to break.  

We use Docker to prevent these things from happening.  
Imagine Docker to be a shipping **container** that contains your app, the exact Node version it needs, and all the required depedencies.  
This container can be taken **anywhere** in any single system, and it will run exactly the same.  

Which means you no longer have to worry about your app working fine on your machine but not working on someone else's.  

---

# Lets' get to work

- start VS Code
- install the Docker extension for VS Code
- clone this repo: https://github.com/piyush-eon/tanstack-query-weather-app
- install Docker:
  - `sudo pacman -S docker docker-compose docker-buildx` on Arch Linux)
  - to check installation: `docker --version`
  - To enable Docker to run on boot: `sudo systemctl enable docker.service`
  - To start it manually: `sudo systemctl start docker.service`
  - to ensure it is running: `systemctl status docker`

---

## Side note 

- **docker**: The Docker engine itself
- **docker-compose**: A tool for managing multi-container Docker applications using Compose files
- **docker-buildx**: A CLI tool extending Docker build capabilities with new features, allowing you to build container images for multiple platforms

---

# Our first Dockerfile

In VS Code, create a file at the root of your React project and name it `Dockerfile`.  
Now we need to write some commands in this Dockerfile so that Docker knows how it is supposed to run our application: 
```dockerfile
FROM node:20-alpine

WORKDIR /myReactApp

COPY package*.json .

RUN npm install

COPY . .

EXPOSE 5173

CMD [ "npm","run","dev" ]
```

## Explainging the dockerfile

The first line sets the **base image** for the Docker container, which in our case will be a lightweight Linux system having Node and npm installed.   
The alpine image is derived from Alpine Linux, known for its small size, helping to keep the final image size down.  

The next instruction sets the **working directory** inside the container to /myReactApp.  
Subsequent commands, like COPY, will be executed in this directory. If the directory doesn't exist, Docker will create it.  

The next line copies the package.json and package-lock.json files from your project's root directory to the working directory (/myReactApp) in the container.  
The `.` notation means "current folder" on any Linux system.  

The next line tells docker to run the specified command.  
In our case, it will install all the dependencies needed to run our app.  
Side note: the list of dependencies is stored in the package.json file located at the root of your project.  

Then, we ask Docker to copy all of the remaining files from our project folder (hosted on our local machine) to the working directory that will be located inside our Docker container.  

After that, we expose the port on which our app will run, so that it can be accessed from outside the docker container.   
If we have created our app using **Vite**, our dev server will be available at http://localhost:5173/   
For older apps built using CRA (create react app), the port number will be 3000.  

And finally, we use `CMD` to start our App (dev server) when we start the docker image.  

---

Note that `RUN` executes commands during the Docker image build process. Each `RUN` instruction adds a new layer to the Docker image.  
It is used to install software packages, set environment variables, and perform configurations needed for the application.  

While `CMD` specifies the default command to run when a container is started from the Docker image.

---

## About the node_modules folder

When we run `npm install`, this creates a `node_modules` folder.  
This folder contains all the external modules and dependencies required by your Node.js project.  
You don't want to copy this folder to your container, which is why you need to create a `.dockerignore` file at the root of your project.  
This is the same logic as including the `node_modules` folder inside our `.gitignore` file.  

Inside the `.dockerignore` file, write the following and save:
```.dockerignore
node_modules
```

---

## DO NOT FORGET THE .env FILE

The `.env` file is a configuration file that stores environment-specific variables, such as database credentials, API keys, and global constants.  
For obvious reasons, it's best practice to keep this kind of data separate from the application code, in a dedicated `.env` file.

If your project does not contain a `.env` file yet, you need to create this file at the root of your project.  
Then, add the environment variables your app needs to this file.

## Modify the package.json file

We need to make sure our `dev` script (the one in `npm run dev`) can run on any machine, regardless of its IP address.  
For that, we modify the dev script by adding a `--host` flag:
```json
"scripts": {
  "dev": "vite --host 0.0.0.0",
}
```

---

# Time to dockerize our App

- from a terminal, `cd` into your project folder, and run `docker build -t appName:dev .`
  - the -t option allows us to tag the image with a unique identifier
  - in the above example, we provide the app name and we indicate it is the dev environment (to differentiate from the prod environment)
  - the final dot indicates the location of all the files needed to run our app, meaning "the current folder" 
- if the docker file was not named "dockerfile", then you'd need to add `-f fileName` to the previous command

Once your docker image has been created, run `docker images` to display all available images.  

# Let's use our newly created image

A Docker image is a read-only template or blueprint used to create containers.  
It contains the application, libraries, dependencies, tools, and other files needed for a container to run.  

To run a container built from our image: `docker run -p 5173:5173 --env-file .env appName:dev`
- the -p option maps port 5173 on the host machine to port 5173 on the Docker container
- This allows you to access an application running inside the container on port 5173 via 'localhost:5173' in your web browser

---

# How apps are dockerized in the real world

In the real world, your app might also contain a backend, not only the React frontend part.  
We need to run all of these services together, and for that we need a `docker-compose.yml` file.  

- at the root of our project folder, let's create a new file called `docker-compose.yml`
- Here's an example of what it might contain:
```yml
version: '3.8'

services:
  frontend:
    build:
      context: .
      dockerfile: Dockerfile

    ports:
      - "5173:5173"

    volumes:
      - .:/myReactApp
      - /myReactApp/node_modules

    env_file:
      - .env

  backend:
    ...
    whatever code you need for your backend
    ...
```

- once your `docker-compose.yml` file is ready, run `docker compose up`
  - this will build your frontend and backend services, handle the network config for you, and then it will create a Docker container in which your app will be running

---

# Hosting our app on a VPS

## Setting up a VPS
  
There are VPS (Virtual Private Server) hosting providers: Hostinger, Linode, Ionos, Kamatera, etc.  
- once you have subscribed to one of them, you need to setup your server
- you'll probably need to select the server location, choose the closest to you or your audience
- select your OS, with Docker pre-installed if possible
- generate an SSH key pair locally and add the public SSH key to your VPS so you can access it securely
  - for that, open a terminal and run `ssh-keygen -t ed25519 -C "comment_to_identifiy_the_key"`
  - choose where to save the key on your local machine
  - add a passphrase for more security
  - `cd ~/.ssh` or `cd <your_SSH_key_location>`
  - `ls` to check the presence of your SSH key pair in the current folder
  - `cat id_ed25519.pub` to display the public key in your terminal
  - copy the public key and add it to your VPS configuration 
- provide the root pwd that will be used to log in to your VPS
- your VPS provider might take a few minutes to set up your new VPS

## SSH into the VPS

- once your VPS is ready, copy its public IP address and use it in the following command:
```bash
ssh root@<VPS_IP_address> 
```
- enter the SSH passphrase
- once connected to your VPS via SSH, make sure Docker is installed with `docker --version`

## Transfer our project files to the VPS

Now, you need to transfer all the files needed for running your application to this VPS.
  - once connected to your VPS via SSH, create a project folder on your VPS: `mkdir app_name`
  - place yourself inside this folder: `cd app_name`
  - open a new terminal to transfer the project files from your local machine to the VPS
    - `cd` into your local project folder
    - run `scp -r "/path/to/local/project/folder" root@<VPS_IP_address>:~/app_name` 

Now you can go back to the terminal where you're connected to the VPS via SSH, and `cd` into your remote project folder.  
You can run `ls` and `ls -a` to make sure all files have been properly transferred to the VPS.  

## Building our Docker image on the VPS

Since our app is very small, we won't need docker-compose.  
- ssh into your VPS
- cd into your remote project folder
- run `docker buildx build --platform linux/amd64 -t app_name:dev`

---
EOF
