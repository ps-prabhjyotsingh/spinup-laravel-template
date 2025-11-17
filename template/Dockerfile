############################################
# Base Image
############################################

# Learn more about the Server Side Up PHP Docker Images at:
# https://serversideup.net/open-source/docker-php/
FROM serversideup/php:8.3-fpm-nginx-alpine AS base

## Uncomment if you need to install additional PHP extensions
# USER root
# RUN install-php-extensions bcmath gd

############################################
# Development Image
############################################
FROM base AS development

# We can pass USER_ID and GROUP_ID as build arguments
# to ensure the www-data user has the same UID and GID
# as the user running Docker.
ARG USER_ID
ARG GROUP_ID

# Switch to root so we can set the user ID and group ID
USER root

# Set the user ID and group ID for www-data
RUN docker-php-serversideup-set-id www-data $USER_ID:$GROUP_ID  && \
    docker-php-serversideup-set-file-permissions --owner $USER_ID:$GROUP_ID --service nginx

# Drop privileges back to www-data    
USER www-data

############################################
# CI image
############################################
FROM base AS ci

# Sometimes CI images need to run as root
# so we set the ROOT user and configure
# the PHP-FPM pool to run as www-data
USER root
RUN echo "user = www-data" >> /usr/local/etc/php-fpm.d/docker-php-serversideup-pool.conf && \
    echo "group = www-data" >> /usr/local/etc/php-fpm.d/docker-php-serversideup-pool.conf

############################################
# Node.js Assets Builder (Optional)
############################################
# Uncomment this stage if you want to build assets during Docker build
# Otherwise, build assets in CI before building the Docker image

# FROM node:20-alpine AS assets
# WORKDIR /app
# COPY package*.json yarn.lock* ./
# RUN corepack enable && \
#     if [ -f yarn.lock ]; then yarn install --frozen-lockfile; \
#     else npm ci; fi
# COPY . .
# RUN if [ -f yarn.lock ]; then yarn build; \
#     else npm run build; fi

############################################
# Production Image
############################################
FROM base AS deploy
COPY --chown=www-data:www-data . /var/www/html

# If using the assets stage above, copy built assets
# COPY --from=assets --chown=www-data:www-data /app/public/build /var/www/html/public/build

# Create the SQLite directory and set the owner to www-data (remove this if you're not using SQLite)
RUN mkdir -p /var/www/html/.infrastructure/volume_data/sqlite/ && \
    chown -R www-data:www-data /var/www/html/.infrastructure/volume_data/sqlite/

USER www-data
