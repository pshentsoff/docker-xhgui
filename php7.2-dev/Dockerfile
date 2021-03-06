FROM        edyan/php:7.2

LABEL       maintainer="Vadim Pshentsov <pshentsoff@gmail.com>"

ARG         DEBIAN_FRONTEND=noninteractive

RUN         apt update && \
            apt upgrade -y && \
            apt install -y \
                curl \
                git \
                git-flow \
                mongodb-server-core supervisor \
                libcap2-bin && \
            setcap CAP_NET_BIND_SERVICE=+eip /usr/bin/php7.2 && \
            apt purge -y libcap2-bin && \
            apt autoremove -y && \
            apt autoclean && \
            apt clean && \
            usermod -u 1000 www-data && \
            groupmod -g 1000 www-data && \
            echo "www-data ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers && \
            find /root /tmp -mindepth 1 -delete && \
            rm -rf /var/lib/apt/lists/* /usr/share/man/* /usr/share/doc/* \
                   /var/cache/* /var/log/* /usr/share/php/docs /usr/share/php/tests

# Prepare Mongodb, supervisord
COPY        conf/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

RUN         mkdir -p /data/db /var/log/mongodb && \
            useradd -d /data/db -M mongodb && \
            chown -R mongodb:mongodb /data/db && \
            # Supervisord
            mkdir -p /var/log/supervisor

ENV         COMPOSER_ALLOW_SUPERUSER 1
RUN         curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer && \
            composer global require hirak/prestissimo --no-plugins --no-scripts

# Clone xhgui and install master as there is no suitable tag
ARG         CLONE_XHGUI_SOURCE="true"
RUN         if [ "$CLONE_XHGUI_SOURCE" = "true" ]; then \
                git clone --depth 1 https://github.com/pshentsoff/xhgui /usr/local/src/xhgui && \
                mkdir vendor && \
                chown -R www-data:www-data /usr/local/src/xhgui && \
                cd /usr/local/src/xhgui && \
                composer install && \
                git flow init -d && \
                composer require --update-no-dev --no-scripts alcaeus/mongo-php-adapter ^1.1; \
            fi
COPY        conf/xhgui.config.php /usr/local/src/xhgui/config/config.php

# Global directives
VOLUME      ["/usr/local/src/xhgui"]
EXPOSE      80 27017
ENV         XHGUI_MONGO_HOST   "mongodb://127.0.0.1"
ENV         MONGO_PORT         27017
ENV         PHP_WEBSERVER_PORT 80

COPY        tests/test.php /root/test.php
COPY        post-run.sh /root/post-run.sh
RUN         chmod +x /root/post-run.sh /root/test.php && \
            chown www-data:www-data /usr/local/src/xhgui/config/config.php && \
            chown -R www-data:www-data /usr/local/src/xhgui/cache

CMD         ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
