# ansible-server/Dockerfile
FROM ubuntu:latest

# Install necessary packages
RUN apt-get update && \
    apt-get install -y ansible openssh-client sshpass vim && \
    apt-get clean

# Create a directory for Ansible configuration
RUN mkdir -p /etc/ansible

# Copy hosts file
COPY hosts /etc/ansible/hosts

# Copy playbook file
COPY playbook.yml /etc/ansible/playbook.yml

CMD ["tail", "-f", "/dev/null"]
