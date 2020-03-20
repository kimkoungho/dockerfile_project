FROM centos:7

### local
RUN localedef -f UTF-8 -i ko_KR ko_KR.utf8 \
    && ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime
ENV LANG ko_KR.utf8
ENV LC_ALL ko_KR.utf8

### user
ARG user=deploy
ARG group=deploy
ARG uid=1000
ARG gid=1000
ARG USER_HOME=/home/deploy

# user home
RUN mkdir -p ${USER_HOME} \
    && chown ${uid}:${gid} ${USER_HOME} \
    && groupadd -g ${gid} ${group} \
    && useradd -d "${USER_HOME}" -u ${uid} -g ${gid} -m -s /bin/bash ${user}
# sudo user
RUN yum install -y sudo \
    && echo "${user} ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

### ssh
RUN yum install -y epel-release \
    && yum install -y openssh-server openssh-clients net-tools hostname \
    && yum clean all

# 비밀번호 인증을 비활성화
RUN sed -i -e 's~^PasswordAuthentication yes~PasswordAuthentication no~g' /etc/ssh/sshd_config

## create ssh key 
RUN ssh-keygen -q -b 1024 -N '' -t rsa -f /etc/ssh/ssh_host_rsa_key \
    && ssh-keygen -q -b 1024 -N '' -t dsa -f /etc/ssh/ssh_host_dsa_key \
    && ssh-keygen -q -b 521 -N '' -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key \
    && ssh-keygen -q -b 521 -N '' -t ed25519 -f /etc/ssh/ssh_host_ed25519_key

# sudo 명령어 경로 추가 - secure_path 주석처리 
RUN echo 'Defaults  env_keep += "PATH PROMPT_COMMAND"' >> /etc/sudoers

# ssh user
RUN mkdir -p ${USER_HOME}/.ssh \
    ## 생성된 키를 인증할 수 있는 키 목록에 추가
    && cat /etc/ssh/ssh_host_rsa_key >> ${USER_HOME}/.ssh/id_rsa \
    && cat /etc/ssh/ssh_host_rsa_key.pub >> ${USER_HOME}/.ssh/authorized_keys \
    && chown -R ${uid}:${gid} ${USER_HOME} \
    && chmod 600 ${USER_HOME}/.ssh/id_rsa \
    && chmod 644 ${USER_HOME}/.ssh/authorized_keys

### utility lib
RUN yum -y install bash-completion curl hostname vim-enhanced wget


EXPOSE 22
USER ${user}

CMD ["sudo", "/usr/sbin/sshd", "-D"]