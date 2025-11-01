##Making file
```bash
cd /usr/lib/zabbix/alertscripts
vim smsapp.sh
```
##Bash File
```bash
#!/usr/bin/env bash
# Usage:
#   smsapp.sh "<MESSAGE>" "<RECEPTOR>" "<SENDER (optional)>" "<API_KEY (optional)>"

API_URL="http://api.smsapp.ir/v2/sms/send/simple"

MESSAGE="$1"
RECEPTOR="$2"
SENDER="${3:-10008642}"
API_KEY="${4:-${SMSAPP_APIKEY:-}}"

if [ -z "$MESSAGE" ] || [ -z "$RECEPTOR" ]; then
  echo "Missing MESSAGE or RECEPTOR" >&2
  exit 1
fi

if [ -z "$API_KEY" ]; then
  echo "Missing API key" >&2
  exit 1
fi

RESP_FILE="$(mktemp)"
HTTP_CODE=$(curl --silent --show-error --location "$API_URL" \
  -H "apikey: $API_KEY" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  --data-urlencode "message=${MESSAGE}" \
  --data-urlencode "receptor=${RECEPTOR}" \
  --data-urlencode "sender=${SENDER}" \
  --data-urlencode "output=Json" \
  -o "$RESP_FILE" -w '%{http_code}')

if [ "$HTTP_CODE" -ge 200 ] && [ "$HTTP_CODE" -lt 300 ]; then
  rm -f "$RESP_FILE"
  exit 0
else
  echo "HTTP $HTTP_CODE" >&2
  cat "$RESP_FILE" >&2
  rm -f "$RESP_FILE"
  exit 1
fi
```
##ADD AlertScriptsPath to Systemd File
```
sudo sed -i 's|^#\s*AlertScriptsPath=.*|AlertScriptsPath=/usr/lib/zabbix/alertscripts|' /etc/zabbix/zabbix_server.conf
```
##Give Premission
```
sudo chown zabbix:zabbix /usr/lib/zabbix/alertscripts/smsapp.sh
sudo chmod 777 /usr/lib/zabbix/alertscripts/smsapp.sh
```
##Test command:
```
sudo -u zabbix bash -lc '/usr/lib/zabbix/alertscripts/smsapp.sh "Massage" "<Phone-number>" "10008642" "<api>"; echo exit:$?'
```
##Now Go on Zabbix UI :


``` On Alerts click on Media Type and then create new One ```
<img width="638" height="525" alt="image" src="https://github.com/user-attachments/assets/8aa901ca-7286-4b6d-b2b7-377e8947914a" />

second tab :

<img width="646" height="227" alt="image" src="https://github.com/user-attachments/assets/f4d19e2b-958a-4927-8d87-5f214b4ad227" />

and then go to Administration , then Click On Macros Then Brief the Macro and Values:

<img width="510" height="79" alt="image" src="https://github.com/user-attachments/assets/f3cd6bd9-18db-4b5b-bfe2-e186e7a43621" />

so for Users Menu On Media Tab write Type to <smsapp.sh> and write phone number on Send to field!

and then brief alerting for getting alarm :

<img width="1763" height="273" alt="image" src="https://github.com/user-attachments/assets/792a4e44-ee73-4098-b07c-c064ed443da1" />


<img width="936" height="349" alt="image" src="https://github.com/user-attachments/assets/e80840be-e727-4ef7-ae76-b46fad53f1c5" />


<img width="936" height="488" alt="image" src="https://github.com/user-attachments/assets/3a64b184-a577-4da0-9c97-06b2cf6353b3" />


<img width="531" height="534" alt="image" src="https://github.com/user-attachments/assets/9d88efd1-2207-4088-bdfb-898170616fdb" />

```bash
[PROBLEM] {EVENT.NAME}
Host: {HOST.NAME}
Severity: {EVENT.SEVERITY}
Time: {EVENT.TIME} {EVENT.DATE}
Item: {ITEM.NAME} = {ITEM.LASTVALUE}
Info: {EVENT.OPDATA}
Tags: {EVENT.TAGS}
EventID: {EVENT.ID}
```

<img width="531" height="419" alt="image" src="https://github.com/user-attachments/assets/d2211b65-bf55-4646-ae59-36a0d2b5cb11" />

```bash
[OK] {EVENT.NAME}
Host: {HOST.NAME}
Recovered: {EVENT.RECOVERY.TIME} {EVENT.RECOVERY.DATE}
Duration: {EVENT.DURATION}
Was: {ITEM.NAME} = {ITEM.LASTVALUE1}
Tags: {EVENT.TAGS}
EventID: {EVENT.ID}
```
















