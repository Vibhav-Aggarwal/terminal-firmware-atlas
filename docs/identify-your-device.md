# How to identify your device's firmware family

## Step 1 — Read the device's System Info

On almost every terminal: `Menu → System Info` or `Menu → System Info → Device Info`

Look for:
- **Firmware version string** — e.g. `Ver 6.70 Build 6.6.1`, `M61BH v3.16.1118`, `ZAM180`
- **Algorithm names** — `ZKFinger VX10.0`, `EbknFace V3.0`, `ZKFace VX3.9`
- **Product name** — `S-FB3K`, `SpeedFace-V5L`, `X2008`

## Step 2 — Network scan

```bash
nmap -p 23,80,443,3718,4370,5005 <device-ip>
```

| Open ports | Likely family |
|------------|---------------|
| 4370 open, 23 open | ZEM510 (Family 1) |
| 4370 open, 3718 open | ZAM180 (Family 2) |
| 5005 open | Realand M61BH (Family 3) |
| 80 open (ISAPI) | Hikvision (Family 4) |

## Step 3 — Test port 4370

```python
from zk import ZK
zk = ZK('<ip>', port=4370, timeout=5, password=0)
conn = zk.connect()
if conn:
    users = zk.get_users()
    print(len(users))  # 0 = disabled, >0 = working
```

- Data returned → Family 1
- Empty / timeout → try password=123456
- Still empty → Family 2 (Pull disabled)

## Step 4 — Test ADMS probe

```bash
curl "http://<device-ip>/iclock/cdata.aspx?SN=test"
```

If the device responds (even with an error), it may speak ADMS. Configure your server IP in the device menu and test with adms-push-server.

## Step 5 — Check WebSocket behavior

Set device to WebSocket mode with URL `ws://<your-laptop-ip>:8089`, run:
```bash
python3 -c "
import socket, hashlib, base64, time
s = socket.socket()
s.connect(('<your-laptop-ip>', 8089))
key = base64.b64encode(b'testkey1234567x').decode()
s.send(f'GET / HTTP/1.1\r\nHost: x\r\nUpgrade: websocket\r\nConnection: Upgrade\r\nSec-WebSocket-Key: {key}\r\nSec-WebSocket-Version: 13\r\n\r\n'.encode())
time.sleep(2)
print(s.recv(4096))
"
```

If you see XML with `<Request>Login</Request>` → Family 3 (Realand F500).  
If you see binary (non-XML) → proprietary SBPCCOMM (vendor-only protocol, set `ws://` URL instead).
