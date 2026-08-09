# Use Node.js 18 on Alpine Linux as base
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files first (layer caching optimization)
COPY package.json package-lock.json* ./

# Install dependencies
RUN npm install

# Copy application code
COPY . .

# Document the port
EXPOSE 3000

# Start the application
CMD ["node", "app.js"]
