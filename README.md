# Nym Mainnet Guide

This guide is intended for Nym Mainnet.
**IMPORTANT: ALWAYS BACK UP KEY FILES FROM THE FOLDER ~/.nym/mixnodes/<your_id>/data – 4 files with .pem extension! Also, save the mnemonic phrase from your wallet. This is the only way to recover your data.**

## Installation
Replace YOUR_WALLET with the address of your new wallet (Nym Mainnet wallet).

```bash
wallet=YOUR_WALLET
echo 'export wallet='$wallet >> $HOME/.bash_profile
```

To install the node from scratch, execute the following one-liner:

```bash
wget -O nym_mainnet.sh https://api.nodes.guru/nym_mainnet.sh && chmod +x nym_mainnet.sh && ./nym_mainnet.sh
```

Open the Nym Wallet and authenticate using the Mnemonic phrase or password.
Go to the Bond section and fill in all fields; you can obtain the necessary information by executing the command:

```bash
. $HOME/.bash_profile
nym-mixnode node-details --id $node_id
```

Replace YOUR_IDENTITY_KEY with the key obtained above:

```bash
nym-mixnode sign --id $node_id --contract-msg YOUR_IDENTITY_KEY
```

Press Confirm; it should proceed without errors. If successful, you should see the following result.

Return to the terminal and wait for the packet mixing to begin. You can check with the command:

```bash
journalctl -u nym-mixnode -o cat -f | grep "Since startup mixed"
```

Ensure that ports 1789, 1790, 8000 (and 443, 22, 80) are open on your node for proper mixnode operation. More information about ports can be found in the official documentation.
If all went well, your node will appear in the NG Mixnet Explorer.

## Delegation
To delegate, open the wallet and select Delegate. Don't forget to leave tokens for the fee.
- Identity key: Mixnode identity key (Nodes.Guru mixnode 8D5QgyAGqCgChDCxMqQKZpYPNb8hpDNQS43eBJUWagnW)
- Amount to delegate: Number of tokens to delegate
Press DELEGATE STAKE.

You can then check that your delegation has been added to the mixnode stake in NG Mixnet Explorer.

## Useful Commands
Add a description for your mixnode (the name that will be displayed in the explorer):

```bash
nym-mixnode describe --id $node_id
systemctl restart nym-mixnode
```

Check how many packets your node has mixed:

```bash
journalctl -u nym-mixnode -o cat -f | grep "Since startup mixed"
```

Restart the node:

```bash
systemctl restart nym-mixnode
```

Update the node to nym-binaries-v2023.5-rolo. After updating, be sure to specify the new node version in the wallet!

```bash
wget -O nym_update.sh https://api.nodes.guru/nym_update.sh && chmod +x nym_update.sh && ./nym_update.sh
```

Increase ulimit (important for future node operation):

- One-liner option:

```bash
wget -O nym_ulimit.sh https://api.nodes.guru/nym_ulimit.sh && chmod +x nym_ulimit.sh && ./nym_ulimit.sh
```

- Manual option:

```bash
echo "DefaultLimitNOFILE=65535" >> /etc/systemd/system.conf
sudo systemctl daemon-reload
sudo systemctl stop nym-mixnode
sudo systemctl start nym-mixnode
```

Check that the ulimit value is 65535:

```bash
grep -i "Max open files" /proc/$(ps -A -o pid,cmd|grep nym-mixnode | grep -v grep |head -n 1 | awk '{print $1}')/limits
```

Check IPv6:
View your external IPv6:

```bash
curl http://v4v6.ipv6-test.com/api/myip.php && echo
```

Check connectivity to google.de via IPv6:

```bash
ping6 www.google.de
```

Check if IPv6 is present in the server's network settings:

```bash
hostname -I
```

Decode your public keys:
Execute the commands (snapd may be installed differently on your OS):

```bash
apt install snapd
snap install base58
ls -1 $HOME/.nym/mixnodes/*/data/public_identity.pem | while read F; do echo === $F ===; grep -v ^- $F | openssl base64 -A -d | base58; echo; done
ls -1 $HOME/.nym/mixnodes/*/data/public_sphinx.pem | while read F; do echo === $F ===; grep -v ^- $F | openssl base64 -A -d | base58; echo; done
```

## Node Removal
```bash
sudo systemctl stop nym-mixnode
sudo systemctl disable nym-mixnode
rm -rf ~/nym ~/.nym
rm -f /etc/systemd/system/nym-mixnode.service
rm -f /usr/bin/nym-mixnode
```

## Troubleshooting
If the node fails to start (e.g., error could not create TCP Listener; or mixed packets at zero for 10 minutes), check the following:
1) Is TCP port 1789 open on the server?
2) Is TCP port 1789 open for incoming/outgoing connections in the server's firewall settings in the server control panel?
3) If you use Google Cloud, AWS, run the node on non-recommended hosts, or on a home computer:
   3.1) Ensure you have a static IP.
   3.2) Refer to the official documentation for node initialization with custom internal and external IP address specifications.
   3.3) Make sure the correct (external) IP is entered in the Bond form.
