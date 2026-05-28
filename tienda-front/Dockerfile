# =============================================================================
# STAGE 1: BUILD ANGULAR
# Compilamos la app Angular con Node.js.
# Angular 21 requiere Node 20+.
# =============================================================================
FROM node:20-alpine AS builder

WORKDIR /app

# Copiamos package.json y package-lock.json PRIMERO (caché de capas Docker)
# Si no cambian las dependencias, Docker reutiliza esta capa.
COPY package.json package-lock.json ./

# Instalamos dependencias (--ci es más estricto y reproducible que npm install)
RUN npm ci

# Copiamos el resto del código fuente
COPY . .

# Build de producción
# Angular 21 usa el builder "@angular/build:application"
# El output por defecto va a: dist/tienda-front/browser/
RUN npm run build -- --configuration production

# =============================================================================
# STAGE 2: NGINX
# Imagen mínima de Nginx para servir los archivos estáticos.
# También hace de proxy reverso hacia el backend (/api/* → backend service).
# Resultado: imagen ~25MB
# =============================================================================
FROM nginx:1.25-alpine

# Eliminamos la configuración default de Nginx
RUN rm /etc/nginx/conf.d/default.conf

# Copiamos nuestra configuración personalizada
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copiamos los archivos compilados de Angular al directorio de Nginx
# Angular 21 con @angular/build:application genera en dist/tienda-front/browser/
COPY --from=builder /tienda-front/dist/tienda-front/browser /usr/share/nginx/html

# Puerto 80 (HTTP dentro del cluster — el SSL lo maneja el Ingress de GCP)
EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
