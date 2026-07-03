# L2TP-Client-to-Site-con-IKEv1

<img width="654" height="276" alt="image" src="https://github.com/user-attachments/assets/edfcc30b-66cc-4465-95dc-13ea7e5fda80" />

**LABORATORIO DE NETWORKING**

**L2TP Client-to-Site con IKEv1**

_Documentación Técnica Profesional_

**Heldry Terrero**

Matrícula: 2025-0719

Materia: Seguridad de Redes

Fecha: Junio 2026

|   |
|---|
|⚠ AVISO IMPORTANTE — Este proyecto fue desarrollado únicamente con fines educativos y de laboratorio controlado, en el marco de la asignatura Seguridad de Redes del ITLA. Los scripts y técnicas documentadas SOLO deben ejecutarse en simuladores autorizados (PNetLab, GNS3, EVE-NG). QUEDA ESTRICTAMENTE PROHIBIDO usar este material en redes públicas o de terceros sin autorización explícita. El uso indebido puede constituir un delito conforme a las leyes de ciberseguridad de la República Dominicana.|

  

  

# 1. Objetivo

VPN Client-to-Site L2TP/IPSec IKEv1. El cliente Windows usa la VPN nativa del sistema. R-L2TP configura AAA, VPDN, Virtual-Template y Crypto Map dinámico. Credenciales: vpnuser/cisco123. Pool VPN: 20.25.7.100-110.

•        Configurar AAA para autenticación PPP local.

•        Habilitar VPDN como servidor L2TP.

•        Crear Virtual-Template con ip unnumbered y pool de IPs.

•        Usar Crypto Map dinámico para IPs variables de clientes.

•        Verificar con show vpdn y show crypto isakmp sa.

# 2. Componentes L2TP/IPSec

|**Componente**|**Función**|**Comando clave**|
|---|---|---|
|AAA|Autenticación local PPP|aaa new-model + aaa authentication ppp default local|
|Usuario local|Credenciales del cliente|username vpnuser password 0 cisco123|
|IP Pool|IPs virtuales para clientes|ip local pool POOL_VPN 20.25.7.100 20.25.7.110|
|VPDN|Servidor L2TP|vpdn enable + vpdn-group 1|
|Virtual-Template|Interfaz por cliente conectado|ip unnumbered + peer default ip pool|
|Crypto Map dinámico|Cifra IPs variables|crypto dynamic-map DYN_MAP|
|no l2tp tunnel auth|Deshabilita auth de tunel L2TP|L2TP usa IPSec para seguridad|

# 3. Topología

<img width="991" height="160" alt="9" src="https://github.com/user-attachments/assets/0985e18b-8082-413b-885b-f2815ca23d7e" />

|---|

|**Dispositivo**|**Interfaz**|**IP**|**Máscara**|**Descripción**|
|---|---|---|---|---|
|R-ISP|e0/1|20.25.6.1|255.255.255.252|Hacia PC-Cliente|
|R-ISP|e0/0|20.25.16.1|255.255.255.252|Hacia R-L2TP|
|R-L2TP|e0/0|20.25.16.2|255.255.255.252|WAN ISP|
|R-L2TP|e0/1|20.25.14.1|255.255.255.0|LAN Interna|
|PC-CLIENTE|NIC|20.25.6.2|255.255.255.252|Windows 10|
|IP Virtual VPN|(asignada)|20.25.7.100-110|255.255.255.0|Pool clientes VPN|

## Credenciales VPN

|**Campo**|**Valor**|
|---|---|
|Tipo VPN (Windows)|L2TP/IPSec con clave precompartida|
|Servidor VPN|20.25.16.2|
|Clave precompartida|cisco123|
|Usuario PPP|vpnuser|
|Contraseña PPP|cisco123|
|Autenticación|MS-CHAPv2|

# 4. Scripts

## 4.1 R-ISP

enable  
configure terminal  
hostname R-ISP  
!  
interface ethernet0/1  
 ip address 20.25.6.1 255.255.255.252  
 no shutdown  
!  
interface ethernet0/0  
 ip address 20.25.16.1 255.255.255.252  
 no shutdown  
!  
do write memory

## 4.2 R-L2TP — Servidor VPN

enable  
configure terminal  
hostname R-L2TP  
!  
aaa new-model  
aaa authentication ppp default local  
aaa authorization network default local  
!  
username vpnuser password 0 cisco123  
!  
interface ethernet0/0  
 description WAN_ISP  
 ip address 20.25.16.2 255.255.255.252  
 no shutdown  
!  
interface ethernet0/1  
 description LAN_INTERNA  
 ip address 20.25.14.1 255.255.255.0  
 no shutdown  
!  
ip route 0.0.0.0 0.0.0.0 20.25.16.1  
!  
ip local pool POOL_VPN 20.25.7.100 20.25.7.110  
!  
crypto isakmp policy 10  
 encryption 3des  
 hash sha  
 authentication pre-share  
 group 2  
!  
crypto isakmp key cisco123 address 20.25.6.2  
!  
crypto ipsec transform-set TS_L2TP esp-3des esp-sha-hmac  
 mode transport  
!  
crypto dynamic-map DYN_MAP 10  
 set transform-set TS_L2TP  
!  
crypto map MAP_L2TP 10 ipsec-isakmp dynamic DYN_MAP  
!  
interface ethernet0/0  
 crypto map MAP_L2TP  
!  
vpdn enable  
!  
vpdn-group 1  
 accept-dialin  
  protocol l2tp  
  virtual-template 1  
 no l2tp tunnel authentication  
!  
interface Virtual-Template1  
 ip unnumbered ethernet0/1  
 peer default ip address pool POOL_VPN  
 ppp authentication ms-chap-v2  
!  
do write memory

|   |
|---|
|3DES + SHA1 + Grupo 2 son requeridos por compatibilidad con el cliente L2TP nativo de Windows 10. Windows no negocia AES-256 en la configuración L2TP/IPSec built-in sin modificar el registro del sistema.|

# 5. Verificación

## 5.1 Conectar desde Windows 10

Configuracion > Red > VPN > Agregar conexion VPN > L2TP/IPSec con clave precompartida. Servidor: 20.25.16.2. PSK: cisco123. Usuario: vpnuser. Password: cisco123.

<img width="338" height="365" alt="vpn" src="https://github.com/user-attachments/assets/45247ea3-344a-488b-aeee-ca337b6dc6ba" />


<img width="1123" height="620" alt="conectado" src="https://github.com/user-attachments/assets/d80a05c6-999a-41fb-abdc-cbc911c7fc37" />

|---|
||

## 5.2 show vpdn en R-L2TP

R-L2TP# show vpdn  
L2TP Tunnel and Session Information Total tunnels 1 sessions 1  
LocID  RemID  Username    State  
1      1      vpnuser     est   <- sesion L2TP activa

<img width="977" height="267" alt="show vpdn" src="https://github.com/user-attachments/assets/a2d825ee-3e45-42f1-8cbd-b445a147dbc4" />

|---|
||

## 5.3 show crypto isakmp sa — QM_IDLE

R-L2TP# show crypto isakmp sa  
dst          src          state    conn-id status  
20.25.16.2   20.25.6.2   QM_IDLE     1001 ACTIVE

<img width="748" height="144" alt="isakmp" src="https://github.com/user-attachments/assets/76559f02-c01b-4de3-aebf-7428dd30c125" />

|---|
||

# 6. Troubleshooting

|**Error Windows**|**Causa**|**Solución**|
|---|---|---|
|Error 789|IPSec no establece SA|Verificar PSK y policy (3DES/SHA/G2)|
|Error 691|Credenciales PPP incorrectas|Verificar username vpnuser password cisco123|
|Error 809|UDP 1701 bloqueado|Habilitar NAT-T: crypto isakmp nat keepalive 20|
|show vpdn vacío|VPDN no habilitado|Verificar vpdn enable y vpdn-group 1|

# 7. Conclusión

L2TP/IPSec Client-to-Site permite acceso remoto con credenciales usuario/contraseña sin software adicional en Windows. La arquitectura combina AAA (autenticación), VPDN (servidor L2TP), Virtual-Template (IP virtual) y Crypto Map dinámico (cifrado).


|RESULTADO: Sesion L2TP activa con vpnuser, QM_IDLE confirmado, acceso a red interna 20.25.14.0/24.|

Heldry Terrero — Matrícula: 2025-0719 — Seguridad de Redes — Junio 2026

Link del Video: [https://youtu.be/0Gs5nlrgTXQ](https://youtu.be/0Gs5nlrgTXQ)  
  

Link del GitHub: [https://github.com/HeldryRoot/L2TP-Client-to-Site-con-IKEv1](https://github.com/HeldryRoot/L2TP-Client-to-Site-con-IKEv1)
