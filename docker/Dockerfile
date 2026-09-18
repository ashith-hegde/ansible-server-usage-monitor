FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y openssh-server sudo && \
    rm -rf /var/lib/apt/lists/*

RUN useradd -m -s /bin/bash ansible && \
    echo "ansible:ansible" | chpasswd && \
    usermod -aG sudo ansible

RUN mkdir -p /var/run/sshd /data && \
    chmod 755 /data

RUN sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]
