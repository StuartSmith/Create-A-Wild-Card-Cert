# Create A Wild Card Cert

This a repository to go through the steps to create a test wild card cert for www.waterfrontsoftware.com

To create the certificates we will be using open SSL which should be available on most linux distrobutions. 

## Step 1: Cert Authority.
 
  The fist step in create a Certificate is to have a Certificate Authority. The purpose of certificate authority cert  is to say that the certificate has already been vetted by a third party and should be trusted. We will need to create a certificate authority and install it's cert so the cert that we create will be trusted. 

### Create a private key for the new certificate authority:

1 Generate cert with pass phrase obviously one should choose a stronger pass phrase than 1234abcd
```
openssl genrsa -des3 -out waterfrontCA.key -passout pass:"1234abcd" 2048
```

1 Or generate with user entering pass phrase.
```
openssl genrsa -des3 -out waterfrontCA.key  2048
```


### Next we will create the root certificate that will be good for 5 years

Create a root certificate for the certificate authority
```
openssl req -x509 -new -nodes -key waterfrontCA.key -sha256 -days 1825 -out waterfrontCA.pem
```

### After running the above commands you should have the following:
1. A private key for the certificate authority - saved in the file `waterfrontCA.key`
1. A root certificate for the certificate authority - saved in the file `waterfrontCA.pem`

The following step, we will create a PFX file to import into windows 
```
openssl pkcs12 -export  -inkey waterfrontCA.key -in waterfrontCA.pem -out waterfrontCA.pfx
```


Next we will need to install our site certifate. 

