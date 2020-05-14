Лабораторная работа. Развертывание коммутируемой сети с резервными каналами
---------
Топология
---------
![](media/baa231110a4a2f671e605c80e495c1e6.png)

Таблица адресации
---------
| Устройство | Интерфейс | IP-адрес    | Маска подсети |
|------------|-----------|-------------|---------------|
| S1         | VLAN 1    | 192.168.1.1 | 255.255.255.0 |
| S2         | VLAN 1    | 192.168.1.2 | 255.255.255.0 |
| S3         | VLAN 1    | 192.168.1.3 | 255.255.255.0 |

Цели
---------
Часть 1. Создание сети и настройка основных параметров устройства

Часть 2. Выбор корневого моста

Часть 3. Наблюдение за процессом выбора протоколом STP порта, исходя из
стоимости портов

Часть 4. Наблюдение за процессом выбора протоколом STP порта, исходя из
приоритета портов

Общие сведения/сценарий
---------
Избыточность позволяет увеличить доступность устройств в топологии сети за счёт
устранения единой точки отказа. Избыточность в коммутируемой сети обеспечивается
посредством использования нескольких коммутаторов или нескольких каналов между
коммутаторами. Когда в проекте сети используется физическая избыточность,
возможно возникновение петель и дублирование кадров.

Протокол spanning-tree (STP) был разработан как механизм предотвращения
возникновения петель на 2-м уровне для избыточных каналов коммутируемой сети.
Протокол STP обеспечивает наличие только одного логического пути между всеми
узлами назначения в сети путем намеренного блокирования резервных путей, которые
могли бы вызвать петлю.

В этой лабораторной работе команда **show spanning-tree** используется для
наблюдения за процессом выбора протоколом STP корневого моста. Также вы будете
наблюдать за процессом выбора портов с учетом стоимости и приоритета.

**Примечание**. Используются коммутаторы Cisco Catalyst 2960s с Cisco IOS версии
15.0(2) (образ lanbasek9). Допускается использование других моделей коммутаторов
и других версий Cisco IOS. В зависимости от модели устройства и версии Cisco IOS
доступные команды и результаты их выполнения могут отличаться от тех, которые
показаны в лабораторных работах.

**Примечание**. Убедитесь, что все настройки коммутатора удалены и загрузочная
конфигурация отсутствует. Если вы не уверены, обратитесь к инструктору.

Необходимые ресурсы
---------
-   3 коммутатора (Cisco 2960 с операционной системой Cisco IOS 15.0(2) (образ
    lanbasek9) или аналогичная модель)

-   Консольные кабели для настройки устройств Cisco IOS через консольные порты

-   Кабели Ethernet, расположенные в соответствии с топологией

Часть 1:	Создание сети и настройка основных параметров устройства
---------

#### Базово настраиваем каждый коммутатор

	S1-3(config)#no ip domain-lookup
	S1-3(config)#enable secret class
	S1-3(config)#line console 0
	S1-3(config)#password cisco
	S1-3(config)#login
	S1-3(config)#logging synchronous
	S1-3(config)#line vty 0 4
	S1-3(config)#password cisco
	S1-3(config)#login
	S1-3(config)#logging synchronous
	S1-3(config)#banner motd #Attention. Unauthorized users are not allowed.#

#### Назначаем IP адреса в VID 1

	S1(config)# interface vlan 1
	S1(config-if)# ip address 192.168.1.1 255.255.255.0
	S1(config-if)# no shutdown
	S2(config)# interface vlan 1
	S2(config-if)# ip address 192.168.1.2 255.255.255.0
	S2(config-if)# no shutdown
	S3(config)# interface vlan 1
	S3(config-if)# ip address 192.168.1.3 255.255.255.0
	S3(config-if)# no shutdown

Часть 2:	Определение корневого моста
---------

#### Отключаем все порты на коммутаторах.

	S1(config)#interface range e0/1-4
	S1(config-if-range)# sh
	S2(config)#interface range e0/1-24
	S2(config-if-range)# sh
	S3(config)#interface range e0/1-24
	S3(config-if-range)# sh

#### Настраиваем подключенные порты в качестве транковых.

	S1(config)#interface range e0/1-4
	S1(config-if)# switchport mode trunk
	S2(config)#interface range e0/1-4
	S2(config-if-range)# switchport mode trunk
	S3(config)#interface range e0/1-4
	S3(config-if-range)# switchport mode trunk

#### Включаем порты F0/2 и F0/4 на всех коммутаторах.

	S1(config)# interface f0/2
	S1(config-if-range)# no sh
	S1(config)# interface f0/4
	S1(config-if-range)# no sh
	S2(config)# interface f0/2
	S2(config-if-range)# no sh
	S2(config)# interface f0/4
	S2(config-if-range)# no sh
	S3(config)# interface f0/2
	S3(config-if-range)# no sh
	S3(config)# interface f0/4
	S3(config-if-range)# no sh

#### Отображаем данные протокола spanning-tree.

	S1#show spanning-tree 
	VLAN0001
	Spanning tree enabled protocol ieee
	Root ID Priority 32769
	Address 0001.C7CB.D05B
	Cost 19
	Port 4(FastEthernet0/4)
	Hello Time 2 sec Max Age 20 sec Forward Delay 15 sec
	
	Bridge ID Priority 32769 (priority 32768 sys-id-ext 1)
	Address 0060.7061.0C9C
	Hello Time 2 sec Max Age 20 sec Forward Delay 15 sec
	Aging Time 20

	Interface Role Sts Cost Prio.Nbr Type

	Fa0/2 Altn BLK 19 128.2 P2p
	Fa0/4 Root FWD 19 128.4 P2p


	S2#show spanning-tree 
	VLAN0001
	Spanning tree enabled protocol ieee
	Root ID Priority 32769
	Address 0001.C7CB.D05B
	Cost 19
	Port 4(FastEthernet0/4)
	Hello Time 2 sec Max Age 20 sec Forward Delay 15 sec
	
	Bridge ID Priority 32769 (priority 32768 sys-id-ext 1)
	Address 0030.F245.C188
	Hello Time 2 sec Max Age 20 sec Forward Delay 15 sec
	Aging Time 20
	
	Interface Role Sts Cost Prio.Nbr Type

	Fa0/2 Desg FWD 19 128.2 P2p
	Fa0/4 Root FWD 19 128.4 P2p


	S3#show spanning-tree 
	VLAN0001
	Spanning tree enabled protocol ieee
	Root ID Priority 32769
	Address 0001.C7CB.D05B
	This bridge is the root
	Hello Time 2 sec Max Age 20 sec Forward Delay 15 sec
	
	Bridge ID Priority 32769 (priority 32768 sys-id-ext 1)
	Address 0001.C7CB.D05B
	Hello Time 2 sec Max Age 20 sec Forward Delay 15 sec
	Aging Time 20
	
	Interface Role Sts Cost Prio.Nbr Type

	Fa0/2 Desg FWD 19 128.2 P2p
	Fa0/4 Desg FWD 19 128.4 P2p

##### С учетом выходных данных, поступающих с коммутаторов, отвечаем на следующие вопросы.
##### Какой коммутатор является корневым мостом? 
*S3*
##### Почему этот коммутатор был выбран протоколом spanning-tree в качестве корневого моста?
*Самый маленький MAC адрес*
##### Какие порты на коммутаторе являются корневыми портами? 
*Ни одного*
##### Какие порты на коммутаторе являются назначенными портами? 
*F 0/4, F 0/2*
##### Какой порт отображается в качестве альтернативного и в настоящее время заблокирован? 
*Порт коммутатора S1 F0/2*
##### Почему протокол spanning-tree выбрал этот порт в качестве невыделенного (заблокированного) порта?
*По причине наибольшей стоимости пути доставки*

Часть 3:	Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов
---------

#### Изменим стоимость порта

	S1(config)# interface f0/2
	S1(config-if)# spanning-tree cost 18

	*до 
	Fa0/2 Altn BLK 19 128.2 P2p
	Fa0/4 Root FWD 19 128.4 P2p

	*после
	Fa0/2 Root FWD 18 128.2 P2p
	Fa0/4 Desg FWD 19 128.4 P2p 

#### Удалите изменения стоимости порта.

	S1(config)# interface f0/2
	S1(config-if)# no spanning-tree cost 18

Часть 4:	Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов
---------

#### Включаем порты F0/1 и F0/3 на всех коммутаторах.

	S1-3(config)# interface f0/1
	S1-3(config-if)# no sh
	S1-3(config)# interface f0/3
	S1-3(config-if)# no sh

##### Какой порт выбран протоколом STP в качестве порта корневого моста на каждом коммутаторе некорневого моста? 
*S1 Fa0/3, S2 Fa0/3*

##### Почему протокол STP выбрал эти порты в качестве портов корневого моста на этих коммутаторах?
*Port ID*

Вопросы для повторения
---------

1.  Какое значение протокол STP использует первым после выбора корневого моста, чтобы определить выбор порта?

*Cost*

2.  Если первое значение на двух портах одинаково, какое следующее значение
    будет использовать протокол STP при выборе порта?

*Bridge ID*

3.  Если оба значения на двух портах равны, каким будет следующее значение,
    которое использует протокол STP при выборе порта?

*Port ID*
