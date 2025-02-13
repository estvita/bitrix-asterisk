# Sync calls with Bitrix24 via ARI, AMI or API (Yeastar)

[Инструкция на русском](README.ru.md)

Tested with Asterisk v. 16, 18, 20 (FreePBX) and [Yeastar](/yeastar/) S50 - if the context names used in the filters differ from those in your system, replace them accordingly.

This script allows sending call history and recording files from Asterisk (FreePBX) to Bitrix24.

## Configuration on the Bitrix24 Side
+ Incoming webhook with permissions: crm, user, telephony. Integrations > Rest API > Other > Incoming Webhook.
+ Outgoing webhook for the ONEXTERNALCALLSTART event (click-to-call). In the "URL of your handler" field, enter the address
 http://X.X.X.X:8000/click2call

### Installation

[RedisJSON](https://github.com/RedisJSON/RedisJSON) is used for temporary storage of call information.

```
docker run -p 6379:6379 --name redis-stack redis/redis-stack:latest
```

```
cd /opt
git clone https://github.com/estvita/bitrix-asterisk.git
cd bitrix-asterisk
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp examples/config.ini config.ini
nano config.ini
```

### Fill in the Data in [config.ini](examples/config.ini)

[app]
+ [debug] - Debug mode (True/False)
+ [port] - Flask app port: 8000
+ [engine] - ami, ari to launch the application

[bitrix] parameters:
+ [url] - Address of the incoming webhook.
+ [token] - Issued by Bitrix when creating an outgoing webhook.
+ [crm_create] - Whether to create a CRM entity or not (1/0).
+ [show_card] - Whether to display the client card or not (1/0).
+ [default_phone] - Default internal number (must be specified in telephony settings - telephony users).

[asterisk] parameters:
+ [ws_type] - wss/ws - required for connecting to ARI.
+ [host] - PBX address (example.com).
+ [port] - AMI/ARI port.
+ [username] - AMI/ARI username.
+ [secret] - AMI/ARI password.
+ [records_url] - URL for call recordings with HTTP Basic Auth (https://example.com/monitor/). Example Apache config: [monitor.conf](examples/monitor.conf).
+ [record_user] - Basic Auth login.
+ [record_pass] - Basic Auth password.
+ [loc_count] - Number of digits for internal extensions. If set to 0, internal calls will also be sent to Bitrix.
+ [loc_contexts] - List of internal (outgoing) call contexts. Default: "from-internal".
+ [in_contexts] - List of inbound call contexts. Default: "from-pstn".
+ [logging] - True/False - Enable/disable logging of received events to a file.

## Running the Integration

```
cd /opt/bitrix-asterisk
source .venv/bin/activate

+ ARI/AMI: python main.py
+ Click2call: gunicorn --bind 0.0.0.0:8000 wsgi:app
+ Yeastar API: python yeastar/app.py

```


## Automatic Startup
Example [systemd](/examples/b24_integration.service) configuration for automatic startup:

```
cp /opt/bitrix-asterisk/examples/b24_integration.service /etc/systemd/system/b24_integration.service
systemctl enable b24_integration.service
systemctl start b24_integration.service
```
