FROM ubuntu:zesty

RUN apt-get update
RUN apt-get install -y memcached

EXPOSE 11211
CMD ["-u", "root", "-m", "128"]
USER daemon
ENTRYPOINT memcached
