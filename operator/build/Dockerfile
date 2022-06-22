FROM quay.io/jonquark/amlen-prototype:latest
RUN yum install -y python3

# Update packages
RUN yum -y upgrade

ADD requirements.txt /tmp/
RUN pip3 install --upgrade -r /tmp/requirements.txt && \
    rm /tmp/requirements.txt

# scripts contains the scripts necessary for the readiness probe
ADD ./scripts/* /usr/share/amlen-server/bin/
RUN sed -i "/HA.AllowSingleNIC/d" /usr/share/amlen-server/config/server.cfg && \
    echo "HA.AllowSingleNIC = 1" >> /usr/share/amlen-server/config/server.cfg 

ENTRYPOINT [ "/usr/share/amlen-server/bin/startServer.sh" ]

