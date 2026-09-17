# ---- Etapa 1: build ----
FROM node:22-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci                 # ci, no install: respeta el lockfile
COPY . .
RUN npm run build

FROM build AS test
ENTRYPOINT ["npm", "run", "test:ci"]

# ---- Etapa 2: nginx sirve los estáticos ----
FROM nginx:alpine
COPY default.conf.template /etc/nginx/templates/default.conf.template
COPY --from=build /app/dist /usr/share/nginx/html
ENV BACKEND_URL=http://backend:8080 \
    DNS_RESOLVER=127.0.0.11
EXPOSE 80
