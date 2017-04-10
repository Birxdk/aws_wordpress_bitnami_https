# Setting up Wordpress on AWS using Bitnami image #

Setup new EC2 instance with Wordpres using the Bitnami image.

### SSH to server ###
`ssh -i "<keyfile>.pem" bitnami@<publicip>`
Example:
ssh -i "instacekey.pem" bitnami@34.214.12.123

### Remove Bitnami banner ###
`sudo /opt/bitnami/apps/wordpress/bnconfig --disable_banner 1`
Restart the Web server (apache).
`sudo /opt/bitnami/ctlscript.sh restart apache`

## HTTPS / SSL running instance ##
Setup https for your site, it will your improve google ranks too.
You can get your SSL certificate from godaddy or similar.

Generate .csr from godaddy cert
`openssl req -new -newkey rsa:2048 -nodes -keyout mycooldomain.com.key -out mycooldomain.com.csr`
Will give you 2 files:
mycooldomain.com.key - This is the private key
mycooldomain.com.csr - This is the Certificate Signing Request

Upload the csr file to the signing authority, and they will give you back two files.

`mycooldomain.com.crt` - Your cert
`gd_bundle.crt` - Certificate Chain

Generate a .pem file, using the mycooldomain.com.key (created without a password)

`cat mycooldomain.com.crt mycooldomain.com.key gd_bundle.crt > mycooldomain.com.pem`

## AWS setup ##
Add your certificate to the AWS Certificate Manager

Create a new Standard Load Balancer under EC2, and create a https mapping and select your SSL certificate.
Make sure you point to your EC2 wordpress instance.

## HTTP -> HTTPS ##
You want to redirect all http traffinc to https, therefore replace/create a `httpd-prefix.conf`.
Upload it to the ec2 server to the folder /opt/bitnami/apps/wordpress/conf
You can use Cyberduck on mac for this.
```
RewriteEngine On
RewriteCond %{HTTP:X-Forwarded-Proto} =http
RewriteRule . https://%{HTTP:Host}%{REQUEST_URI} [L,R=permanent]

    # App url moved to root
    DocumentRoot "/opt/bitnami/apps/wordpress/htdocs"
    #Alias /wordpress/ "/opt/bitnami/apps/wordpress/htdocs/"
#Alias /wordpress "/opt/bitnami/apps/wordpress/htdocs"

Include "/opt/bitnami/apps/wordpress/conf/httpd-app.conf"
```
And replace the existing one on the server, path:
/opt/bitnami/apps/wordpress/conf

`X-Forwarded-Proto` is a header from ELB, which is used to get the protocol.

Restart apache 
`sudo /opt/bitnami/ctlscript.sh restart apache`

## Route53 ##
Route 53, create A (if a use alias) or CNAME and point it to the ELB dns name

If you are getting mixed content errors because of some paths to the css/js/images are absolute and starts with http add the plugin "Remove Http" and activate. This will ensure no mixed content.