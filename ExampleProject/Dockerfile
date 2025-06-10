# Specifiy a base image
FROM node:14-alpine

# Install some dependencies
COPY ./index.js ./
COPY ./package.json ./

RUN npm install

# Default Command
CMD ["npm", "start"]