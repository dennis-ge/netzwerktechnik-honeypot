ARG ELK_VERSION

FROM docker.elastic.co/beats/filebeat:${ELK_VERSION}

COPY config/filebeat.yml /usr/share/filebeat/filebeat.yml
# must run as root to access /var/lib/docker and /var/run/docker.sock
USER root
RUN mkdir /usr/share/filebeat/dockerlogs
RUN chown -R root /usr/share/filebeat/
RUN chmod -R go-w /usr/share/filebeat/