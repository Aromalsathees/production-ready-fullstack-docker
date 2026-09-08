# Stage 1: Build
FROM node:20 AS build

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

# Build arguments for environment variables
ARG VITE_SERVER_BASE_URL

ENV VITE_SERVER_BASE_URL=$VITE_SERVER_BASE_URL

RUN npm run build


# Stage 2: Nginx
FROM nginx:alpine

# Copy React build files from Stage 1
COPY --from=build /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]