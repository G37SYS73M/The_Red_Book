# Listeners

1. Go to **Cobalt Strike > Listeners** to bring up the Listeners tab.
2. Click **Add** to create a new listener.

### HTTP <a href="#http" id="http"></a>

1. Name: http
2. Payload: Beacon HTTP
3. HTTP Hosts: www.bleepincomputer.com
4. HTTP Host (Stager) : www.bleepincomputer.com

### SMB <a href="#smb" id="smb"></a>

1. Name: smb
2. Payload: Beacon SMB
3. Pipename: TSVCPIPE-4b2f70b3-ceba-42a5-a4b5-704e1c41337

### TCP <a href="#tcp" id="tcp"></a>

1. Name: tcp
2. Payload: Beacon TCP
3. Port: 4444
4. Bind to localhost: False

### TCP (local) <a href="#tcp-local" id="tcp-local"></a>

1. Name: tcp-local
2. Payload: Beacon TCP
3. Port: 1337
4. Bind to localhost: True
