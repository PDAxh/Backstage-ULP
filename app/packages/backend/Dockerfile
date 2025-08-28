# ---- Build stage ----
    FROM node:20-bookworm AS build

    WORKDIR /app
    COPY . .
    
    # Backstage uses yarn workspaces
    RUN corepack enable \
     && yarn install --frozen-lockfile \
     && yarn tsc \
     && yarn build:backend
    
    # ---- Runtime stage ----
    FROM node:20-bookworm
    
    # For sharp & other deps
    RUN apt-get update && apt-get install -y python3 make g++ \
     && rm -rf /var/lib/apt/lists/*
    
    ENV NODE_ENV=production
    WORKDIR /app
    
    # Only copy the built backend bundle and required packages
    COPY --from=build /app/packages/backend/dist packages/backend/dist
    COPY --from=build /app/packages/backend/package.json packages/backend/package.json
    COPY --from=build /app/package.json /app/yarn.lock ./
    
    # Install only production deps for the backend
    RUN corepack enable \
     && yarn workspaces focus --production @backstage/backend
    
    # Backstage listens on 7007 by default
    EXPOSE 7007
    
    # Backstage reads config from app-config.yaml + app-config.production.yaml
    CMD ["node", "packages/backend", "--config", "app-config.yaml", "--config", "app-config.production.yaml"]
    