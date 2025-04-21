FROM jenkins/inbound-agent
USER root
RUN mkdir -p /mnt/ebs-logs
RUN apt-get update && apt-get install -y docker
RUN apt-get install -y docker-compose
RUN apt-get install -y python3
RUN apt-get install -y python3-venv
RUN apt-get install -y python3-pip
RUN groupadd -g 992 docker2
RUN usermod -aG docker2 jenkins
USER jenkins
EXPOSE 8080 50000

