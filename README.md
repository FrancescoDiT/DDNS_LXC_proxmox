# Creating a container LXC with docker inside
## **REQUIREMENTS**
* **OS**: a Debian template (you can find it by clicking the button "Templates" in *Datacenter* **>** *your_node_name* **>** *local(your_node_name)* **>** *CT Templates*)
  
(I'm using *debian-12-standard_12.7-1_amd64.tar.zst*).

* **Container Hardware**
  * **CPU**: 1 core
  * **Memory**: 128MB
  * **Bootdisk**: 2GB

* A public domain (I'm using a [cloudflare](https://www.cloudflare.com/ "cloudflare official site") domain).

---

### 1. Create a subdomain

Go to your domain provider page and look up for the *create subdomain* section, probably under the *DNS* section, then create a subdomain.

**subdomain table structure**
* **Name**: give it a name (i.e. ddns),
* **Content**: set to a random ip value (i.e. 8.8.8.8),
* **Proxy Status**: *DNS only*,
* **TTL**: *auto*,
Save it.

### 2. Set script data and autorun it

On Proxmox, enter you Debian LXC container console and update and upgrade your container:

```bash
apt update && apt upgrade -y
```

If you don't have curl installed:

```bash
apt install curl
```

And create a file wherever you want (if you want to save the template create two files):

```bash
nano ./cloudflare-template.sh
```

Copy this script in it (or in both if you want to save the template), and insert all the required data:

```ini
auth_email="YOUR_AUTH_EMAIL" #required, the email of your cloudflare account
auth_method="global"
auth_key="YOUR_AUTHorAPI_KEY" #required, in the overview page there is a link to it, follow the steps
zone_identifier="YOUR_ZONE_IDENTIFIER" #required, you can find it in the "overview" page
record_name="subdomain.domain.com" #required
ttl=3600

ip=$(curl -s http://ipv4.icanhazip.com)
record_id=$(curl -s -X GET "https://api.cloudflare.com/client/v4/zones/$zone_identifier/dns_records?name=$record_name" \
    -H "X-Auth-Email: $auth_email" \
    -H "X-Auth-Key: $auth_key" \
    -H "Content-Type: application/json" | jq -r '{"result"}[] | .[0] | .id')

update=$(curl -s -X PUT "https://api.cloudflare.com/client/v4/zones/$zone_identifier/dns_records/$record_id" \
    -H "X-Auth-Email: $auth_email" \
    -H "X-Auth-Key: $auth_key" \
    -H "Content-Type: application/json" \
    --data '{"type":"A","name":"'"$record_name"'","content":"'"$ip"'","ttl":'"$ttl"',"proxied":false}')

echo "$update" #if you don't want any log, comment this line
```
Then save and exit from nano and run this command to let your file be executable:

```bash
chmod +x ./cloudflare-template.sh
```

run your file with:

```bash
./cloudflare-template.sh
```

and add it to crontab, so it can run automatically:

```bash
crontab -e
```

add this row in crontab:

```ini
#this script runs every minute, but you can adjust it as you prefer
* * * * * * /path/to/cloudflare-template.sh
```

then save and exit, and that's it!

check your subdomain column **Value** on you domain provider website to see if it changed.
