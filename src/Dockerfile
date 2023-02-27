FROM docker.io/nginx:stable
RUN rm /etc/nginx/conf.d/*
ADD nginx.conf /etc/nginx/conf.d/
ADD index.html /usr/share/nginx/html/
RUN chmod 444 /usr/share/nginx/html/index.html