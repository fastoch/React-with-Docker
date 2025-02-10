# React-with-Docker

React + Docker Tutorial: https://www.youtube.com/watch?v=zi6cdUyZ_uQ&t=4s  

---

# Why use Docker?

Docker is one of the most critical skills to have in the current job market (2025).  
It gives you a massive advantage over developers who don't have this skill.  

# What we'll learn

- How to dockerize a React application using **Dockerfile** along with **Docker Compose**
- How to deploy this dockerized app on a private server (no Netlify, no Vercel)
- How the apps are actually deployed in the real world

# What is Docker?

Docker allows you to separate your applications from your infrastructure so you can deliver software quickly.  
By taking advantage of Docker's methodologies, you can significantly reduce the delay between writing code and running it in production.  

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
```

## Explanation of our dockerfile

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

And finally, ... how to start our App when we start the docker image ...   


Note that `RUN` executes commands during the Docker image build process. Each `RUN` instruction adds a new layer to the Docker image.  
It is used to install software packages, set environment variables, and perform configurations needed for the application.  

While `CMD` specifies the default command to run when a container is started from the Docker image.

---

## Important note

When we run `npm install`, this creates a `node_modules` folder.  
This folder contains all the external modules and dependencies required by your Node.js project.  
You don't want to copy this folder to your container, which is why you need to create a `.dockerignore` file at the root of your project.  
This is the same logic as including the `node_modules` folder inside our `.gitignore` file.  

Inside the `.dockerignore` file, write the following and save:
```.dockerignore
node_modules
```

---





@8/24
---
EOF
