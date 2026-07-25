# Build stage
FROM node:22-alpine AS build
WORKDIR /app

COPY package.json package-lock.json* ./
RUN npm install

COPY . .
ARG VITE_API_URL=
ENV VITE_API_URL=$VITE_API_URL
RUN npm run build

# Runtime: tiny Node static server + /api proxy (rootless-friendly)
FROM node:22-alpine
WORKDIR /app
ENV NODE_ENV=production \
    PORT=80 \
    API_TARGET=http://backend:8000
COPY --from=build /app/dist ./dist
COPY server.mjs ./server.mjs
EXPOSE 80
CMD ["node", "server.mjs"]
