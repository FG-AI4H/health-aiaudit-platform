FROM node:8.11.2 as node

ARG NODE_ENV
WORKDIR /code

# Add dependencies
ADD ./package.json /code
ADD ./bower.json /code
ADD ./gulpfile.js /code
ADD ./.eslintrc /code
ADD ./karma.conf.js /code

# Install Prerequisites
RUN npm install -g bower gulp
RUN npm install phantomjs-prebuilt
RUN npm link gulp
RUN npm cache clean -f
RUN npm install
RUN npm install -g qs
RUN bower install --allow-root
ADD frontend /code/frontend

RUN gulp production

FROM nginx:1.13-alpine

# Adding NODE_ENV here as well since this is a multistage build
ARG NODE_ENV

#original
COPY docker/prod/nodejs/nginx_production.conf /etc/nginx/conf.d/default.conf
COPY --from=node /code /code

COPY ssl /etc/ssl



