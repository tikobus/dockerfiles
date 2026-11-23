ARG BASE_ROCKY_VERSION=9.8.20260525.0-minimal

FROM ghcr.io/tikobus/rockylinux:${BASE_ROCKY_VERSION}

ARG NOVNC_VERSION=1.7.0

RUN microdnf install -y python3 python3-pip tar gzip \
 && microdnf clean all \
 && python3 -m venv /opt/venv \
 && /opt/venv/bin/pip install --no-cache-dir websockify

# noVNC
ADD https://github.com/novnc/noVNC/archive/refs/tags/v${NOVNC_VERSION}.tar.gz /tmp/novnc.tar.gz
RUN mkdir -p /opt/novnc \
 && tar xzf /tmp/novnc.tar.gz -C /opt/novnc --strip-components=1 \
 && rm /tmp/novnc.tar.gz

# websockify 插件:token 即 "host:port",由浏览器指定目标
COPY <<'EOF' /opt/app/anytarget.py
import os, re

class AnyTarget:
    def __init__(self, src):
        rx = os.environ.get("ALLOW_REGEX")
        self.allow = re.compile(rx) if rx else None

    def lookup(self, token):
        m = re.fullmatch(r"([A-Za-z0-9.\-]+|\[[0-9a-fA-F:]+\]):(\d{1,5})", token or "")
        if not m:
            return None
        host, port = m.groups()
        if self.allow and not self.allow.fullmatch(f"{host}:{port}"):
            return None
        return (host.strip("[]"), port)
EOF

# 首页:输入地址和端口后跳转到 noVNC
COPY <<'EOF' /opt/novnc/index.html
<!doctype html>
<html lang="zh">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>noVNC</title>
<style>
  body{font-family:system-ui,sans-serif;display:flex;justify-content:center;margin-top:15vh}
  form{display:flex;flex-direction:column;gap:12px;width:280px}
  input,button{padding:8px;font-size:16px}
</style>
</head>
<body>
<form id="f">
  <h3>连接 VNC 服务器</h3>
  <input id="h" placeholder="主机名或 IP" required autofocus>
  <input id="p" type="number" value="5900" min="1" max="65535" required>
  <button>连接</button>
</form>
<script>
document.getElementById('f').onsubmit = function (e) {
  e.preventDefault();
  var target = document.getElementById('h').value.trim() + ':' + document.getElementById('p').value;
  var path = 'websockify?token=' + encodeURIComponent(target);
  location.href = 'vnc.html?autoconnect=1&resize=scale&path=' + encodeURIComponent(path);
};
</script>
</body>
</html>
EOF

ENV PYTHONPATH=/opt/app \
    LISTEN_PORT=6080 \
    ALLOW_REGEX=""

EXPOSE 6080

CMD exec /opt/venv/bin/websockify --web=/opt/novnc \
      --token-plugin=anytarget.AnyTarget --token-source=any \
      0.0.0.0:${LISTEN_PORT}