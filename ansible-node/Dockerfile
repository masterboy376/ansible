# ansible-node/Dockerfile
FROM ubuntu:latest

# Install necessary packages
RUN apt-get update && \
    apt-get install -y openssh-server python3 && \
    apt-get clean

# Start SSH service
RUN mkdir /var/run/sshd
RUN echo 'root:password' | chpasswd
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed -i 's/#Port 22/Port 2222/' /etc/ssh/sshd_config

EXPOSE 2222
CMD ["/usr/sbin/sshd", "-D"]
