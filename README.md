# az-kv-whitelist-github-actions-ip
Snippet to whitelist GitHub Actions list of IP in Azure KeyVault Firewall



Whitelist GitHub IP for KV
see [this article on stackoverflow](https://stackoverflow.com/questions/68011051/how-do-i-whitelist-github-actions-ip-addresses-in-azure-web-app-using-githubs-a)
```sh

# https://stackoverflow.com/questions/13777387/check-for-ip-validity
# the if $ip -like ^*\.*\.*\.*$   part just weeds out the IPv6 addresses, which aren't needed
#   ^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$  
#!/bin/bash

for ip in $(curl https://api.github.com/meta | jq .actions[])
do
  STR_LENGTH=`expr $(echo -e $ip | wc -c)`
  IP_CIDR=${ip:1:$STR_LENGTH-3}

  byte1=`echo "$IP_CIDR"|xargs|cut -d "." -f1`
  byte2=`echo "$IP_CIDR"|xargs|cut -d "." -f2`
  byte3=`echo "$IP_CIDR"|xargs|cut -d "." -f3`
  byte4=`echo "$IP_CIDR"|xargs|cut -d "." -f4|cut -d "/" -f1`

  # echo "byte1 byte2 byte3 byte4: $byte1.$byte2.$byte3.$byte4"
  if [[ $byte1.$byte2.$byte3.$byte4 =~ ^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$ && $byte1 -ge 0 && $byte1 -le 255 && $byte2 -ge 0 && $byte2 -le 255 && $byte3 -ge 0 && $byte3 -le 255 && $byte4 -ge 0 && $byte4 -le 255 ]]
    then
      # echo "Found Action IP Cidr $IP_CIDR"
      az keyvault network-rule add  --ip-address $IP_CIDR --name <your KV name kv-xxxx>
  fi
done
```
