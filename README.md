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

## Our first Dockerfile

In VS Code, create a file at the root of your React project and name it `Dockerfile`.  
Now we need to write some commands in this Dockerfile so that Docker knows how it is supposed to run our application: 
```dockerfile
FROM node:20-alpine

```


@5/24
---
EOF
