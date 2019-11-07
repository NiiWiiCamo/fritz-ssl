# fritz-ssl

Fritz!Box OpenSSL Zertifizierung mit eigener CA

1. Erstellen einer CA:

1.1 Privaten Key der CA erstellen:
openssl genrsa -aes256 -out private/ca-private.pem 4096

1.2 Public Key der CA generieren:
openssl req -x509 -new -nodes -extensions v3_ca -key private/ca-private.pem -days 3650 -out ca-pubkey.pem -sha512

1.3 Public Key (Root Zertifikat) in den eigenen Zertifikatsspeicher importieren:
Bei MacOS muss das Zertifikat importiert und danach noch manuell als "vertrauenswürdig" markiert werden.


2. Erstellen der Zertifikate für die Fritzbox

2.1 Erstellen einer fritzbox.cnf für v3 Requests für hostname, DynDNS und IP:
nano fritzbox/location/fritzbox-location.cnf

[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no
[req_distinguished_name]
C = DE
ST = Berlin
L = Berlin
O = Ich.meinedomain
OU = IT
CN = fritz.box
[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names
[alt_names]
DNS.1 = fritz.box
DNS.2 = DynDNS.meinedomain
IP.1 = 192.168.178.1


2.2 Erstellen eines Private Keys:
openssl genrsa -out fritzbox/location/fb-location-privkey.pem 4096

2.3 Erstellen einer erweiterten Zertifikatsanfrage (v3_req):
openssl req -new -key fritzbox/location/fb-location-privkey.pem -out fritzbox/location/fb-location-request.csr -extensions v3_req -config fritzbox/location/fritzbox-location.cnf

2.4 Pubkey gemäß Zertifikatsanfrage signieren:
openssl x509 -req -days 730 -in fritzbox/location/fb-location-request.csr -CA ca-pubkey.pem -CAkey private/ca-private.pem -CAcreateserial -out fritzbox/location/fb-location-pubkey.pem -sha512 -extensions v3_req -extfile fritzbox/location/fritzbox-location.cnf

2.5 Fritzbox Private Key und Zertifikat in einer Datei zusammenfassen:
cat fritzbox/location/fb-location-privkey.pem fritzbox/location/fb-location-pubkey.pem > fritzbox/location/fb-location-cert.pem

2.6 In der Fritzbox unter "Internet/Freigaben/Fritzbox Dienste" das eigene Zertifikat hochladen.

