# nginx-layer7-lb
Nginx Setup for Layer 7 (Application Layer) Load Balancing with Client Certificate


## Advantage of using Layer 7 Load Balancer
- URI/Header/Session based Sticky Session possible
- TLS Off loading is done at Load Balancer level to avoid CPU intensive TLS handshaking at Application Level
- Load Balancer to Backend Server can have longer Keep Alive time to avoid frequent TLS handshake 

## Limitations
- 2 TCP connection - 1 from client to Load Balancer and 1 from Load Balancer to Backend Server
- No easy way to pass the client certificate to backend server

## Nginx Setup
Please refer `nginx.conf` file under `conf` directory.

Few important config sections:
- Backend Server Configuration - here you have option to add multiple servers (for HA and LB) and load balancing strategy
  ````
  upstream x-509-server {
     server localhost:8443;
  }
  ````

- Nginx Server Certificate and Key
        Signed using Root Certificate (rootCA_Alok.crt) and Root Certificate should be avaiable at Client
  ````
  ssl_certificate      /Users/aloksingh/cert/localhost.crt;
  ssl_certificate_key  /Users/aloksingh/cert/localhost.key;
  ````
        
- client certificate - in case Nginx need to validate Client as well (mutual authentication)
  ````
  ssl_verify_client on;
  ssl_client_certificate /Users/aloksingh/cert/rootCA_Alok.crt;
  ````

- In case - backend Server need to validate Nginx as well
    - the below certificate is to validate Ngix by backend
    - the certificate must be signed with Root Certificate and should be avaiable at backend trust store
  ````
  proxy_ssl_certificate     /Users/aloksingh/cert/clientNginx.crt;
  proxy_ssl_certificate_key /Users/aloksingh/cert/clientNginx.key;
  proxy_ssl_ciphers         HIGH:!aNULL:!MD5;
  ````

## Hit the Secure URL
    curl --cacert rootCA_Alok.crt --key clientAlok.key --cert clientAlok.crt https://localhost:443/api/user

   Where:
   - `rootCA_Alok.crt` is Load Balancer (Nginx) server Root Certificate
   - `clientAlok.key` is client private key
   - `clientAlok.crt` is client certificate
   - Assuming the `clientAlok.crt` signed using `Root Certificate` (rootCA_Alok.crt). So that Niginx can trust the client certificate.
