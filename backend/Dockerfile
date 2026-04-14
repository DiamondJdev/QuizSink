# Multi-stage Dockerfile for NestJS Backend

# ============================================
# Base Stage - Common dependencies
# ============================================
FROM node:20-alpine AS base

# Set working directory
WORKDIR /app

# Install build dependencies and dumb-init
RUN apk add --no-cache dumb-init python3 make g++

# Copy package files
COPY package*.json ./

# ============================================
# Development Stage
# ============================================
FROM base AS development

# Install all dependencies (including devDependencies)
RUN npm i

# Copy source code
COPY . .

# Expose port
EXPOSE 5200

# Start in development mode with hot reload
CMD ["dumb-init", "npm", "run", "start:dev"]

# ============================================
# Builder Stage - Compile TypeScript
# ============================================
FROM base AS builder

# Install all dependencies (needed for build)
RUN npm i

# Copy source code
COPY . .

# Build the application
RUN npm run build

# ============================================
# Production Stage
# ============================================
FROM node:20-alpine AS production

# Set working directory
WORKDIR /app

# Install dumb-init for proper signal handling
RUN apk add --no-cache dumb-init

# Set production environment
ENV NODE_ENV=production

# Copy package files
COPY package*.json ./

# Copy production dependencies and built output from builder
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist

# Copy .env.example as reference (actual .env should be provided at runtime)
COPY src/.env.example ./src/.env.example

# Create data directory with proper permissions
RUN mkdir -p /app/data && chown -R node:node /app/data

# Use the non-root node user
USER node

# Expose port
EXPOSE 5200

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD node -e "require('http').get('http://localhost:5200', (r) => {process.exit(r.statusCode === 404 ? 0 : 1)})"

# Start the application
CMD ["dumb-init", "node", "dist/main"]
