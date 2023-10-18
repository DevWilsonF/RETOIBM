FROM cp.icr.io/cp/appc/ace:12.0.8.0-r1@sha256:432986d77b781291b057ec5a01327cc9417df0d143982e00bc45302674923856

USER root
COPY *.bar /tmp
RUN export LICENSE=accept \
    && . /opt/ibm/ace-12/server/bin/mqsiprofile \
     && set -x && for FILE in /tmp/*.bar; do \
        echo "$FILE" >> /tmp/deploys && \
        ibmint package --compile-maps-and-schemas --input-bar-file "$FILE" --output-bar-file /tmp/temp.bar  2>&1 | tee -a /tmp/deploys && \
        ibmint deploy --input-bar-file /tmp/temp.bar --output-work-directory /home/aceuser/ace-server/ 2>&1 | tee -a /tmp/deploys; done \
    && ibmint optimize server --work-dir /home/aceuser/ace-server \
      && chmod -R ugo+rwx /home/aceuser/

USER 1001