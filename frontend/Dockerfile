# Stage 1: Build React app
FROM node:18-alpine as build

WORKDIR /app

# Copy package files first
COPY package*.json ./

# Install dependencies with legacy-peer-deps
RUN npm install --legacy-peer-deps

# Fix ajv version issue explicitly
RUN npm install ajv@^6 ajv-keywords@^3 --legacy-peer-deps

# Copy source code
COPY . .

# Build React app
RUN npm run build

# Stage 2: Serve with Nginx
FROM nginx:alpine

# Copy build output to nginx html folder
COPY --from=build /app/build /usr/share/nginx/html

# Expose default nginx port
EXPOSE 80

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
