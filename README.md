# Задание 1.

На картинке изображена схема сети, состоящих из трех офисов.

Необходимо создать туннели между каждым офисом.

Между R1-R2 создать ВПН канал и обеспечить связность между клиентами маршрутизаторов по зашифрованному каналу со следующими настройками:
ISAKMP(Ikev1) Phase 1 протоколы:

шифрование 3des  /
хеширование md5
аутентификация pre-share
Диффи-Хелманн группа: 2
время жизни 3600 секунд
IPSEC Transform-set:

ESP: esp-3des
AH: ah-md5-hmac
Mode: Tunnel
Между R1-R3 создать ВПН канал и обеспечить связность между клиентами маршрутизаторов по зашифрованному каналу со следующими настройками:
ISAKMP(Ikev1) Phase 1 протоколы:

шифрование aes 256
хеширование sha512
аутентификация pre-share
Диффи-Хелманн группа: 5
время жизни 3600 секунд
IPSEC Transform-set:

ESP: esp-aes 256
ESP-AH: esp-sha512-hmac
AH: ah-sha256-hmac
Mode: Tunnel
Между R2-R3 создать ВПН канал и обеспечить связность между клиентами маршрутизаторов по зашифрованному каналу со следующими настройками:
ISAKMP(Ikev1) Phase 1 протоколы:

шифрование aes 256
хеширование sha512
аутентификация pre-share
Диффи-Хелманн группа: 5
время жизни 3600 секунд
IPSEC Transform-set:

ESP: esp-aes 256
ESP-AH: esp-sha256-hmac
AH: ah-sha-hmac
Mode: Tunnel
Отправьте полный список конфигураций и политик. Требуется, чтоб все три туннеля могли работать одновременно.

# ОТВЕТ:

# Предварительная настройка (Интерфейсы и Маршрутизация)

R1:

interface Loopback1
 ip address 1.1.1.1 255.255.255.255
! (Остальные интерфейсы настроены по схеме)
ip route 0.0.0.0 0.0.0.0 100.0.0.2

R2:

interface Loopback1
 ip address 2.2.2.2 255.255.255.255
! (Остальные интерфейсы настроены по схеме)

R3:

cisco
interface Loopback1
 ip address 3.3.3.3 255.255.255.255
! (Остальные интерфейсы настроены по схеме)
ip route 0.0.0.0 0.0.0.0 100.0.0.2

# Настройка IPSec (IKEv1) туннелей:

! 1. Настройка ISAKMP Policy для R2 (Sequence 10)
crypto isakmp policy 10
 encr 3des
 hash md5
 authentication pre-share
 group 2
 lifetime 3600
crypto isakmp key CISCO address 100.0.0.2

! 2. Настройка ISAKMP Policy для R3 (Sequence 20)
crypto isakmp policy 20
 encr aes 256
 hash sha512
 authentication pre-share
 group 5
 lifetime 3600
crypto isakmp key CISCO address 100.0.0.3

! 3. Настройка Transform-set
! Для R2 (3des, md5, ah)
crypto ipsec transform-set TS-R2 esp-3des ah-md5-hmac
 mode tunnel

! Для R3 (aes, sha512, ah-sha256)
crypto ipsec transform-set TS-R3 esp-aes 256 ah-sha256-hmac
 mode tunnel

! 4. Настройка ACL (Интересный трафик)
! Трафик от локальной сети R1 (192.168.5.0) к сети R2 (172.16.1.0)
ip access-list extended VPN-TO-R2
 permit ip 192.168.5.0 0.0.0.255 172.16.1.0 0.0.0.255

! Трафик от локальной сети R1 (192.168.5.0) к сети R3 (10.10.10.0)
ip access-list extended VPN-TO-R3
 permit ip 192.168.5.0 0.0.0.255 10.10.10.0 0.0.0.255

! 5. Настройка Crypto Map
crypto map MAP-TO-OFFICES 10 ipsec-isakmp
 set peer 100.0.0.2
 set transform-set TS-R2
 match address VPN-TO-R2

crypto map MAP-TO-OFFICES 20 ipsec-isakmp
 set peer 100.0.0.3
 set transform-set TS-R3
 match address VPN-TO-R3

! 6. Применение на интерфейсе
interface GigabitEthernet0/0
 crypto map MAP-TO-OFFICES

 Маршрутизатор R2

! 1. Настройка ISAKMP Policy для R1 (Sequence 10)
crypto isakmp policy 10
 encr 3des
 hash md5
 authentication pre-share
 group 2
 lifetime 3600
crypto isakmp key CISCO address 100.0.0.1

! 2. Настройка ISAKMP Policy для R3 (Sequence 20)
crypto isakmp policy 20
 encr aes 256
 hash sha512
 authentication pre-share
 group 5
 lifetime 3600
crypto isakmp key CISCO address 100.0.0.3

! 3. Настройка Transform-set
! Для R1 (3des, md5, ah)
crypto ipsec transform-set TS-R1 esp-3des ah-md5-hmac
 mode tunnel

! Для R3 (aes 256, sha512, esp-ah, ah)
crypto ipsec transform-set TS-R3 esp-aes 256 esp-sha512-hmac ah-sha-hmac
 mode tunnel

! 4. Настройка ACL
! Трафик от сети R2 (172.16.1.0) к сети R1 (192.168.5.0)
ip access-list extended VPN-TO-R1
 permit ip 172.16.1.0 0.0.0.255 192.168.5.0 0.0.0.255

! Трафик от сети R2 (172.16.1.0) к сети R3 (10.10.10.0)
ip access-list extended VPN-TO-R3
 permit ip 172.16.1.0 0.0.0.255 10.10.10.0 0.0.0.255

! 5. Настройка Crypto Map
crypto map MAP-TO-OFFICES 10 ipsec-isakmp
 set peer 100.0.0.1
 set transform-set TS-R1
 match address VPN-TO-R1

crypto map MAP-TO-OFFICES 20 ipsec-isakmp
 set peer 100.0.0.3
 set transform-set TS-R3
 match address VPN-TO-R3

! 6. Применение на интерфейсе
interface GigabitEthernet0/0
 crypto map MAP-TO-OFFICES

 Маршрутизатор R3

 ! 1. Настройка ISAKMP Policy для R1 (Sequence 10)
crypto isakmp policy 10
 encr aes 256
 hash sha512
 authentication pre-share
 group 5
 lifetime 3600
crypto isakmp key CISCO address 100.0.0.1

! 2. Настройка ISAKMP Policy для R2 (Sequence 20)
! Внимание: Настройки R2-R3 совпадают с R1-R3 по фазе 1, но transform-set отличается.
crypto isakmp policy 20
 encr aes 256
 hash sha512
 authentication pre-share
 group 5
 lifetime 3600
crypto isakmp key CISCO address 100.0.0.2

! 3. Настройка Transform-set
! Для R1 (aes 256, sha512, ah-sha256)
crypto ipsec transform-set TS-R1 esp-aes 256 ah-sha256-hmac
 mode tunnel

! Для R2 (aes 256, sha512, esp-ah, ah-sha)
crypto ipsec transform-set TS-R2 esp-aes 256 esp-sha512-hmac ah-sha-hmac
 mode tunnel

! 4. Настройка ACL
! Трафик от сети R3 (10.10.10.0) к сети R1 (192.168.5.0)
ip access-list extended VPN-TO-R1
 permit ip 10.10.10.0 0.0.0.255 192.168.5.0 0.0.0.255

! Трафик от сети R3 (10.10.10.0) к сети R2 (172.16.1.0)
ip access-list extended VPN-TO-R2
 permit ip 10.10.10.0 0.0.0.255 172.16.1.0 0.0.0.255

! 5. Настройка Crypto Map
crypto map MAP-TO-OFFICES 10 ipsec-isakmp
 set peer 100.0.0.1
 set transform-set TS-R1
 match address VPN-TO-R1

crypto map MAP-TO-OFFICES 20 ipsec-isakmp
 set peer 100.0.0.2
 set transform-set TS-R2
 match address VPN-TO-R2

! 6. Применение на интерфейсе
interface GigabitEthernet0/0
 crypto map MAP-TO-OFFICES

 # Задание 2. Туннели инкапсуляции (GRE)

Маршрутизатор R1

! Туннель до R2
interface Tunnel12
 ip address 192.168.100.1 255.255.255.252
 tunnel source Loopback1
 tunnel destination 2.2.2.2
! Туннель до R3
interface Tunnel13
 ip address 192.168.100.5 255.255.255.252
 tunnel source Loopback1
 tunnel destination 3.3.3.3

! Маршрутизация трафика клиентов через туннели
! Сеть R2 (172.16.1.0) доступна через Tunnel12
ip route 172.16.1.0 255.255.255.0 Tunnel12
! Сеть R3 (10.10.10.0) доступна через Tunnel13
ip route 10.10.10.0 255.255.255.0 Tunnel13

! Чтобы туннель поднялся, нужен маршрут до Loopback удаленных узлов
ip route 2.2.2.2 255.255.255.255 100.0.0.2
ip route 3.3.3.3 255.255.255.255 100.0.0.3

Маршрутизатор R2

! Туннель до R1
interface Tunnel21
 ip address 192.168.100.2 255.255.255.252
 tunnel source Loopback1
 tunnel destination 1.1.1.1

! Туннель до R3
interface Tunnel23
 ip address 192.168.100.9 255.255.255.252
 tunnel source Loopback1
 tunnel destination 3.3.3.3

! Маршрутизация
ip route 192.168.5.0 255.255.255.0 Tunnel21
ip route 10.10.10.0 255.255.255.0 Tunnel23

! Маршруты до Loopback
ip route 1.1.1.1 255.255.255.255 100.0.0.1
ip route 3.3.3.3 255.255.255.255 100.0.0.3

Маршрутизатор R3

! Туннель до R1
interface Tunnel31
 ip address 192.168.100.6 255.255.255.252
 tunnel source Loopback1
 tunnel destination 1.1.1.1

! Туннель до R2
interface Tunnel32
 ip address 192.168.100.10 255.255.255.252
 tunnel source Loopback1
 tunnel destination 2.2.2.2

! Маршрутизация
ip route 192.168.5.0 255.255.255.0 Tunnel31
ip route 172.16.1.0 255.255.255.0 Tunnel32

! Маршруты до Loopback
ip route 1.1.1.1 255.255.255.255 100.0.0.1
ip route 2.2.2.2 255.255.255.255 100.0.0.2
