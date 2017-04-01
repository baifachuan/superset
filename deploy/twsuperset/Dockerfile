FROM alpine:edge

RUN echo 'http://dl-cdn.alpinelinux.org/alpine/edge/testing' >> /etc/apk/repositories && \
    apk add --no-cache g++ python3-dev git nodejs && \
        ln -s /usr/include/locale.h /usr/include/xlocale.h && \
    apk add --no-cache bash curl wget libstdc++ tzdata && \
        ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
        echo "Asia/Shanghai" > /etc/timezone && \
    pip3 install numpy cython && \
    pip3 install pandas && \
    find / -name "*.pyc" -exec rm -f {} ';' && \
    find / -name "*.pyo" -exec rm -f {} ';' && \
    rm -rf /var/cache/apk/* /tmp/* ~/.[^.]*



COPY pykylin.zip /tmp

RUN cd /tmp && unzip pykylin.zip -d .

RUN cd /tmp/pykylin && \
        pip3 install -r ./requirements.txt && \
        python3 setup.py install

RUN apk add --no-cache libffi-dev openldap-dev \
        postgresql-dev && \
    pip3 install superset psycopg2 && \
    cd /tmp && \
        git clone git://github.com/baifachuan/superset.git && \
    cd superset && \
        sed -i "s/# 'zh'/'zh'/" superset/config.py && \
        sed -i "s/BABEL_DEFAULT_LOCALE = 'en'/BABEL_DEFAULT_LOCALE = 'zh'/" superset/config.py && \
        ./superset/assets/js_build.sh && \
    rm -rf /usr/lib/python*/site-packages/superset* && \
        python3 setup.py install && \
    find / -name "*.pyc" -exec rm -f {} ';' && \
    find / -name "*.pyo" -exec rm -f {} ';' && \
    rm -rf /var/cache/apk/* /tmp/* ~/.[^.]*

# Default config
ENV LANG=C.UTF-8 \
    LC_ALL=C.UTF-8 \
    PATH=$PATH:/home/superset/.bin \
    PYTHONPATH=/home/superset/superset_config.py:$PYTHONPATH \
    SQLALCHEMY_DATABASE_URI=sqlite:////home/superset/.superset/superset.db

# Run as superset user
WORKDIR /home/superset
COPY superset .
RUN addgroup superset && \
    adduser -h /home/superset -G superset -D superset && \
    mkdir /home/superset/.superset && \
    touch /home/superset/.superset/superset.db && \
    chown -R superset:superset /home/superset && \
    chmod +x /home/superset/.bin/*
USER superset

# Deploy
EXPOSE 8088
HEALTHCHECK CMD ["curl", "-f", "http://localhost:8088/health"]
ENTRYPOINT ["superset"]
CMD ["runserver"]