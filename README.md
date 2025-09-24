测试 github actions 配合 docker 进行 CI/CD

并将文件使用 ssh 将 docker 容器 部署到服务器上


下面这是
.github/workflows/deploy.yaml文件

``` yaml

on:
  push:
    branches: [main]

permissions:
  contents: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: login to docker
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      - name: build and push
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: yang0709/yang-docker-demo:latest
      - name: Deploy to Remote Server
        uses: appleboy/ssh-action@v1.0.2
        with:
          host: ${{ secrets.SERVER_HOST }} # 服务器 IP 或域名
          username: ${{ secrets.SERVER_USER }} # 服务器用户名（如 ubuntu, root）
          key: ${{ secrets.SERVER_SSH_KEY }} # SSH 私钥（PKCS#8 格式）
          port: ${{ secrets.SERVER_PORT }} # SSH 端口，通常是 22
          script: |
            sudo docker --version 
            if [ "$(sudo docker ps -a | grep yang-docker-app)" ]; then
              sudo docker rm -f yang-docker-app
            fi
            sudo docker pull yang0709/yang-docker-demo:latest
            sudo docker run -d -p 90:80 --name 'yang-docker-app' yang0709/yang-docker-demo:latest

```



如何编写Dockerfile
``` Dockerfile
FROM node:22-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

