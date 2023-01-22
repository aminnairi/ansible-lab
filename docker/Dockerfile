FROM debian:latest

ARG SSH_KEY_PUBLIC

RUN apt update \
  && apt install --yes openssh-server python3 \
  && service ssh start \
  && mkdir -p /root/.ssh/ \
  && echo "${SSH_KEY_PUBLIC}" > /root/.ssh/authorized_keys \
  && yes | ssh-keygen -t ed25519 -N ""

EXPOSE 22

CMD ["/usr/sbin/sshd","-D"]
