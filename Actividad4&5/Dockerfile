FROM cp.icr.io/cp/appc/ace:12.0.8.0-r1@sha256:432986d77b781291b057ec5a01327cc9417df0d143982e00bc45302674923856

USER root

# Los paquetes MQ a instalar
ARG MQ_URL
ARG MQ_URL_USER
ARG MQ_URL_PASS
ARG MQ_PACKAGES="MQSeriesRuntime*.rpm MQSeriesJava*.rpm MQSeriesJRE*.rpm MQSeriesGSKit*.rpm MQSeriesClient*.rpm"
ARG INSTALL_JRE=0

ARG MQM_UID=888

COPY *.bar /tmp
COPY install-mq.sh /usr/local/bin/
COPY install-mq-client-prereqs.sh /usr/local/bin/

RUN export LICENSE=accept \
  && . /opt/ibm/ace-12/server/bin/mqsiprofile \
  && set -x && for FILE in /tmp/*.bar; do \
     echo "$FILE" >> /tmp/deploys && \
     ibmint package --compile-maps-and-schemas --input-bar-file "$FILE" --output-bar-file /tmp/temp.bar  2>&1 | tee -a /tmp/deploys && \
     ibmint deploy --input-bar-file /tmp/temp.bar --output-work-directory /home/aceuser/ace-server/ 2>&1 | tee -a /tmp/deploys; done \
  && ibmint optimize server --work-dir /home/aceuser/ace-server \
  # Instalar MQ. Para evitar un error de "archivo de texto ocupado" usamos Sleep antes de instalar.
  && chmod -R ugo+rwx /home/aceuser/ \
  && chmod u+x /usr/local/bin/install-*.sh \
  && sleep 1 \
  && install-mq-client-prereqs.sh $MQM_UID \
  && install-mq.sh $MQM_UID \
  && chown -R 1001:root /opt/mqm/*  \
  && chown 1001:root /usr/local/bin/*mq* \
  && mkdir -p /var/mqm/data \
  && chown -R 1001:root /var/mqm \
  && chmod -R 777 /var/mqm
  

ENV MQCERTLABL=aceclient

USER 1001
  


