FROM ubuntu:22.04
ENV DEBIAN_FRONTEND=noninteractive TZ=Asia/Ho_Chi_Minh

RUN apt-get update && apt-get install -y \
    openssh-server sudo locales tzdata vim curl net-tools dbus-x11 \
 && rm -rf /var/lib/apt/lists/*

RUN mkdir -p /var/run/sshd

RUN useradd -ms /bin/bash student \
 && echo "student:student" | chpasswd \
 && usermod -aG sudo student \
 && sed -i 's/#\?PasswordAuthentication .*/PasswordAuthentication yes/' /etc/ssh/sshd_config \
 && sed -i 's@session\s\+required\s\+pam_loginuid.so@session optional pam_loginuid.so@g' /etc/pam.d/sshd

EXPOSE 22
CMD ["/usr/sbin/sshd","-D"]