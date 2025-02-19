# SPDX-FileCopyrightText: 2021 The HedgeDoc developers (see AUTHORS file)
#
# SPDX-License-Identifier: CC-BY-SA-4.0

# BASE
FROM docker.io/node:20-alpine@sha256:a329b146bcc99a36caa73056e60714d0911ca5c229ade3eb27e9283dc78c9eb6 AS base
RUN apk add --no-cache tini
ENTRYPOINT ["tini", "--"]

# BUILD
FROM base AS builder

RUN apk add --no-cache libc6-compat
ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1

ENV YARN_CACHE_FOLDER=/tmp/.yarn
USER node
WORKDIR /usr/src/app

COPY --chown=node . .
RUN --mount=type=cache,sharing=locked,uid=1000,gid=1000,target=/tmp/.yarn \
    yarn install --immutable

ARG TURBO_TEAM
ARG TURBO_API
ARG TURBO_TOKEN

RUN yarn build --filter=frontend --no-cache --no-daemon

# RUNNER
FROM base
ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1

LABEL org.opencontainers.image.title='HedgeDoc production frontend image'
LABEL org.opencontainers.image.url='https://hedgedoc.org'
LABEL org.opencontainers.image.source='https://github.com/hedgedoc/hedgedoc'
LABEL org.opencontainers.image.documentation='https://github.com/hedgedoc/hedgedoc/blob/develop/docs/content/dev/docker.md'
LABEL org.opencontainers.image.licenses='AGPL-3.0'

WORKDIR /usr/src/app

COPY --from=builder --chown=node:node /usr/src/app/frontend/dist ./

USER node

ENV PORT 3001
EXPOSE 3001/tcp

WORKDIR /usr/src/app/frontend
CMD ["node", "server.js"]
