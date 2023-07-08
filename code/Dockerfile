# https://hub.docker.com/r/aliyunfc/headless-ffmpeg
FROM aliyunfc/headless-ffmpeg

# set time zone (current is Shanghai, China)
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
RUN dpkg-reconfigure -f noninteractive tzdata

ENV LANG zh_CN.UTF-8
ENV LANGUAGE zh_CN:zh
ENV LC_ALL zh_CN.UTF-8
# set Xvfb auth file
ENV XAUTHORITY=/tmp/Xauthority

WORKDIR /code

ENV PUPPETEER_SKIP_DOWNLOAD=true
RUN npm install puppeteer-core \
  express \
  ali-oss \
  fs  \
  --registry http://registry.npm.taobao.org

COPY ./record.sh ./record.sh
COPY ./record.js ./record.js
COPY ./server.js ./server.js

RUN mkdir -p /var/output

EXPOSE 9000

ENTRYPOINT ["node", "server.js"]