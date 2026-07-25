# Build stage
FROM node:22-alpine AS build
WORKDIR /app

COPY package.json package-lock.json* ./
RUN npm install

COPY . .
# Empty VITE_API_URL => browser calls same origin (/api/...), proxied by nginx
ARG VITE_API_URL=
ENV VITE_API_URL=$VITE_API_URL
RUN npm run build

# Runtime stage — custom nginx.conf is the main config (rootless-friendly logs/temp)
FROM nginx:1.27-alpine
RUN rm -f /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/nginx.conf
COPY --from=build /app/dist /usr/share/nginx/html
# Ensure nginx can write temp paths under /tmp in rootless environments
RUN mkdir -p /tmp/client_temp /tmp/proxy_temp /tmp/fastcgi_temp /tmp/uwsgi_temp /tmp/scgi_temp \
    && chmod -R 777 /tmp
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
