# Ansible Docker Setup

This README provides instructions on setting up an Ansible environment using Docker containers. We'll create two Docker containers: one as the Ansible server and another as the Ansible node.

## Requirements

- Docker installed on your machine
- Basic knowledge of Docker and Ansible

## Step-by-Step Setup

### 1. Create Dockerfiles

**Dockerfile for Ansible Server:**

```Dockerfile
# ansible-server/Dockerfile
FROM ubuntu:latest

# Install necessary packages
RUN apt-get update && \
    apt-get install -y ansible openssh-client sshpass && \
    apt-get clean

# Create a directory for Ansible configuration
RUN mkdir -p /etc/ansible

# Copy hosts file
COPY hosts /etc/ansible/hosts

# Copy playbook file
COPY playbook.yml /etc/ansible/playbook.yml

CMD ["tail", "-f", "/dev/null"]
```

**Dockerfile for Ansible Node:**

```Dockerfile
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
```

### 2. Create Additional Files

Create a `hosts` file in the `ansible-server` directory:

```ini
# ansible-server/hosts
[node]
ansible-node ansible_host=ansible-node ansible_user=root ansible_ssh_private_key_file=/root/.ssh/id_rsa ansible_port=2222

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

Create a `playbook.yml` file in the `ansible-server` directory:

```yaml
---
# ansible-server/playbook.yml
- hosts: node
  become: true
  vars:
    ansible_become_pass: password
  tasks:
    - name: Ensure Python is installed
      apt:
        name: python3
        state: present
    - name: Install nginx
      apt:
        name: nginx
        state: present
      notify:
        - start nginx

  handlers:
    - name: start nginx
      service:
        name: nginx
        state: started
```

### 3. Build Docker Images

Navigate to each directory and build the Docker images:

```sh
cd ansible-server
docker build -t ansible-server .

cd ../ansible-node
docker build -t ansible-node .
```

### 4. Create a Docker Network

Create a Docker network that both containers will use:

```sh
docker network create ansible-net
```

### 5. Run Docker Containers

Run the Ansible server and node containers on the created network:

```sh
docker run -d --name ansible-server --network ansible-net ansible-server
docker run -d --name ansible-node --network ansible-net ansible-node
```

### 6. Set Up SSH Keys

Generate and copy the SSH keys:

```sh
docker exec -it ansible-server bash
ssh-keygen -t rsa -N "" -f /root/.ssh/id_rsa
sshpass -p 'password' ssh-copy-id -i /root/.ssh/id_rsa.pub -p 2222 root@ansible-node
```

### 7. Run the Ansible Playbook

Run the playbook:

```sh
docker exec -it ansible-server ansible-playbook /etc/ansible/playbook.yml
```

## Troubleshooting

### Connection Issues

**Error: Hostname Resolution Failed**

Ensure both containers are on the same network:

```sh
docker network inspect ansible-net
```

Update the `hosts` file to use `ansible-node` hostname.

**Error: SSH Connection Refused**

Verify the SSH service on the node:

```sh
docker exec -it ansible-node bash
service ssh status
```

Ensure SSH is listening on port 2222:

```sh
grep 'Port 2222' /etc/ssh/sshd_config
service ssh restart
```

**Error: Permission Denied**

Ensure the SSH key is correctly distributed:

```sh
docker exec -it ansible-node bash
chmod 700 /root/.ssh
chmod 600 /root/.ssh/authorized_keys
```

Manually test SSH connection:

```sh
docker exec -it ansible-server bash
ssh -i /root/.ssh/id_rsa -p 2222 root@ansible-node
```

## Conclusion

By following these steps, you should be able to set up and manage an Ansible environment using Docker containers effectively. If you encounter any issues, refer to the troubleshooting section for common errors and their solutions. Also, we can use Ansible docker module to better manage containers.

---
