FROM kong/kong:3.4
USER root
RUN apt update && apt install -y nodejs npm && npm install -g kong-pdk
# Copy js-hello plugin  to /usr/local/kong/js-plugins
COPY --chown=kong --chmod=755 ./plugins/js_hello/package.json /usr/local/kong/js-plugins/package.json
COPY --chown=kong --chmod=755 ./plugins/js_hello/js_hello.js /usr/local/kong/js-plugins/hello.js
RUN cd /usr/local/kong/js-plugins/ && npm install
# Copy ts-hello plugin  to /usr/local/kong/tss-plugins
COPY --chown=kong --chmod=755 ./plugins/ts_hello/package.json /usr/local/kong/ts-plugins/package.json
COPY --chown=kong --chmod=755 ./plugins/ts_hello/ts_hello.ts /usr/local/kong/ts-plugins/hello.ts
RUN cd /usr/local/kong/ts-plugins/ && npm install
USER kong

