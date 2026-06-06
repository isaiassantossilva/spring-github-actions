# syntax=docker/dockerfile:1

# ---- Stage 1: build ----
FROM node:22-alpine AS build
WORKDIR /app

# Instala dependências usando o lockfile (cache eficiente)
COPY package.json package-lock.json ./
RUN npm ci

# Copia o restante do código e gera o build de produção
COPY . .
RUN npm run build

# ---- Stage 2: runtime ----
FROM nginx:1.27-alpine AS runtime

# Configuração do Nginx com suporte a SPA (fallback para index.html)
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Artefatos estáticos gerados pelo Vite
COPY --from=build /app/dist /usr/share/nginx/html

EXPOSE 80

# Healthcheck simples
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget -qO- http://localhost/ >/dev/null 2>&1 || exit 1

CMD ["nginx", "-g", "daemon off;"]
