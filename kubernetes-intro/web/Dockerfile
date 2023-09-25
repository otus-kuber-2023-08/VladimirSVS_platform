FROM nginx
ARG USER webuser
ARG USER_UID=1001
ARG USER_GID=1001
ENV USER webuser
EXPOSE 8000
RUN DEBIAN_FRONTEND=noninteractive \
    && groupadd --gid $USER_GID $USER \
    && useradd -m -d /app -s /usr/sbin/nologin --uid $USER_UID --gid $USER_GID $USER
COPY homework.html /app/homework.html
COPY nginx.conf /etc/nginx/nginx.conf
RUN chown -R $USER_UID:$USER_GID /app/ \
    && rm /etc/nginx/conf.d/default.conf \
    && touch /var/run/nginx.pid \
    && chown -R $USER_UID:$USER_GID /var/run/nginx.pid \
    && chown -R $USER_UID:$USER_GID /var/cache/nginx
USER $USER

CMD ["nginx", "-g", "daemon off;"]