# Use a smaller, more recent base image if possible, e.g., Debian or Alpine
# Ubuntu 22.04 is fine as well.
FROM ubuntu:22.04

# Set environment variables (optional, but good practice)
ENV DEBIAN_FRONTEND=noninteractive

# Install build tools and dependencies
RUN apt-get update && \
    apt-get install -y build-essential gcc g++ make manpages-dev # manpages-dev is useful for understanding syscalls, not strictly necessary for building

# Set the working directory inside the container
WORKDIR /app

# Copy only necessary files for building first, to leverage Docker's build cache
COPY Makefile proxy_parse.h proxy_parse.c proxy_server_with_cache.c ./

# Build the project
# Using 'make' will automatically use the Makefile
RUN make all

# Clean up build artifacts if you want a smaller image (optional)
# RUN make clean

# Expose the proxy port
EXPOSE 8080

# Default command to run the proxy on port 8080
# Use exec form to properly handle signals (e.g., Ctrl+C)
CMD ["./proxy", "8080"]