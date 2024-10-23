## Playing with SOCKS5 during review

These are very rough notes when reviewing https://github.com/bitcoin/bitcoin/pull/29420.  Not claiming full accuracy, so consult the RFC [1928](https://datatracker.ietf.org/doc/html/rfc1928) as needed.
Wanted to stimulate the test framework's SOCKS5 implementation to verify that the redirection/detouring was occurring as expected.

### Initial greeting from the client

VER|NAUTH|AUTH

e.g. for no authentication:
```
0x 05 01 00
```

The server selects no authentication:
```
0x 05 00
```
(or 0xFF if no acceptable methods were offered)

### Request/Reply
Client sends request details.

VER|CMD|RSV|ATYP|DST.ADDR|DST.PORT

e.g., Client wishes to connect to 127.0.0.1 at port 10000
```
0x05 01 00 03 09 31 32 37 2e 30 2e 30 2e 31 27 10
```

e.g., Server replies to the request (succeeded)
VER|REP|RSV|ATYP|BND.ADDR|BND.PORT
```
0x05 00 00 01 00 00 00 00 00 00
```

Afterward, send and receive data through the socket.

### Review
Checked with a hacky little sanity test where a connection was made to the proxy on 127.0.0.1:10000 (set up to detour to 127.0.0.1:10001).  A small echo server listened on 10001 (echoing back "testack").  The new detouring functionality of the proxy seemed to work as expected.

<details>
<summary>hacky test</summary>

```
build/test/functional/feature_framework_unit_tests.py
```

```diff
diff --git a/test/functional/feature_framework_unit_tests.py b/test/functional/feature_framework_unit_tests.py
index 14d83f8a70..9bbfcf987e 100755
--- a/test/functional/feature_framework_unit_tests.py
+++ b/test/functional/feature_framework_unit_tests.py
@@ -29,6 +29,7 @@ TEST_FRAMEWORK_MODULES = [
     "script",
     "script_util",
     "segwit_addr",
+    "socks5",
     "wallet_util",
 ]
 
diff --git a/test/functional/test_framework/socks5.py b/test/functional/test_framework/socks5.py
index 3387c8a1fe..e18d62ee3c 100644
--- a/test/functional/test_framework/socks5.py
+++ b/test/functional/test_framework/socks5.py
@@ -7,6 +7,7 @@
 import select
 import socket
 import threading
+import unittest
 import queue
 import logging
 
@@ -230,3 +231,28 @@ class Socks5Server():
         s.connect(self.conf.addr)
         s.close()
         self.thread.join()
+
+class TestFrameworkSocks5(unittest.TestCase):
+    def test_destinations(self):
+        socks_conf = Socks5Configuration()
+        socks_conf.addr = ('127.0.0.1', 10000)
+        socks_conf.unauth = True
+        socks_conf.auth = False
+        socks_conf.destinations_factory = lambda addr, port : {"actual_to_addr": '127.0.0.1', "actual_to_port": 10001}
+        socks_serv = Socks5Server(socks_conf)
+        socks_serv.start()
+        # send initial greeting, receive response
+        client_sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
+        client_sock.connect(('127.0.0.1', 10000))
+        sendall(client_sock, bytearray([0x05, 0x01, 0x00]))
+        response = recvall(client_sock, 2)
+        assert response == bytearray([0x05, 0x00])
+        # send connect (to 127.0.0.1:10000), receive response
+        sendall(client_sock, bytearray([0x05, 0x01, 0x00, 0x03, 0x09, 0x31, 0x32, 0x37, 0x2e, 0x30, 0x2e, 0x30, 0x2e, 0x31, 0x27, 0x10]))
+        response = recvall(client_sock, 10)
+        assert response == bytearray([0x05, 0x00, 0x00, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00])
+        # send "test" and observe response on 10001 (e.g. with echo server)
+        sendall(client_sock, b'test')
+        response = recvall(client_sock, 7)
+        assert response == b'testack'
+        socks_serv.stop()
```
</details>

<details>
<summary>echo server</summary>

```python
import socket

HOST = "127.0.0.1"
PORT = 10001

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.bind((HOST, PORT))
    s.listen()
    conn, addr = s.accept()
    with conn:
        while True:
            data = conn.recv(1024)
            if not data:
                break
            conn.sendall(data + b'ack')
```
</details>
