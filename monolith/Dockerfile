FROM ubuntu:16.04

#Update ubuntu & install mongo, ruby, ess., git
RUN apt-get update
RUN apt-get install -y mongodb-server ruby-full ruby-dev build-essential git
RUN gem install bundler

#Clone repo into container
RUN git clone https://github.com/Artemmkin/reddit.git

#Copy config files from localhost
COPY mongod.conf /etc/mongod.conf
COPY db_config /reddit/db_config
COPY start.sh /start.sh

#install dependencies
RUN cd /reddit && bundle install
RUN chmod 0777 /start.sh

#start service
CMD ["/start.sh"]
