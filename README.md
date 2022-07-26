# volexpainer-vue2

## Project setup

```
npm install
```

### Compiles and hot-reloads for development

```
npm run serve
```

### Compiles and minifies for production

```
npm run build
```

### Lints and fixes files

```
npm run lint
```

## Docker setup

### Creating docker image

```shell
docker build -t volexpainter/dockerize-vue2 .
```

### Running container

```shell
docker run -it -p 8080:80 --rm --name dockerize-vue2-app volexpainter/dockerize-vue2
```
