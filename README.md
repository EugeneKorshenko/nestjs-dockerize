# NestJS Dockerize
Two stage Dockerfile for NestJS application with TypeORM and i18n
```
# Stage: Build
FROM node:10.15 AS builder
WORKDIR /usr/src/backend
COPY . .
# Build NestJS Application and copy locale files
RUN npm install && npm run build && cp -a src/i18n/. dist/i18n/

# Stage: Deploy
FROM node:10.15-alpine
WORKDIR /root/
COPY package*.json ./
RUN apk --update add --no-cache --virtual .build-deps \
    gcc \
    g++ \
    make \
    python \
    && npm ci --only=production \
    && apk del --no-cache .build-deps

# Bundle app source
COPY --from=builder /usr/src/backend/dist .
ENV TYPEORM_ENTITIES **/*.entity.js

EXPOSE 3001
CMD [ "node", "main.js" ]
```
