FROM jasonish/suricata

RUN dnf -y install python3-pip

RUN pip3 install suricata-language-server

COPY entrypoint.sh /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
