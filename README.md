<p align="center">
  :whale:
  <br /><br />
  A simple app for teaching the basics of Docker.
</p>

---

#### Basic Image

```
FROM node:16-alpine
WORKDIR /app

COPY . .

RUN yarn install

CMD ["node", "index.js"]
```

#### Building & Running

```
$ docker build -t docker-demo .
$ docker run --rm -it -e "PORT=3000" -p 3000:3000 docker-demo
```

#### Caching Install

Only copy what's absolutely necessary for each step to leverage Docker's [layer caching](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#leverage-build-cache).

```
FROM node:16-alpine
WORKDIR /app

COPY package.json .
RUN yarn install

COPY . .

CMD ["node", "index.js"]
```

#### Building Images in the Public Folder

```
FROM node:16-alpine
WORKDIR /app

RUN apk add pngquant imagemagick

COPY public ./public
COPY scripts/prepare-images ./scripts/prepare-images
RUN sh ./scripts/prepare-images

COPY package.json .
RUN yarn install

COPY . .

CMD ["node", "index.js"]
```

#### Multi-Stage 

Ensure your final image is super clean by using [multi-stage builds](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#use-multi-stage-builds).

```
# =======
# BASE
# =======
FROM node:16-alpine as base



# ========
# BUILD
# ========
FROM base as build
WORKDIR /app

RUN apk add pngquant imagemagick

COPY public ./public
COPY scripts/prepare-images ./scripts/prepare-images
RUN sh ./scripts/prepare-images

COPY package.json .
RUN yarn install

COPY . .



# ========
# FINAL
# ========
FROM base
WORKDIR /app

COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/index.js .

CMD ["node", "index.js"]
```
