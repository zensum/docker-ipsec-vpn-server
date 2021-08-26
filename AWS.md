# Get started
Launch a debian image on a arm machine e.g `c6g.medium`

1. ssh into it
---
`ssh -i ~/Downloads/Lauchkeypair.pem admin@some-aws-dns-name.eu-central-1.compute.amazonaws.com`


2. install stuff
---
```sh
# docker
sudo apt-get update
sudo apt-get install -y git apt-transport-https ca-certificates curl gnupg lsb-release
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=arm64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io

git clone https://github.com/zensum/docker-ipsec-vpn-server.git vpn
```


3. build it
---
```sh
sudo docker build -t vpn -f Dockerfile.debian .
```


4. configure it
---
```sh
touch .env
# add your config to the .env file like the one below
"""
VPN_DNS_NAME=a.lot.of.characters.eu-central-1.compute.amazonaws.com
VPN_CLIENT_NAME=user_1
"""
```

4. run it
---
```sh
sudo docker run \
    --name vpn \
    --env-file ./.env \
    --restart=always \
    -v vpn_data:/etc/ipsec.d \
    -p 500:500/udp \
    -p 4500:4500/udp \
    -d --privileged \
    vpn
```


5. get first config
---
```sh
sudo docker exec -it vpn bash
cat /etc/ipsec.d/user_1.mobileconfig
# copy the contents and paste it in a new file on your computer
```
Optional, but add dns to mobile config dict
```xml
<key>DNS</key>
<dict>
    <key>ServerAddresses</key>
    <array>
        <string>172.31.0.2</string>
        <string>10.0.0.1</string>
    </array>
    <key>SupplementalMatchDomainsNoSearch</key>
    <integer>0</integer>
</dict>
```

6. get password for config
---
```sh
sudo docker logs vpn
# check for *IMPORTANT* Password for client config files:
# the_password_for_files
```

7. add a user
---
```sh
ikev2.sh
# follow the steps
# copy the path for the config you wanna use

# if the user wants a .mobileconfig use the same steps as below, otherwise do it like this
# to extract the .p12 cert
base64 /etc/ipsec.d/user_2.p12
# copy the contents
```

```sh
# on your machine decode the base64
echo "the string you just copied from docker" | base64 --decode > user_2.p12
```

8. test if it works
---
So now you do
```sh
host a_domain_that_should_go_through_the_vpc.eu-central-1.compute.amazonaws.com
# if the output is a local IP we have success
```

