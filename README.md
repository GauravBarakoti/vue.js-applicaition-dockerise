# Single-Page Application in Vue.js

SPA, some applications require rich interactivity, deep session depth, and non-trivial stateful logic on the frontend. The best way to build such applications is to use an architecture where Vue not only controls the entire page, but also handles data updates and navigation without having to reload the page. This type of application is typically referred to as a Single-Page Application (SPA).

In short, a single-page application is an app that doesn’t need to reload the page during its use and works within a browser.

![Alt text](image.png)

In my case, I have a sample Single-Page Application of Vue.js.

Our goal here is to Dockerise this application, push the Image to Amazon ECR and then deploy it using ECS Fargate attach ELB and ASG.

I have tried using a single-stage Dockerfile but was getting the 404 error when launching a new container because the application was not running in the container because there was nothing that served the vue.js application.

Then, I used a multi-stage Dockerfile which worked fine here is the file: (I'll explain this Dockerfile shortly)

```
FROM node:lts-alpine as build-stage
WORKDIR /app
COPY . .
RUN npm cache clean --force
RUN npm install --legacy-peer-deps
RUN npm run build

FROM nginx:stable-alpine as production-stage
COPY --from=build-stage /app/dist /usr/share/nginx/html
COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

And, this also reduces the size of the image drastically.

Use the following commands to verify if it's working or not:

```
sudo docker build -t vueimagejs .
```

Now, to run a container:

```
sudo docker container run -d -p 8081:80 vueimagejs
```

Verify by 
```
curl localhost:8081
```

Or just do `localhost:8081` on the browser.

Then Create a repo. in `ECR` for that configure `AWS-CLI`.

```
sudo apt install awscli
```

then,

```
aws configure
```
provide the user cred which has ECS access for creating repo. in ECR.

```
aws ecr create-repository --repository-name <repo_name> --region <region_name>
```

Go to the ECR console, Select your Repository and click `view push commands`.

Go to the `ECS console` and First make the task definition.

Create new task definition -> Provide Name -> Select AWS Fargate -> OS as Linux
Provide CPUs in my case I choose 1vCPU and Memory as 3GB.
Leave Task role and Task Execution Role as default {if you need to login to your Fargate container you can assign the appropriate Task Execution Role }

Then provide the container details
Name, Image URI from the ECR, port mapping details and CPU, GPU, Memory hard limit, Memory soft limit.

Then You can provide the Environment Variables.

Leave the rest for Default and hit Create.

Go to `Clusters`:

Provide your Cluster Name.
Select Infrastructure as Fargate.
Then Create.

Now, Create a `Elastic Load Balancer`

Create a `Service` with your desired configuration.
Select the Load Balancer you just created. And create the Target Group accordingly.
And, create the service.

Now, If you hit DNS of Load Balancer your website will be visible although when you hit something like about or anything on your page and reload your page you'll get a `404 error`.

So you might think what can be the reason behind this. This is because of the single-page application, you are missing that SPA is not server side rendering. At least for the majority. So, when you access /anything your web server won't redirect it to index.html. However, if you click on any vuejs router link, it will work due to the fact that the javascript got in action, but not the server side.

So to resolve this we will use the location block in the nginx configuration file.

Here is the configuration file:

```
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

`try_files` tests for the existence of the file in the local file system and may rewrite the URL, if it does exist, it only remembers it - then continues processing the rest of the location block.

This all details are already added to the `Dockerfile`.

Dockerfile first build your vue.js application and copy the `dist` folder to another nginx image and then also copy the nginx conf file in the `/etc/nginx/conf.d/` folder.

In this way, we can achieve our goal.

Feel free to adapt this documentation to your specific requirements and Flask application configuration.


# **Thank You**

I hope you find it useful. If you have any doubt in any of the step then feel free to contact me.
If you find any issue in it then let me know.

<!-- [![Build Status](https://img.icons8.com/color/452/linkedin.png)](https://www.linkedin.com/in/gaurav-barakoti-27002223b) -->


<table>
  <tr>
    <th><a href="https://www.linkedin.com/in/gaurav-barakoti-27002223b" target="_blank"><img src="https://img.icons8.com/color/452/linkedin.png" alt="linkedin" width="30"/><a/></th>
    <th><a href="mailto:bestgaurav1234@gmail.com" target="_blank"><img src="https://img.icons8.com/color/344/gmail-new.png" alt="Mail" width="30"/><a/>
</th>
  </tr>
</table>




