FROM debian:jessie-backports

# aptitude
RUN apt-key update
RUN apt-get update
RUN apt-get remove apt-listchanges
RUN apt-get install nginx -y
RUN apt-get install curl -y
RUN openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048

ADD ./nginx /etc/nginx/sites-enabled/
RUN mkdir -p /var/log/nginx
RUN touch /var/log/nginx/access.log
RUN touch /var/log/nginx/error.log
# redirect nginx logs to stderr/stdout so k8s logging works
# a bit more intuitively!
RUN ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log

# copy old SSL certs
#COPY ./ssl/fullchain.pem /etc/letsencrypt/live/willarson.com/fullchain.pem
#COPY ./ssl/privkey.pem /etc/letsencrypt/live/willarson.com/privkey.pem

ENTRYPOINT nginx -g "daemon off;"
EXPOSE 80
