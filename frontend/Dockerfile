# ---------- Build Stage ----------
FROM node:20-alpine AS builder

WORKDIR /app

# Copy dependency files
COPY package*.json ./

# Install dependencies
RUN npm ci

# Copy project files
COPY . .

# Build production files
RUN npm run build


# ---------- Production Stage ----------
FROM nginx:alpine

# Copy nginx configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copy built files from build stage
COPY --from=builder /app/dist /usr/share/nginx/html

# Expose container port
EXPOSE 6000

# Start nginx server
CMD ["nginx", "-g", "daemon off;"]
