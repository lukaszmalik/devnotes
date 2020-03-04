FROM python:slim

RUN apt-get -y update

RUN apt-get -y install curl
RUN apt-get -y install jq

RUN pip install awscli --upgrade

RUN curl -sL https://deb.nodesource.com/setup_10.x | bash - && apt-get -y install nodejs
RUN npm install aws-es-kibana@1.0.8 -g

#https://github.com/santthosh/aws-es-kibana/issues/50
RUN mkdir /root/.aws && touch /root/.aws/credentials

COPY mmgaws usr/bin

CMD eval $(mmgaws $ACCOUNT DevTeam-Loktar) && aws-es-kibana -b 0.0.0.0 $TARGET_ES