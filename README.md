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

The following step, we will create a PFX file to import into windows 
```
openssl pkcs12 -export  -inkey waterfrontCA.key -in waterfrontCA.pem -out waterfrontCA.pfx
```

### After running the above commands you should have the following:
1. A private key for the certificate authority - saved in the file `waterfrontCA.key`
1. A root certificate for the certificate authority - saved in the file `waterfrontCA.pem`
1. A pfx file to import into windows for the root authority.


```
# To Run this script run the following in the powershell command Prompt ...
# set-executionpolicy remotesigned 
# or ...
# powershell –ExecutionPolicy Bypass .\Insert-Cert.ps1

Write-Host '#---------------------------------------------------------';
Write-Host '#';
Write-Host '#  Inserts a Cert into the local Store ';
Write-Host '#';
Write-Host '#';
Write-Host '#---------------------------------------------------------';
Write-Host '#';

function Insert-Cert {
    $CertPass = ConvertTo-SecureString "1234abcd" -AsPlainText -Force 

    $localpath = (Get-Item -Path ".\" -Verbose).FullName;
    $PFx =  "$($localpath)\waterfrontCA.pfx"

    Write-host $PFx
    Import-PfxCertificate -FilePath $PFx -Password $CertPass  -CertStoreLocation Cert:\LocalMachine\Root -Exportable
}


Insert-Cert
```



## Step 2: Site Certificate

Once the Certificate Authority Certificates have been created we can go ahead and create our site certificates. Create a private key for the site cert.

```
openssl genrsa -out www.waterfrontsoftare.com.key 2048
```

Create a certificate signing request other wise known as CSR

```
openssl req -new -key  www.waterfrontsoftare.com.key -out  www.waterfrontsoftare.com.csr
```

Creat the ext file required to sign the cert

```
cat <<EOT >>www.waterfrontsoftare.com.ext
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = www.waterfrontsoftare.com
EOT
```

Create the site certificate using the CSR and config file:
```
openssl x509 -req -in www.waterfrontsoftware.com.csr -CA waterfrontCA.pem -CAkey waterfrontCA.key -CAcreateserial -out www.waterfrontsoftware.com.crt -days 1825 -sha256 -extfile www.waterfrontsoftware.com.ext
```

If working with IIS we will need to change the site cert from crt to pfx below is the command line to do so. 
```
openssl pkcs12 -export -out www.waterfrontsoftware.com.pfx -inkey www.waterfrontsoftware.com.key -in www.waterfrontsoftware.com.crt

```


