# Impervious.AI Hackathon Notes

The official quick start guide for the Impervious Hackathon can be found here:
<https://docs.impervious.ai/quickstart>

This project tries to give you step by step instructions for setting up a basic
dev environment.

It focuses on the configuration for Polar that allows you to do local
development using the `regtest` network.

Once you've played around with the Impervious API using Polar and have found
your "Lightning Sealegs" you'll find it easy to migrate your code to a
pair of development nodes running at Voltage on `testnet`.

To get started clone this repository to your local workstation and cd into it:
```
git clone https://github.com/lispmeister/legendary-fiesta.git
cd legendary-fiesta
```


## Create Voltage Account

The voucher for the hackathon is `IMPERVIOUSHACKATHON`
You should be able to claim the voucher at this address:
<https://app.voltage.cloud/hackforfreedom/>

Create a Voltage account and then use the voucher to create at least one LND
node on `testnet`.


## Lightning Labs Faucet

1. You need to create an outbound connection from one of your LND nodes to the faucet first
2. Login to the shell of your LND node and issue the following command:
```
lncli connect 0270685ca81a8e4d4d01beec5781f4cc924684072ae52c507f8ebe9daf0caaab7b@159.203.125.125:9735
```
3. Once the connection has been established you'll be able to send yourself some
   coins: <https://faucet.lightning.community/>
4. Use the pubkey of your LND node as the address in the first field and
   default parameters suggested by the form.

Alternatively you can use one of the dashboards (Thunderhub, Ride The Lightning)
on the Voltage dashboard to create the initial connection to the faucet.


## Polar

Download Polar here: <https://lightningpolar.com/>

### Create LND Nodes on Polar

Create a simple three node network with a single `bitcoind` node and two `LND` nodes
named Alice and Bob.
Start the three node network and push the "Quick Mine" button a couple of times
to create some test coins.
Fund the two `LND` nodes with coins from your `bitcoind` treasury by clicking
the "Deposit" button.

### Activate Keysend

Click to select each `LND` node in the network diagram and select the action
tab. There you'll want to select "Edit Options" and in the pop-up click on the
"prefill" icon. You'll then edit the text in the box and add the parameter
`--accept-keysend` (see example below). Click "Save" and you're done. You'll
need to to this for both `LND` nodes.

```
Name: Alice

GRPC Host	127.0.0.1:10001
REST Host	https://127.0.0.1:8081
P2P Internal	038c85b83aa4d2f0cdbcedded633201f6346c05bc09e19eb81f512b4d8d4d1319f@172.18.0.3:9735
P2P External	038c85b83aa4d2f0cdbcedded633201f6346c05bc09e19eb81f512b4d8d4d1319f@127.0.0.1:9735

lnd --accept-keysend --noseedbackup --trickledelay=5000 --alias=alice --externalip=alice
      --tlsextradomain=alice --tlsextradomain=polar-n1-alice
      --listen=0.0.0.0:9735 --rpclisten=0.0.0.0:10009 --restlisten=0.0.0.0:8080
      --bitcoin.active --bitcoin.regtest --bitcoin.node=bitcoind
      --bitcoind.rpchost=polar-n1-backend1 --bitcoind.rpcuser=polaruser
      --bitcoind.rpcpass=polarpass
      --bitcoind.zmqpubrawblock=tcp://polar-n1-backend1:28334
      --bitcoind.zmqpubrawtx=tcp://polar-n1-backend1:28335
```

```
Name: Bob

GRPC Host	127.0.0.1:10002 
REST Host	https://127.0.0.1:8082
P2P Internal	03e9d9b4e0cee81aeed29fec0eb87283717122f02017168ac55311178899e9a571@172.18.0.2:9735
P2P External	03e9d9b4e0cee81aeed29fec0eb87283717122f02017168ac55311178899e9a571@127.0.0.1:9736

lnd --accept-keysend --noseedbackup --trickledelay=5000 --alias=bob --externalip=bob
      --tlsextradomain=bob --tlsextradomain=polar-n1-bob --listen=0.0.0.0:9735
      --rpclisten=0.0.0.0:10009 --restlisten=0.0.0.0:8080 --bitcoin.active
      --bitcoin.regtest --bitcoin.node=bitcoind
      --bitcoind.rpchost=polar-n1-backend1 --bitcoind.rpcuser=polaruser
      --bitcoind.rpcpass=polarpass
      --bitcoind.zmqpubrawblock=tcp://polar-n1-backend1:28334
      --bitcoind.zmqpubrawtx=tcp://polar-n1-backend1:28335
```

Once you're done with "Edit Options" you can stop the network and start it
again. It should come up with `keysend` activated.

## Download Impervious Binary

We are assuming you will run the following commands inside the directory of the
cloned repository with these instructions.


Download the most recent release binary from here: <https://github.com/imperviousai/imp-releases/releases>.
You should download all three files for your platform ending with `tar.gz`, `md5` and `sha256`. We will use
the latter two to verify the integrity of the release binary.

### Don't Trust Verify
Show the sha256 hash we downloaded from Github:
```
cat 541e67181900b6fc7f1f8a01424f279fc9d774ff5d8bbedeaee6472b4d938f88
```

Calculate the sha256 of the `tar.gz`:
```
sha256sum impervious-v0.1.2-darwin-amd64.tar.gz
```
Result:
```
541e67181900b6fc7f1f8a01424f279fc9d774ff5d8bbedeaee6472b4d938f88  impervious-v0.1.2-darwin-amd64.tar.gz
```

Looks like we have a match! 

Note: Usually their would be a `sig` file too that contains the signature of
this hash GPG signed by the developer who tagged the release.

### Unpack the Release
Unpack the tar.
```
tar zxvf impervious-v0.1.2-darwin-amd64.tar.gz
```

Create a sub directory and move the binary there
```
mkdir bin
mv impervious ./bin
```

### Remove OSX Quarantine Attribute
#### Using the Finder
In the Finder start it with "Ctrl-Click Open" to remove the
tainted flag OSX adds to the executable. You'll get an error message when you
try to run the commands later in the doc if you forget to do this.

#### Using the Shell
Alternatively run the following command in a shell.
Show the existing xattr for the `impvervious` binary:
```
xattr ./bin/impervious # show the existing xattr
```
Result:
```
com.apple.quarantine
```

Remove the quarantine xattr:
```
sudo xattr -r -d com.apple.quarantine ./bin/impervious
```

### Convenience Script

If you want the `impervious` binary on your path you can source the
following shell script to add the binary to your path:
```
# add the impervious bin dir to our path
IMPDIR=`pwd`
export PATH="$IMPDIR/bin/impervious:$PATH"
```


## Boot First IMP

If you click on one of the `LND` nodes in the Polar network diagram you can
select the "Connect" tab. There you'll find the connection details for the IMP
configuration file below. The pubkey is shown in the list on the "Info" tab for
the node.

Edit the template provided below and provide the `LND` node details as
specified.

Our template for the `Alice_Imp` config in the file `alice-imp-config.yaml` looks
like this:

```yaml
# Server configurations
server:
  enabled: true # enable the GRPC/HTTP/websocket server
  grpc_addr: 127.0.0.1:8881 #IP:port for GRPC
  http_addr: 127.0.0.1:8882 #IP:port for HTTP/websocket
# Redis DB configurations
sqlite3:
  username: admin
  password: supersecretpassword # this will get moved to environment variable or generated dynamically
###### DO NOT EDIT THE BELOW SECTION#####
# Services
service_list:
  - service_type: federate
    active: true
    custom_record_number: 100000
    additional_service_data:
  - service_type: vpn
    active: true
    custom_record_number: 200000
    additional_service_data:
  - service_type: message
    active: true
    custom_record_number: 400000
    additional_service_data:
  - service_type: socket
    active: true
    custom_record_number: 500000
    additional_service_data:
  - service_type: sign
    active: true
    custom_record_number: 800000
    additional_service_data:
###### DO NOT EDIT THE ABOVE SECTION#####

# Lightning
lightning:
  lnd_node:
    ip: 127.0.0.1 #IP of your LND node: Alice
    port: 10001 #GRPC port of your LND node
    pub_key: <replace this with your pubkey for the Alice node> #get your LND pubkey with "lncli getinfo"
    tls_cert: <replace this with the path to the tls.cert file for your Alice node>
    admin_macaroon: <replace this with the path to the admin.macaroon file for your Alice node>
federate:
  ttl: 31560000 #Federation auto delete in seconds
  imp_id: Alice_Imp #plain text string of your IMP node name
vpn:
  price: 100 #per hour
  server_ip: 127.0.0.1 #public IP of your VPN server
  server_port: 51820 #port you want to listen on
  subnet: 10.0.0.0/24 #subnet you want to give to your clients. .1 == your server IP.
  server_pub_key: asdfasdfasdf #get this from your WG public key file
  allowed_ips: 0.0.0.0/0 #what subnets clients can reach. Default is entire world.
  binary_path: /usr/bin/wg #where your installed the "wg" command.
  dns: 8.8.8.8 #set your preferred DNS server here.
socket:
  server_ip: 1.1.1.1 #public IP of your socket server
```

Boot our first Imp:
```
./bin/impervious -config alice-imp-config.yaml
```

## Boot Second IMP

Our template for the `Bob_Imp` config in the file `bob-imp-config.yaml` looks
like this:

```yaml
# Server configurations
server:
  enabled: true # enable the GRPC/HTTP/websocket server
  grpc_addr: 127.0.0.1:9991 #IP:port for GRPC
  http_addr: 127.0.0.1:9992 #IP:port for HTTP/websocket
# Redis DB configurations
sqlite3:
  username: admin
  password: supersecretpassword # this will get moved to environment variable or generated dynamically
###### DO NOT EDIT THE BELOW SECTION#####
# Services
service_list:
  - service_type: federate
    active: true
    custom_record_number: 100000
    additional_service_data:
  - service_type: vpn
    active: true
    custom_record_number: 200000
    additional_service_data:
  - service_type: message
    active: true
    custom_record_number: 400000
    additional_service_data:
  - service_type: socket
    active: true
    custom_record_number: 500000
    additional_service_data:
  - service_type: sign
    active: true
    custom_record_number: 800000
    additional_service_data:
###### DO NOT EDIT THE ABOVE SECTION#####

# Lightning
lightning:
  lnd_node:
    ip: 127.0.0.1 #IP of your LND node: Bob
    port: 10002 #GRPC port of your LND node
    pub_key: <replace this with your pubkey for the Bob node> #get your LND pubkey with "lncli getinfo"
    tls_cert: <replace this with the path to the tls.cert file for your Bob node>
    admin_macaroon: <replace this with the path to the admin.macaroon file for
    your Bob node>
federate:
  ttl: 31560000 #Federation auto delete in seconds
  imp_id: Bob_Imp #plain text string of your IMP node name
vpn:
  price: 100 #per hour
  server_ip: 127.0.0.1 #public IP of your VPN server
  server_port: 51821 #port you want to listen on
  subnet: 10.0.0.0/24 #subnet you want to give to your clients. .1 == your server IP.
  server_pub_key: asdfasdfasdf #get this from your WG public key file
  allowed_ips: 0.0.0.0/0 #what subnets clients can reach. Default is entire world.
  binary_path: /usr/bin/wg #where your installed the "wg" command.
  dns: 8.8.8.8 #set your preferred DNS server here.
socket:
  server_ip: 1.1.1.1 #public IP of your socket server
```

Boot our second Imp:
```
./bin/impervious -config bob-imp-config.yaml
```


## Create a Connection

We have two LND nodes and two IMP nodes. We need to make sure the two LND nodes
are connected to each other and have at least one channel open. Let's take care
of that. Under the "Actions" tab for the `Alice` node click on "Launch" to open
a terminal window on the container running our `Alice` LND node.

Check we can connect to our LND node:
```
lncli getinfo
```

You might have to change the IP address used in the connection command below.
For this command you'll want to choose the "P2P Internal" address in the
"Connect" tab of your "Bob" node. Connect to `Bob`:

```
lncli connect <pubkey for Bob node>@172.26.0.4:9735
```
Result:
```
[lncli] rpc error: code = Unknown desc = already connected to peer: 03e9d9b4e0cee81aeed29fec0eb87283717122f02017168ac55311178899e9a571@172.26.0.4:9735
```

OK, turns out we are already connected. Let's check if we have a channel open:
```
lncli listchannels
```
Result:
```
{
    "channels": [
    ]
}
```
No open channels yet. Let's check our wallet balance:
```
lncli walletbalance
```
Result:
```
{
    "total_balance": "3000000",
    "confirmed_balance": "3000000",
    "unconfirmed_balance": "0"
}
```

Got some coins! We need to open a channel to Bob.
```
lncli openchannel \
--node_key <pubkey for Bob node>\
--local_amt 1000000 
```
Result:
```
{
        "funding_txid": "1733b99a8dec3e0cd3640255a27566dec9b74e6ac9ecf3b5c5d402741e849c93"
}
```

We're on the regtest network. Let's push the "Quick Mine" button in Polar a
couple of times to get the block mined that records our channel commit.

Check our wallet balance again:
```
{
    "total_balance": "1988950",
    "confirmed_balance": "1988950",
    "unconfirmed_balance": "0"
}
```

Check our channels:
```
lncli listchannels
```
Result:
```

    "channels": [
        {
            "active": true,
            "remote_pubkey": "03e9d9b4e0cee81aeed29fec0eb87283717122f02017168ac55311178899e9a571",
            "channel_point": "1733b99a8dec3e0cd3640255a27566dec9b74e6ac9ecf3b5c5d402741e849c93:1",
            "chan_id": "168225279115265",
            "capacity": "1000000",
            "local_balance": "990950",
            "remote_balance": "0",
            "commit_fee": "9050",
            "commit_weight": "600",
            "fee_per_kw": "12500",
            "unsettled_balance": "0",
            "total_satoshis_sent": "0",
            "total_satoshis_received": "0",
            "num_updates": "0",
            "pending_htlcs": [
            ],
            "csv_delay": 144,
            "private": false,
            "initiator": true,
            "chan_status_flags": "ChanStatusDefault",
            "local_chan_reserve_sat": "10000",
            "remote_chan_reserve_sat": "10000",
            "static_remote_key": true,
            "commitment_type": "STATIC_REMOTE_KEY",
            "lifetime": "119",
            "uptime": "119",
            "close_address": "",
            "push_amount_sat": "0",
            "thaw_height": 0,
            "local_constraints": {
                "csv_delay": 144,
                "chan_reserve_sat": "10000",
                "dust_limit_sat": "573",
                "max_pending_amt_msat": "990000000",
                "min_htlc_msat": "1",
                "max_accepted_htlcs": 483
            },
            "remote_constraints": {
                "csv_delay": 144,
                "chan_reserve_sat": "10000",
                "dust_limit_sat": "573",
                "max_pending_amt_msat": "990000000",
                "min_htlc_msat": "1",
                "max_accepted_htlcs": 483
            }
        }
    ]
}
```

This looks good!


## Simple Test

We connect to the `Alice_Imp` node and send a message to the `Bob_Imp` node:
```
curl --location --request POST '127.0.0.1:8882/v1/message/send' \
--header 'Content-Type: application/json' \
--data-raw '{
    "msg": "hello",
    "pubkey": "03e9d9b4e0cee81aeed29fec0eb87283717122f02017168ac55311178899e9a571"
}'
```
Result:
```
{"id":"4c1d471a-7467-47b5-9652-89e99d9df20b"}%
```

We check the logs of our `Alice_Imp` IMP:
```
{"level":"debug","ts":"2021-08-05T19:05:27.559+0400","caller":"SnXd2shi.go:1","msg":"[LND] SignMessage","message":"4c1d471a-7467-47b5-9652-89e99d9df20bhello"}
{"level":"debug","ts":"2021-08-05T19:05:27.564+0400","caller":"D7aPhOjw.go:1","msg":"[LND] SignMessage success","signature":"rdqhezeji5kci38co875ax6ghab3oj9bdxgdtbopzgckfekkx56xrptieiut1tozfudwd38gho9dags5dx6ewxx4obt4zyuumg8inhpy"}
{"level":"debug","ts":"2021-08-05T19:05:27.780+0400","caller":"gEPxcH0V.go:1","msg":"[LND] Payment succeeded"}
{"level":"debug","ts":"2021-08-05T19:05:27.780+0400","caller":"M4s3q8wL.go:1","msg":"[LM] SendData successful"}
{"level":"debug","ts":"2021-08-05T19:05:27.780+0400","caller":"AOzc4KIC.go:1","msg":"[Messenger] NewMsg successful","message_id":"4c1d471a-7467-47b5-9652-89e99d9df20b"}
{"level":"debug","ts":"2021-08-05T19:05:27.780+0400","caller":"Uzn23buF.go:1","msg":"[Core] SendMessage success","message_id":"4c1d471a-7467-47b5-9652-89e99d9df20b"}
{"level":"info","ts":"2021-08-05T19:05:27.780+0400","caller":"oCXAgE8s.go:1","msg":"[Server] SendMessage success","message_id":"4c1d471a-7467-47b5-9652-89e99d9df20b"}
```

We check the logs of our `Bob_Imp` IMP:
```
{"level":"debug","ts":"2021-08-05T19:05:27.756+0400","caller":"GK1uhog8.go:1","msg":"[LND] VerifySignature success","valid":true}
{"level":"debug","ts":"2021-08-05T19:05:27.756+0400","caller":"vrGQDaio.go:1","msg":"[LND] Keysend received","keysend_message":"hello"}
{"level":"debug","ts":"2021-08-05T19:05:27.756+0400","caller":"yMz5wnQq.go:1","msg":"[ServiceHandler] Received message for record","record":400000,"message_id":"4c1d471a-7467-47b5-9652-89e99d9df20b"}
{"level":"debug","ts":"2021-08-05T19:05:27.756+0400","caller":"wvqz9iJR.go:1","msg":"[MSG] message received","message":"hello"}
{"level":"debug","ts":"2021-08-05T19:05:27.756+0400","caller":"_uzgsmIt.go:1","msg":"[ServiceHandler] Publishing message to websocket","message_id":"4c1d471a-7467-47b5-9652-89e99d9df20b"}
{"level":"debug","ts":"2021-08-05T19:05:27.756+0400","caller":"SfLu26vt.go:1","msg":"[ServiceHandler] Handled message","message_id":"4c1d471a-7467-47b5-9652-89e99d9df20b"}
```

Success!


