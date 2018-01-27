
see https://imququ.com/post/self-hosted-ngrokd.html for more.

## prepare

* point `test.example.com` to your server IP
* point `*.test.example.com` to your server IP
* setup `golang` tools chain on your (build) server

## build

```
git clone git@github.com:haishanh/ngrok.git
cd ngrok
```

generate self signed certificate

```bash
NGROK_DOMAIN="test.example.com"

openssl genrsa -out base.key 2048
openssl req -new -x509 -nodes -key base.key -days 10000 -subj "/CN=$NGROK_DOMAIN" -out base.pem
openssl genrsa -out server.key 2048
openssl req -new -key server.key -subj "/CN=$NGROK_DOMAIN" -out server.csr
openssl x509 -req -in server.csr -CA base.pem -CAkey base.key -CAcreateserial -days 10000 -out server.crt

cp base.pem assets/client/tls/ngrokroot.crt
```

build for server

```bash
sudo make release-server
```

build for client(**note**, you can build for Darwin on Linux)

```bash
$ sudo PATH="/usr/local/go/bin:$PATH" GOOS=darwin GOARCH=amd64 make release-client
$ tar zcvf darwin.tar.gz bin/darwin_amd64

# build fro Raspberry PI
$ sudo PATH="/usr/local/go/bin:$PATH" GOOS=linux GOARCH=arm make release-client

haishan@server:~/repo/ngrok$ tree bin
bin
├── darwin_amd64
│   ├── ngrok
│   └── ngrokd
├── go-bindata
├── linux_arm
│   └── ngrok
├── ngrok
└── ngrokd
```

## start the service on server side

```bash
NGROK_DOMAIN="test.example.com"
./bin/ngrokd -tlsKey=server.key -tlsCrt=server.crt -domain=$NGROK_DOMAIN -httpAddr=":8081" -httpsAddr=":8082"
```

port `8081` will be used for http, `8082` will be used for https. Another port `4443` will be used for client-server communication.


## prepare on the client

get `darwin.tar.gz` from the build server, untar it and move `ngrok` some where.

create `ngrok.cfg` with content:

```yaml
server_addr: test.example.com:4443
trust_host_root_certs: false
```

## usage on the client

```bash
./ngrok -subdomain pub -proto=http -config=ngrok.cfg 80
```
