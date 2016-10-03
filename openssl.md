# openssh - working with SMIME
## Signing message
* generate selfsigned cerificate and private key for signing
 * generate keys for signing
  ```bash
  openssl req -newkey rsa:2048 -nodes -sha256 -keyout signer_priv.pem -out signer_pub.pem
```
 * from public key create certificate (self sign public cerificate by private key)
  ```bash
openssl x509 -req -in signer_pub.pem -signkey signer_priv.pem -out cert.pem -sha256 -days 3650
```
  * append signed public certificate to private key
  ```bash
cat cert.pem >> signer_priv.pem
```
   * replace unsigned public key by signed public certificate
  ```bash
mv cert.pem signer_pub.pem
```
* sign message
```bash
openssl smime -sign -text -text -signer signer_priv.pem -in message.txt > message.sig
```
* verify message
```bash
openssl smime -verify -in message.sig -CAfile signer_pub.pem
```
## Encrypting message
* generate private key and public certificate for encryption
```bash
openssl req -newkey rsa:2048 -nodes -sha256 -keyout enc_priv.pem -out enc_pub.pem
openssl x509 -req -in enc_pub.pem -signkey enc_priv.pem -out cert.pem -sha256 -days 3650
cat cert.pem >> enc_priv.pem
mv cert.pem enc_pub.pem
```
* encrypt message
```bash
openssl smime -encrypt -in message.sig -des3 enc_pub.pem > message.enc
```
* decrypt message
```bash
openssl smime -decrypt -in message.enc -recip enc_pub.pem -inkey enc_priv.pem
```
## Sign and encrypt
* sender: sign and encrypt
```bash
echo "command param" | openssl smime -sign -text -text -signer signer_priv.pem | openssl smime -encrypt -des3 enc_pub.pem > msg.enc
```
* receiver: decrypt and verify
```bash
cat msg.enc | openssl smime -decrypt -recip enc_pub.pem -inkey enc_priv.pem | openssl smime -verify -CAfile signer_pub.pem
cat msg.enc | openssl smime -decrypt -recip enc_pub.pem -inkey enc_priv.pem | openssl smime -verify -CAfile signer_pub.pem 2> /dev/null | sed -n 3p
```


## script gen_keys.sh
```bash
#!/bin/sh
openssl req -newkey rsa:2048 -nodes -sha256 -keyout signer_priv.pem -out signer_pub.pem -subj "/C=GB/ST=London/L=London/O=Global Security/OU=IT Department/CN=example.com"
openssl x509 -req -in signer_pub.pem -signkey signer_priv.pem -out cert.pem -sha256 -days 3650
cat cert.pem >> signer_priv.pem
mv cert.pem signer_pub.pem
openssl req -newkey rsa:2048 -nodes -sha256 -keyout enc_priv.pem -out enc_pub.pem -subj "/C=GB/ST=London/L=London/O=Global Security/OU=IT Department/CN=example.com"
openssl x509 -req -in enc_pub.pem -signkey enc_priv.pem -out cert.pem -sha256 -days 3650
cat cert.pem >> enc_priv.pem
mv cert.pem enc_pub.pem
mkdir sender
cp enc_pub.pem sender/
mv signer_priv.pem sender/
mkdir receiver
mv enc_pub.pem receiver/
mv enc_priv.pem receiver/
mv signer_pub.pem receiver/
```
## sender sign and encrypt
```bash
cd sender
echo "command param" | openssl smime -sign -text -text -signer signer_priv.pem | openssl smime -encrypt -des3 enc_pub.pem > msg.enc
```
## receiver
```bash
cd receiver
cat msg.enc | openssl smime -decrypt -recip enc_pub.pem -inkey enc_priv.pem | openssl smime -verify -CAfile signer_pub.pem
```

