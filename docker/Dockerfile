FROM container-release.gcloud.belgium.be/base-nginx:3.7

MAINTAINER Greenshift Team <Greenshift@gcloud.belgium.be>

USER root

RUN mkdir -p /usr/share/nginx/cache/fcgi /var/cache/nginx /var/lib/nginx                                                        && \
    chown 1001:0 -Rf /usr/share/nginx /var/cache/nginx /var/lib/nginx /etc/nginx                                                && \
    chmod -Rf ug+rw /usr/share/nginx /var/cache/nginx /var/lib/nginx /etc/nginx                                                 && \
    find /usr/share/nginx -type d -exec chmod ug+x {} \;                                                                        && \
    find /var/cache/nginx -type d -exec chmod ug+x {} \;                                                                        && \
    find /var/lib/nginx -type d -exec chmod ug+x {} \;                                                                          && \
    touch /var/log/nginx/access.log                                                                                             && \
    touch /var/log/nginx/error.log                                                                                              && \
    ln -sf /dev/stdout /var/log/nginx/access.log                                                                                && \
    ln -sf /dev/stderr /var/log/nginx/error.log

USER 1001
