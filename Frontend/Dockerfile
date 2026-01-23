# ---------- Stage 1: Build ----------
FROM node:22-alpine AS builder
WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

COPY . .

# Build the React app
ARG REACT_APP_BACKEND_URL=http://localhost:8082
ENV REACT_APP_BACKEND_URL=$REACT_APP_BACKEND_URL
RUN npm run build

# ---------- Stage 2: Production ----------
FROM node:22-alpine
WORKDIR /app

# Install serve to run the built React app
RUN npm install -g serve

# Copy built files from builder stage
COPY --from=builder /app/build ./build

# Expose port
EXPOSE 3000

# Start the production server
CMD ["serve", "-s", "build", "-l", "3000"]
