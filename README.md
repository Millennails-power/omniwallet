## What is Mastercoin?

[from wiki.mastercoin.org]
Mastercoin is both a new type of currency (MSC) and a platform. It is a new protocol layer 
running on top of bitcoin like HTTP runs on top of TCP/IP. Its purpose is to build upon the 
core Bitcoin protocol and add new advanced features, with a focus on a straight-forward and 
easy to understand implementation which allows for analysis and its rapid development. 

## What is this Web-Wallet?

We anticipate most users will not want to understand the technical details of running a standalone wallet.
To counter user dissatisfaction and garner a strong following, Mastercoin will require an easy
to use wallet not requiring user installation. Web-wallets are the typical manifestation of this
requirement, but we recognize the concern for security is not typically heeded. The concern for
security is another motivation in this regard, and a section will be written regarding those 
concerns.

### Risks 

Web wallet security
Traditionally, web wallets are designed with the security implication of handling user data. 
In our case, user data includes bitcoin private keys. Since this is a appealing target for
criminals, our aim in this project will be to minimize the risk of the user in using our 
service, by developing software to run without the handling of user private keys. This has been
attempted with success by the web wallet and blockchain query service blockchain.info. We are 
confident this level of security can and should be implemented by our software.

## Setup

Install sx
```
sudo apt-get install git build-essential autoconf libtool libboost-all-dev pkg-config libcurl4-openssl-dev libleveldb-dev libzmq-dev libconfig++-dev libncurses5-dev
wget http://sx.dyne.org/install-sx.sh
sudo bash ./install-sx.sh
```
update ~/.sx.cfg with an obelisk server details.  Don't have one already set up?  Here's how to build one on Rackspace: https://gist.github.com/curtislacy/8424181
```
# ~/.sx.cfg Sample file.
service = "tcp://162.243.29.201:9091"
```
Make sure you have python libraries installed - note that we use ``apt-get`` to install python-git.  Pip installs an older, stable version, and we need things that start in beta version 0.3.2.
```
sudo apt-get install git python-simplejson python-git python-pip
sudo pip install ecdsa
sudo pip install pycoin
```
Install nginx, and drop in the config included with this codebase.
```
sudo apt-get install uwsgi uwsgi-plugin-python
sudo -s
nginx=stable # use nginx=development for latest development version
add-apt-repository ppa:nginx/$nginx
apt-get update 
apt-get install nginx
exit
sudo cp etc/nginx/sites-available/default /etc/nginx/sites-available
```
Find this section near the beginning of /etc/nginx/sites-available/default:
```
        ## Set this to reflect the location of the www directory within the omniwallet repo.
        root /home/cmlacy/omniwallet/www/;
        index index.html index.htm;
```
Change the ``root`` directive to reflect the location of your omniwallet codebase (actually the www directory within that codebase).
Run npm install
```
npm install
```

## Running

Start nginx by running:
```
sudo service nginx start
```
Using the config included, nginx will launch an HTTP server on port 80.
Start the blockchain parser and python services by running:

```
app.sh
```

This will create a parsing & validation work area in /tmp/omniwallet, and begin parsing the blockchain using the server listed in your .sx.cfg file (see above).

## Development

Most of the HTML in /www is checked in, however three files are generated at install:
```
www/Address.html
www/index.html
www/simplesend.html
```
These are generated by the gen_www.sh script as a part of "npm install".  If you need to regenerate them, and do not wish to run a full "npm install" (though that's pretty fast), you can also run:
```
grunt build
```
If you install the development dependencies (``npm install --development``), you'll also be able to use the ``serve-static.js`` script, which can save you the effort of running nginx if you're just doing development on the static HTML pages.

## API

### Sending coins to an address:

#### Validate the source address:
```
var dataToSend = { addr: from_addr };
$.post('/wallet/validateaddr/', dataToSend, function (data) {}).fail( function() {} );
```
``data`` will contain one of:

| Value                      | Meaning                  |
| -------------------------- | ------------------------ |
| ``{ "status": "OK" }``     | Address is a valid source |
| ``{ "status": "invalid pubkey" }``     | Public key on the blockchain for that address is invalid. |
| ``{ "status": "missing pubkey" }``     | No public key exists on the blockchain for this address.  Usually this means that the address has not yet sent any coins anywhere else. |
| ``{ "status": "invalid address" }``     | This address just isn't valid. |

#### Encode and sign the transaction:
```
var dataToSend = { 
	from_address: from_address, 
	to_address: to_address, 
	amount: amount, 
	currency: currency, 
	fee: fee, 
	marker: marker
};
$.post('/wallet/send/', dataToSend, function (data) {

	//data should have fields sourceScript and transaction
	$('#sourceScript').val(data.sourceScript);
	$('#transactionBBE').val(data.transaction);

}).fail(function () {} );
```

#### Actually push up the transaction:
```
var rawTx = Crypto.util.bytesToHex(sendTx.serialize());
var dataToSend = { signedTransaction: rawTx };
$.post('/wallet/pushtx/', dataToSend, function (data) {}).fail( function() {} );
```
No return value in ``data``.

### Make a trade offer:

#### Validate the source address:
```
var dataToSend = { addr: from_addr };
$.post('/wallet/validateaddr/', dataToSend, function (data) {}).fail( function() {} );
```
``data`` will contain one of:

| Value                      | Meaning                  |
| -------------------------- | ------------------------ |
| ``{ "status": "OK" }``     | Address is a valid source |
| ``{ "status": "invalid pubkey" }``     | Public key on the blockchain for that address is invalid. |
| ``{ "status": "missing pubkey" }``     | No public key exists on the blockchain for this address.  Usually this means that the address has not yet sent any coins anywhere else. |
| ``{ "status": "invalid address" }``     | This address just isn't valid. |

#### Encode the trade offer:
```
var dataToSend = { seller: from_address, amount: amount, price: price, min_buyer_fee: min_buyer_fee, fee: fee, blocks: blocks, currency: currency };
$.post('/wallet/sell/', dataToSend, function (data) {

	//data should have fields sourceScript and transaction\
	$('#sourceScript').val(data.sourceScript);
	$('#transactionBBE').val(data.transaction);

}).fail(function () {} );
```

#### Actually push up the transaction:
```
var rawTx = Crypto.util.bytesToHex(sendTx.serialize());
var dataToSend = { signedTransaction: rawTx };
$.post('/wallet/pushtx/', dataToSend, function (data) {}).fail( function() {} );
```
No return value in ``data``.

### Accept a trade offer:

#### Validate the "buyer" address - the address accepting the offer:
```
var dataToSend = { addr: buyer };
$.post('/wallet/validateaddr/', dataToSend, function (data) {}).fail( function() {} );
```
``data`` will contain one of:

| Value                      | Meaning                  |
| -------------------------- | ------------------------ |
| ``{ "status": "OK" }``     | Address is a valid buyer |
| ``{ "status": "invalid pubkey" }``     | Public key on the blockchain for that address is invalid. |
| ``{ "status": "missing pubkey" }``     | No public key exists on the blockchain for this address.  Usually this means that the address has not yet sent any coins anywhere else. |
| ``{ "status": "invalid address" }``     | This address just isn't valid. |
#### Encode the acceptance:
```
var dataToSend = { buyer: buyer, amount: amount, tx_hash: tx_hash };
$.post('/wallet/accept/', dataToSend, function (data) {

	//data should have fields sourceScript and transaction
	$('#sourceScript').val(data.sourceScript);
	$('#transactionBBE').val(data.transaction);

}).fail( function () {} );
```
#### Actually push up the signed transaction **Note - different endpoint from the others!**:
```
var dataToSend = { signedTransaction: signedTransaction };
$.post('/wallet/signed/', dataToSend, function (data) {} ).fail( function () {} );
```
No return value in ``data``.
