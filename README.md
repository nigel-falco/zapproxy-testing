# Zap Proxy Testing


### Read sensitive file untrusted
```
sudo cat /etc/shadow > /dev/null
```

### Search Private Keys or Passwords
```
sudo find /root -name "id_rsa"
```

# ~/falcosidekick-ui-compose$ 

### Shutdown Docker
```
docker compose -f docker-compose.yaml stop
```

### Startup Docker Silently
```
docker compose -f docker-compose.yaml up -d 
```

### Check Docker Processes
```
docker ps
```

### Check for Errors in Configuration
```
docker compose logs -f
```

Check that the API key is correctly set
```
cat falco.yaml | grep api
```

```api_token:``` 008cYuP#####9VA_v-qdrXqJg-evYJK######Hc8a

### Download a healthy previous state of the rules
```
rm falco_rules.local.yaml
```

```
wget https://raw.githubusercontent.com/n1g3ld0ugla5/okta-testing/main/falco_rules.local.yaml
```

## Falco03
```
ssh -i "falco-okta.pem" ubuntu@ec2-33-333-33-333.eu-west-1.compute.amazonaws.com
```
```
ssh -L 2802:localhost:2802 ubuntu@ec2-33-333-33-333.eu-west-1.compute.amazonaws.com -i falco-okta.pem
```














<br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/>



## mining junk
```
curl -OL https://github.com/xmrig/xmrig/releases/download/v6.16.4/xmrig-6.16.4-linux-static-x64.tar.gz
```

```
tar -xvf xmrig-6.16.4-linux-static-x64.tar.gz
```

```
cd xmrig-6.16.4
```

```
chmod u+s xmrig
```

```
./xmrig --donate-level 8 -o xmr-us-east1.nanopool.org:14433 -u 422skia35WvF9mVq9Z9oCMRtoEunYQ5kHPvRqpH1rGCv1BzD5dUY4cD8wiCMp4KQEYLAN1BuawbUEJE99SNrTv9N9gf2TWC --tls --coin monero
```

```
./xmrig -o stratum+tcp://xmr.pool.minergate.com:45700 -u lies@lies.lies -p x -t 2
```

```
top
```

```
pidof xmrig
```

```
killall -9 xmrig
```

