# pfsense using Let's Encrypt Certificates and HAProxy

**Use cases:**
- Connect securely to web servers or web administration interfaces behind a firewall or in a DMZ.
- Mitigate invalid/unverified certificate errors like ```NET::ERR_CERT_AUTHORITY_INVALID``` when connecting to HTTPS only devices.


**Requirements / Notes:**
- Owner access to domain name with access to API such as Cloudflare or Namecheap
- Ability to be able to create valid e-mail addresses/forwarders for this domain.
- This documentation will look at a wildcard certificate setup so the same Let's Encrypt certificate can be used for all sub domains ie. this.yourdomain.com or that.yourdomain.com.
- If you want to just get rid of certificate errors on internal devices or to test the certificates are working you could internally you can use a Virtual IP address under ```Firewall > Virtual IPs```. Simply add a single valid /32 IP address from your local network and provide a HAProxy_VIP description for ease of identifying it as it will be required for the HAProxy FrontEnd configuration. This means you do not have to open any external ports up on your WAN interface.


### Install required packages:
Goto menu ```System > Package Manager > Available Packages```

1. Find and install package Name 'acme'
2. Find and install package Name 'haproxy'

### Getting a Let's Encrypt Certificate

1. Create to e-mail addresses/forwarders for the domain one for staging and one for production. There are rate limits on the Let's Encrypt production server so we will use the staging server to test the certificate renew process before trying for a production server certificate.
    -  Example: lestage@yourdomain.com and leprod@yourdomain.com
2. Goto menu ```Services > Acme Certificate```

#### Staging (testing) account key

3. Click > ```Account keys```
4. Click > ```+ Add```
Complete as follows:
```
Name:           Staging
Description:    Staging
ACME Server:    Let's Encrypt Staging ACME v2 (for TESTING purposes)
E-Mail Address: lestage@yourdomain.com
```
5. Click > ```Create new account key```
6. Click > ```Register ACME account key```
7. Click > ```Save```

#### Production account key

8. Click > ```Account keys```
9. Click > ```+ Add```
Complete as follows:
```
Name:           Prod
Description:    Production
ACME Server:    Let's Encrypt Production ACME v2 (Applies rate limits...)
E-Mail Address: leprod@yourdomain.com
```
10. Click > ```Create new account key```
11. Click > ```Register ACME account key```
12. Click > ```Save```

#### Add a Certificate (using Staging account key)

1. Click > ```Account keys```
2. Click > ```+ Add```
Complete as follows:
```
Name:               wildcard-yourdomain-com
Description:        wildcard certificate for yourdomain.com
Status:             Active
Acme Account:       Staging
Private Key:        4096-bit RSA
OSSP Must Staple:   *Unchecked
Domain SAN list:    *Click Add
                    Mode        Domainname              Method
                    Enabled     *.yourdomain.com        DNS-Provider
                    Enabled     yourdomain.com          DNS-Provider
```
***You will need to enter you DNS provider API key or other method for Let's Encrypt to validate with your domain. Some providers will want you to specify the IP accessing the API. In this case you will need to add your pfsense's public WAN IP to the API allow list***
```
Leave the remaining defaults:
DNS-Sleep:                  120
Certificate renewal after:  60
```
3. Click > ```Save```
4. Test the certificate by clicking ```Issue/Renew``` and wait as this may take sometime.
5. Once complete an information box should appear. Check the information box for errors and see if the ```Last renewed``` date and time updated to the current date and time. If it didn't then there is a problem. If you see messages about not being able to create a TXT record there may been issues with your domain API key access or method you selected. Check the settings and try again.  

***Don't continue to next steps until the Staging test certificate renew is working***  

6. If it worked click the pencil under ```Actions``` and edit the certificate options.
7. Change: ```ACME account:     Prod``` and go to the bottom and click ```Save```
8. Generate the production certificate by clicking ```Issue/Renew``` and wait.
9. A information box should be displayed indicated the certificate was successfully issued.
10. Goto menu ```System > Certificate Manager``` You should see a list of certificates two with reference to STAGING our test certificates. Below them should be two more production certificates similar to ```Acmecert: O=Let's Encrypt, CN=R3, C=US``` and ```Acmecert: O=Internet Security Research Group, CN=ISRG Root X1, C=US```.

### Testing Certificate on pfsense webConfigurator interface

1. You may need to add a host DNS record for example: firewall.yourdomain.com if this does not already resolve to your pfsense firewall.
2. To do this goto menu ```Services > DNS Resolver```
3. Goto down to ```Host Overrides``` and click ```+ Add```
Complete as follows:
```
Host:           firewall (or name of your choice)
Domain:         yourdomain.com
IP Address:     192.168.1.254 (this needs to be your internal pfsense IP address)
Description:    pfsense Firewall
```
4. Goto bottom and click ```Save```
5. Once saved click ```Apply Changes```
6. Test the record by attempting to ping ```firewall.yourdomain.com```. You may need to flush your DNS cache if this is not resolving to the host override IP specified. If correct continue to the next step.
7. Goto menu ```System > Advanced```
8. Under ```Admin Access``` ```webConfigurator```
9. Check the setting ```SSL/TLS Certificate``` from webConfigurator default to the new certificate ```wildcard-yourdomain-com```.
10. Goto bottom and click ```Save```
11. You should now be able test the pfsense web interface in you web browser.
12. Type in the address of https://firewall.yourdomain.com
13. If your new certificate work a secured page should load without any certificate warnings or errors.


## TO BE CONTINUED... ##
Configuring HAProxy for use with wildcard Let's Encrypt certificate...





