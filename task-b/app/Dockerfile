# syntax=docker/dockerfile:1.7

# ---- Build stage ----
FROM node:20-alpine AS build
WORKDIR /app

# Install only production deps (reproducible)
COPY package*.json ./
RUN npm ci --omit=dev

# Copy source
COPY . .

# ---- Runtime stage (minimal) ----
FROM alpine:3.23 AS runtime
WORKDIR /app
ENV NODE_ENV=production

# Install only what's needed to run node (no npm)
RUN apk add --no-cache libstdc++ ca-certificates \
  && addgroup -S nodejs \
  && adduser -S nodeuser -G nodejs

# Copy Node runtime binaries + libs from build image
COPY --from=build /usr/local/bin/node /usr/local/bin/node
COPY --from=build /usr/local/lib/node_modules /usr/local/lib/node_modules

# Make node available without pulling npm tooling into PATH
RUN ln -s /usr/local/lib/node_modules /usr/local/lib/node_modules \
  && ln -s /usr/local/bin/node /usr/local/bin/node

# Copy app + installed node_modules from build stage
COPY --from=build /app /app

USER nodeuser
EXPOSE 3000
CMD ["node", "index.js"]