# ---- Build stage ----
FROM node:23-alpine3.21 AS build

WORKDIR /app

# Install deps first (better caching)
COPY package*.json ./
RUN npm ci --omit=dev

# Copy source
COPY . .

# ---- Runtime stage ----
FROM node:23-alpine3.21 AS runtime

ENV NODE_ENV=production
WORKDIR /app

# Create a non-root user (standard hardening)
RUN addgroup -S nodejs && adduser -S nodeuser -G nodejs

# Copy only what we need from build stage
COPY --from=build /app /app

# Drop privileges
USER nodeuser

# App listens on 3000 (matches your index.js default)
EXPOSE 3000

# Safer default for containers
CMD ["node", "index.js"]