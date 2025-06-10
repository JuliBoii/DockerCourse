# Use an existing Docker image as a base
FROM alpine

# Download & Install dependencies
RUN apk add --update redis

# Tell the image what to do when it starts
# as a container
CMD ["redis-server"]
