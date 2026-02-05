# ---- Build stage ----
FROM node:20-alpine AS build
WORKDIR /app

COPY package*.json ./
RUN npm ci --omit=dev
COPY . .

# ---- Runtime stage (minimal) ----
FROM alpine:3.23 AS runtime
WORKDIR /app
ENV NODE_ENV=production

RUN apk add --no-cache libstdc++ ca-certificates \
  && addgroup -S nodejs \
  && adduser -S nodeuser -G nodejs

# Copy node runtime
COPY --from=build /usr/local/bin/node /usr/local/bin/node

# Copy app (includes node_modules from build stage)
COPY --from=build /app /app

USER nodeuser
EXPOSE 3000
CMD ["node", "index.js"]