FROM centos:latest

# root user cannot start memcached
RUN groupadd -r memcache && useradd -r -g memcache memcache

RUN yum update -y

RUN yum install -y memcached

USER memcache
EXPOSE 11211
CMD ["memcached"]


