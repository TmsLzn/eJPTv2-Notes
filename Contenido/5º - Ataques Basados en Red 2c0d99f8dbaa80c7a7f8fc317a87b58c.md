# 5º - Ataques Basados en Red 2c0d99f8dbaa80c7a7f8fc317a87b58c

## 5º - Ataques Basados en Red

Los ataques de red se basan en la organización de la red, el tráfico de red y los protocolos de red como:

* ARP
* DHCP
* SMB
* FTP
* Telnet
* SSH

## Filtrado de tráfico con Wireshark

La pestaña de visualización (View) permite al usuario añadir más columnas; con esto, se pueden añadir los puertos de origen (src) y destino (dest) al datagrama.

Existen diferentes capas al inspeccionar los paquetes:

* **Cabeceras (Sección Frame):** La sección de cabecera se ubica con los datos de la trama, contiene información como la longitud en bytes del paquete, la interfaz donde se capturó el paquete y el ID del paquete.
* **Capa de Red (Ethernet II):** La sección de red contiene información como los nombres de host/dominios de origen y destino y las direcciones MAC reales de origen y destino.
* **Capa IP (Protocolo de Internet Versión 4):** Contiene la información sobre las IPv4 de origen y destino.
* **Capa TCP (Protocolo de Control de Transmisión):** Contiene la información sobre los puertos de origen y destino y las banderas (flags) establecidas para el paquete.
* **Capa HTTP o Capa de Aplicación:** Contiene los datos reales del cuerpo del paquete; en el caso de HTTP, el método y los datos transmitidos.

Para exportar artefactos capturados de los flujos TCP/HTTP/Otros se puede hacer mediante `File > Export Objects > HTTP`.

## SMB Relay Attack

SMB Relay Attack consiste en un MITM por el cual redirigimos todo el tráfico SMB que iba a cierta IP hacia la nuestra. Procedimiento para llevarlo a cabo:

* En metasploit usamos módulo `windows/smb/smb_relay`
* En caso de tener un `DNS` el servidor aplicar `DNS Spoofing`
* Y por último lanzamos un `ARP Spoofing` desde el `Víctima <----> Gateway`
* Deberémos obtener una `meterpreter` cuando la víctima acceda a smb

## DNS Spoofing

Para llevar a cabo un DNS Spoofing en una red deberemos de:

* Crear un archivo que redirija peticiones DNS desde un punto de la jerarquía hacia una IP. Esto se hace con: `echo "nuestra_ip" "*rutaDNS.com" > dns"`
* Ejecutamos `dnsspoof` : `dnsspoof -i "interfaz" -f "archivo_dns"`

## Tshark

`Tshark` es una herramienta utilizada para analizar archivos pcap y sus paquetes vía CLI (línea de comandos), es muy similar a tcpdump.

* `tshark -r <captura_pcap>` mostrará el contenido de la captura.
* `tshark -r <captura_pcap> | wc -l` contará la cantidad de líneas, que es lo mismo que el recuento total de paquetes.
* `tshark -r <captura_pcap> -z io,phs -q` mostrará las estadísticas de la Jerarquía de Protocolos (PHS).
* `tshark -r <captura_pcap> -c <conteo_paquetes>` solo mostrará los primeros n paquetes.
* `tshark -r <captura_pcap> -Y '<protocolo>' | more` mostrará el tráfico de un protocolo específico.
* `tshark -r <captura_pcap> -Y 'ip.src==<ip_origen> && ip.dst==<ip_destino>'` mostrará el tráfico desde la IP origen a la IP destino.
* `tshark -r <captura_pcap> -Y 'http.request.method==<método>' | more` mostrará las peticiones que coincidan con el método especificado.
* `tshark -r <captura_pcap> -Y 'http.request.method==<método>' -Tfields -e frame.time -e ip.src -e http.request.full_uri | more` mostrará la petición que coincida con el método especificado; los campos que mostrará son la hora, la IP de origen y la URL solicitada.
* `tshark -r <captura_pcap> -Y 'http contains <cadena_búsqueda>'` buscará la cadena especificada.
* `tshark -r <captura_pcap> -Y 'http.request.method==<método> && http.host==<url_host>' -Tfields -e ip.dst` mostrará la IP de destino coincidente.
* `tshark -r <captura_pcap> -Y 'ip contains <cadena_búsqueda> && ip.src=<ip_origen>' -Tfields -e ip.src -e http.cookie` mostrará la IP de origen y la cookie agregada para la cadena coincidente especificada.
* `tshark -r <captura_pcap> -Y 'ip.src==<ip> && http' -Tfields -e http.user_agent` mostrará la IP de origen y el User-Agent utilizado.

## Envenenamiento ARP (ARP Poisoning)

Para el envenenamiento ARP es importante obtener las direcciones MAC de la red objetivo, se puede usar `wireshark` para ello. Luego, para el spoofing ARP, se puede usar la herramienta `arpspoof` ejecutando:

* `echo 1 > /proc/sys/net/ipv4/ip_forward`
* `arpspoof -i <interfaz> -t <ip_objetivo> -r <host>`

## Análisis de Tráfico WiFi

Filtros de WireShark:

* `wlan contains <SSID>` buscará el tráfico dentro de la red SSID especificada, luego dentro de los paquetes en la sección IEEE se puede encontrar la dirección MAC del transmisor. En la sección `IEEE 802.11 Wireless LAN` dentro de un paquete se puede encontrar el algoritmo de cifrado asociado con la seguridad de la red.
* `(wlan.fc.type_subtype == 0x0008) && (!(wlan.wfa.ie.wpa.version == 1)) && !(wlan.tag.number == 48)` buscará redes wifi abiertas.
* `(wlan.ta == <MAC addr>) || (wlan.ra == <MAC addr>)` mostrará los paquetes recibidos y transmitidos en un dispositivo.
* `wlan contains <ssid> && wlan.fc.type_subtype == 8` localizará los paquetes de tráfico con WPS; para ver si WPS estaba habilitado se puede comprobar buscando `tag vendor specific WPS` dentro del paquete.
* `((wlan.addr == <MAC del router>)) && ((wlan.fc.type_subtype == 0x0020))` buscará tráfico que contenga datos con la dirección MAC especificada.
* `((wlan.bssid == <MAC destino>)) && ((wlan.addr == <MAC del router>)) && ((wlan.fc.type_subtype == 0x0001))` buscará paquetes de solicitud de asociación.

## Filtrado WiFi (con Tshark)

* `tshark -r <captura> -Y 'wlan.fc.type_subtype==0x000c'` mostrará paquetes de desautenticación (deauth).
* `tshark -r <captura> -Y 'eapol'` mostrará los paquetes del handshake.
* `tshark -r <captura> -Y 'wlan.fc.type.subtype==8' -Tfields -e wlan.ssid -e wlan.bssid` mostrará el ssid y bssid de los dispositivos que se conectaron a la red abierta.
* `tshark -r <captura> -Y 'wlan.ssid=<ssid>' -Tfields -e wlan.bssid` mostrará los dispositivos que se conectaron a la red.
* `tshark -r <captura> -Y 'wlan.ssid=<ssid>' -Tfields -e wlan_radio.channel` mostrará los canales usados en la red.
* `tshark -r <captura> -Y 'wlan.fc.type_subtype==0x000c' -Tfields -e wlan.ra` mostrará los dispositivos que recibieron mensajes de desautenticación.
* `tshark -r <captura> -Y 'wlan.ta==<MAC address> && http' -Tfields -e http.user_agents` mostrará el tráfico transmitido desde la dirección MAC especificada y mostrará solo aquellos que contengan tráfico HTTP.
