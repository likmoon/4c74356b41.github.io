---
id: 2541
title: Self Service виртуальных машин. Изменение виртуальной машины с портала
date: 2014-07-16T00:03:35+00:00
author: rootilo
layout: post
guid: http://4c74356b41.com/post1381
permalink: /post2541
categories:
  - Virtualization and Cloud
tags:
  - Guide
  - Operations Manager
  - Private Cloud VM Self Service
  - Service Manager
  - System Center 2012 R2
  - Virtual Machine Manager
---
Продолжаю цикл статей про самообслуживание пользователей. Перед прочтением этой статьи рекомендую ознакомиться с &#8220;[подготовительными процедурами](http://4c74356b41.com/post1139)&#8220;.

1. [Создание виртуальной машины](http://4c74356b41.com/post1176);
  
2. Изменение виртуальной машины;
  
3. [Удаление виртуальной машины](http://4c74356b41.com/post1326);
  
4. [Создание сервиса из шаблона сервиса](http://4c74356b41.com/post1062).

Итак, я предполагаю что у Вас есть SCSM, Orchestrator, SCOM и VMM, которые [настроены, интегрированы и работают](http://4c74356b41.com/post1139).
  
Если Вы еще не знакомы с Orchestrator, подробнее процесс создания runbook&#8217;а, аналогичного данному, описан в [другой статье.](http://4c74356b41.com/post1176)

Со стороны SCSM Вам нужен только Request Offering опубликованный на портале, создание его аналогично созданию [других Request Offering&#8217;ов для связки SCSM и Orchestrator](http://4c74356b41.com/post1284), поэтому его я описывать не буду. Перечислю только информацию, которую Вам нужно передать в Orchestrator. Виртуальную машину, подтверждение на возможность немедленной перезагрузки и, по желанию, количество процессоров, количество оперативной памяти, VLAN ID, Сколько ГБ места добавить на жесткий диск (или что угодно что придумает Ваша фантазия).

Со стороны Orchestrator Вам нужен Runbook (удивительно), который выглядит приблизительно вот так
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/pc-modifyvm-00.png" rel="attachment wp-att-5253"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/pc-modifyvm-00-300x145.png" alt="pc-modifyvm-00" width="300" height="145" /></a>

**Создаем Runbook**

**1. Initialize Data**
  
Создайте шаг и создайте переменные как на картинке. 😉
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/pc-modifyvm-01.png" rel="attachment wp-att-5258"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/pc-modifyvm-01-300x205.png" alt="pc-modifyvm-01" width="300" height="205" /></a>
  
Обращаю внимание, если Вы хотите чтобы виртуальная машина перезагружалась только после подтверждения (в моем случае, на портале создан запрос на который пользователь может ответить &#8220;Да&#8221; или &#8220;Нет&#8221;) Вам необходимо настроить &#8220;Link&#8221; от шага &#8220;Initialize Data&#8221; до шага &#8220;Get SR&#8221;, где переменная &#8220;Shall We Restart VM&#8221; ровняется &#8220;Да&#8221;.
  
В моем runbook&#8217;е, при выборе &#8220;Нет&#8221; запрос передается в другой runbook, который ждет следующего окна обслуживания и только тогда перегружает виртуальную машину. Это костыль. Правильнее сделать запись с VM ID в базу данных и runbook, который будет запускаться по рассписанию и из базы данных брать данные о виртуальных машинах и изменениях, которые необходимо к ним применить.
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/pc-modifyvm-02.png" rel="attachment wp-att-5261"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/pc-modifyvm-02-300x206.png" alt="pc-modifyvm-02" width="300" height="206" /></a>

**2. Get SR**
  
Этот шаг и далее, до шагов VMM, из SCSM integration pack.
  
Action: Get Object
  
Connector: SCSM Connector
  
Class: Service Request
  
Filter: &#8220;SC Object GUID&#8221; equals {SR GUID from &#8220;Initialize Data&#8221;}

**3. Get Relationship SR to VM**
  
Action: Get Relationsship
  
Connector: SCSM Connector
  
Object Class: Service Request
  
Object GUID: {SC Object GUID from &#8220;Get SR&#8221;}
  
Related Class: Virtual Machine

**4. Get Object - VM**
  
Action: Get Object
  
Connector: SCSM Connector
  
Class: Virtual Machine
  
Filter: &#8220;SC Object GUID&#8221; equals {Related object GUID from &#8220;Get Relationship SR to VM&#8221;}

**5. Get Relationship - Affected User**
  
Action: Get Relationsship
  
Connector: SCSM Connector
  
Object Class: Service Request
  
Object GUID: {SC Object GUID from &#8220;Get SR&#8221;}
  
Related Class: Active Directory User

**6. Get Object - Affected User**
  
Action: Get Object
  
Connector: SCSM Connector
  
Class: Service Templates
  
Filter: &#8220;SC Object GUID&#8221; equals {Related object GUID from &#8220;Get Relationship - Affected User&#8221;}

**7. Get VM**
  
Далее идут шаги VMM
  
Action: Get VM
  
Connector: VMM Connector
  
Filter: VM Name equals {Display Name from &#8220;Get Object - VM&#8221;}
  
Filter: Owner equals domain{User Name from &#8220;Get Object - Affected User&#8221;}

**8. Stop VM**
  
Action: Stop VM
  
Connector: VMM Connector
  
Filter: VM ID equals {VM ID from &#8220;Get VM&#8221;}

**9. Update VM**
  
Action: Update VM
  
Connector: VMM Connector
  
Filter: VM ID equals {VM ID from &#8220;Get VM&#8221;}
  
CPU Count: {Processor from &#8220;Initialize Data&#8221;}
  
Memory(MB): {RAM from &#8220;Initialize Data&#8221;}
  
Внимание, если у виртуальной машины стоит динамическая память, а runbook изменит значение Memory на значение больше чем максимальная память VM перейдет в состояние Failed. Кажется это &#8220;By Design&#8221;, шаг Update VM не умеет работать с динамической памятью. Workaround - PowerShell.

**10. Get Disk**
  
Action: Get Disk
  
Connector: VMM Connector
  
Filter: VM ID equals {VM ID from &#8220;Get VM&#8221;}

**11. Update Disk**
  
Action: Update Disk
  
Connector: VMM Connector
  
Filter: VM ID equals {Virtual Disk Drive ID from &#8220;Get Disk&#8221;}
  
Additional Size: {HDD from &#8220;Initialize Data&#8221;}

По аналогии Вы можете создать и шаги для изменения свойств адаптера.
  
Данные шаги не последовательны по причине того, что если шаг &#8220;Get Disk&#8221; или &#8220;Get Network Adapter&#8221; вернет 2 значения (или более), все следующие шаги будут повторятся несколько раз (по числу возвращенных объектов).

После этого можно передавать VM ID в Runbook &#8220;Launch VM&#8221;. Шаг &#8220;Launch VM&#8221; соответствует уже настроенному шагу &#8220;Launch VM&#8221; ([тут](http://4c74356b41.com/post1227)).

**Модификация Runbook&#8217;а &#8220;Launch VM&#8221;**
  
Если Вы не читали мы уже [создали](http://4c74356b41.com/post1227) этот runbook. Пришло время добавить в него шаги для сценария изменения VM.
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/module-runbooks-00.png" rel="attachment wp-att-5129"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/module-runbooks-00-300x175.png" alt="module-runbooks-00" width="300" height="175" /></a>
  
Содайте линк к событию &#8220;Delay 120&#8221;, а в линке задайте условие Reserved from Initialize Data does not equals &#8220;NewVM&#8221;
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/pc-modifyvm-03.png" rel="attachment wp-att-5264"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/pc-modifyvm-03-300x187.png" alt="pc-modifyvm-03" width="300" height="187" /></a>

**2. Delay 120**
  
Action: Run .Net Script
  
Type: PowerShell
  
Script: Sleep 120
  
Ждем пока VMM произведет изменения с дисками и сетевыми адаптерами виртуальной машины.

**3. Start VM**
  
Action: Start VM
  
Connector: VMM Connector
  
Filter: {VM ID from &#8220;Get VM&#8221;}