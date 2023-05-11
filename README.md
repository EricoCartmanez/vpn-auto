If you want to automate your openvpn3 login so that you don't have to type username, password and token every time you connect, you can follow this guide. 
Of course the purpose of 2FA is pretty much eliminated with this process so, proceed at your own risk.

Parts of the guide can also be applied to openvpn(2) which is easier because it has the `--auth-user-pass` option.


1. In your ovpn config file you add the following:

```
<auth-user-pass>
username
password
</auth-user-pass>
```


2. Then you execute in terminal:

```
openvpn3 config-import --config /path/to/config.ovpn --name MY_NAME --persistent
```

3. From now on you can start your VPN like this:
```
openvpn3 session-start --config MY_NAME
```

------------


If you want to automate the 2FA also, you will need to have the BASE32 of your 2FA. If you have it it's fine, if not you will need the QR code.

4. If you have the QR code you will need a tool called `zbarimg` which you can install like this:
`sudo apt-get install zbar-tools`

5. Then `zbarimg QR.png` and you should get the value of "secret" which will look something like this: `F32RFKQYJLMM4B9Q`

6. In your .bashrc you should export the value you got:
`export VPN_BASE32=F32RFKQYJLMM4B9Q`


7. If you DON'T have the QR code and you 're using authy you can get it following this:
https://gist.github.com/gboudreau/94bb0c11a6209c82418d01a59d958c93

For other apps I have no idea, google it :)

8 There another tool needed which will produce the tokens and you install it like this:
`sudo apt-get install oathtool`

9. So, the simple script we 'll use will look like this:
```
#!/usr/bin/bash

TOTP=$(oathtool --totp --base32 ${VPN_BASE32})

echo "$TOTP"

echo "$TOTP" > /tmp/tok

sleep 2

openvpn3 session-start --config MY_NAME

```


10. In order to use the script and feed the token to it, we 'll use an alias (which you should place in your .bashrc/.zshrc etc)
```
alias auto-vpn='touch /tmp/tok;bash /path/to/script.sh < /tmp/tok'
```

11. You should be good to go running (in a new shell) `auto-vpn`

I know we create the `tok` file already in the script but the alias expects it to be already there before the script runs as it seems, because of the use of `<`. 
