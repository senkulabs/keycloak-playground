ARG KC_VERSION=26.3.3

FROM maven:3.9.9-eclipse-temurin-21 AS keycloak-md5

WORKDIR /app

COPY ./providers/keycloak-md5 .

RUN mvn -B package --file pom.xml

FROM quay.io/keycloak/keycloak:${KC_VERSION} AS builder

# Redeclare in order can be used inside the FROM
ARG KC_VERSION

# Enable health and metrics support
ENV KC_HEALTH_ENABLED=true
ENV KC_METRICS_ENABLED=true

# Configure a database vendor
ENV KC_DB=postgres

# Enable certain Keycloak features
ENV KC_FEATURES=token-exchange,admin-fine-grained-authz:v1,persistent-user-sessions

# Add provider JAR files to providers directory
COPY --chown=keycloak:keycloak --from=keycloak-md5 /app/target/keycloak-md5.jar /opt/keycloak/providers/keycloak-md5.jar

# Install custom plugins 
RUN touch -m --date=@1743465600 /opt/keycloak/providers/*
RUN /opt/keycloak/bin/kc.sh build

#
# NPM Dependencies
#

FROM node:22-alpine AS npm

WORKDIR /app

COPY ./hazel .

RUN npm ci

RUN npm run build

FROM quay.io/keycloak/keycloak:${KC_VERSION}

WORKDIR /opt/keycloak

COPY --from=builder /opt/keycloak /opt/keycloak
COPY --chown=keycloak:keycloak --from=npm /app/themes /opt/keycloak/themes

# Remove devMode property from all theme.properties files
RUN find /opt/keycloak/themes/*/login -name "theme.properties" -exec sed -i '/devMode=/d' {} \;

ENTRYPOINT ["/opt/keycloak/bin/kc.sh"]
