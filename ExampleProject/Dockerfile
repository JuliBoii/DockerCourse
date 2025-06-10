# Specifiy a base image
FROM node:14-alpine

WORKDIR /usr/workspace

# Install some dependencies
COPY ./package.json ./
RUN npm install

COPY ./index.js ./

# Default Command
CMD ["npm", "start"]