FROM golang
ENV user root

# aptitude stuff, because my blog stores content on AWS S3.
RUN apt-get update
RUN apt-get install libyaml-dev -y
RUN apt-get install python-dev -y
RUN apt-get install python-pip -y
RUN pip install awscli --upgrade

# setup area for blog content
RUN mkdir /var/daedalus
RUN chown ${user} /var/daedalus
ADD . /var/daedalus
RUN mkdir /var/static
RUN chown ${user} /var/static

# add empty file
RUN touch /var/static/packed.yaml
RUN chown ${user} /var/static/packed.yaml

# build go app
ENV basedir /usr/local/go/src/github.com/lethain/daedalus
RUN mkdir -p ${basedir}
ADD . ${basedir}
WORKDIR ${basedir}/cmd/daedalus
RUN go install

RUN /var/daedalus/scripts/prod_update.sh

WORKDIR ${basedir}
ENTRYPOINT /usr/local/go/bin/daedalus -config ${basedir}/configs/prod.yaml

# blog's port, note it's not 80! frontend will do that
EXPOSE 6500
