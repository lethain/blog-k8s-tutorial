FROM debian:jessie-backports

# packages
RUN apt-key update
RUN apt-get update
RUN apt-get remove apt-listchanges
RUN apt-get install certbot -y -t jessie-backports
RUN apt-get install cron -y
RUN apt-get install nginx -y
RUN apt-get install curl -y
RUN openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
RUN apt-get install libyaml-dev -y
RUN apt-get install python-dev -y
RUN apt-get install python-pip -y
RUN pip install gsutil

# logs and stuff
RUN touch /var/log/cron.log
RUN mkdir -p /var/log/nginx
RUN touch /var/log/nginx/access.log
RUN touch /var/log/nginx/error.log
RUN ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log
RUN rm /etc/cron.d/certbot

# credentials
COPY service.pem /
COPY boto /root/.boto
COPY credstore /root/.gsutil/credstore

# copy old SSL certs
COPY ./ssl/fullchain.pem /etc/letsencrypt/live/willarson.com/fullchain.pem
COPY ./ssl/privkey.pem /etc/letsencrypt/live/willarson.com/privkey.pem

ADD cron /etc/cron.d/certbot-cron
ADD ./nginx /etc/nginx/sites-enabled/
RUN chmod 0644 /etc/cron.d/certbot-cron

ENTRYPOINT cron && nginx -g "daemon off;"
EXPOSE 80
# Test run:
# certbot certonly -a webroot --webroot-path=/usr/share/nginx/html/ -d willarson.com -d www.willarson.com -d lethain.com --test-cert
# Production run:
# certbot certonly -a webroot --webroot-path=/usr/share/nginx/html/ -d willarson.com -d www.willarson.com -d lethain.com -d www.lethain.com -d systemsandpapers.com -d www.systemsandpapers.com
# add a cron: $ certbot certonly --webroot -w /var/www/example -d example.com -d www.example.com -w /var/www/thing -d thing.is -d m.thing.is