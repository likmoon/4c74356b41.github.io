---
id: 1326
title: Self Service виртуальных машин. Удаление виртуальной машины с портала
date: 2014-07-10T11:18:30+00:00
author: rootilo
layout: post
guid: http://4c74356b41.com/post1326
permalink: /post1326
categories:
  - Virtualization and Cloud
tags:
  - Guide
  - Operations Manager
  - Orchestrator
  - Private Cloud VM Self Service
  - Service Manager
  - System Center 2012 R2
  - Virtual Machine Manager
---
Продолжаю цикл статей про самообслуживание пользователей. Перед прочтением этой статьи рекомендую ознакомиться с &#8220;[подготовительными процедурами](http://4c74356b41.com/post1139)&#8220;.

1. [Создание виртуальной машины](http://4c74356b41.com/post1176);
  
2. [Изменение виртуальной машины](http://4c74356b41.com/post1381);
  
3. Удаление виртуальной машины;
  
4. [Создание сервиса из шаблона сервиса](http://4c74356b41.com/post1062).

Итак, я предполагаю что у Вас есть SCSM, Orchestrator, SCOM и VMM, которые [настроены, интегрированы и работают](http://4c74356b41.com/post1139).
  
Если Вы еще не знакомы с Orchestrator, подробнее процесс создания runbook&#8217;а, аналогичного данному, описан в [другой статье.](http://4c74356b41.com/post1176)

Со стороны SCSM Вам нужен только Request Offering опубликованный на портале, создание его аналогично созданию [других Request Offering&#8217;ов для связки SCSM и Orchestrator](http://4c74356b41.com/post1284), поэтому его я описывать не буду. Перечислю только информацию, которую Вам нужно передать в Orchestrator. Виртуальную машину и подтверждение на удаление.

Со стороны Orchestrator Вам нужен Runbook (удивительно), который выглядит приблизительно вот так
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/pc-del-00.png" rel="attachment wp-att-5243"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/pc-del-00-300x205.png" alt="pc-del-00" width="300" height="205" /></a>

**Создаем Runbook**

**1. Initialize Data**
  
Создайте шаг и создайте переменные как на картинке. 😉
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/pc-del-01.png" rel="attachment wp-att-5247"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/pc-del-01-300x206.png" alt="pc-del-01" width="300" height="206" /></a>
  
Обращаю внимание, если Вы хотите чтобы runbook исполнялся только после подтверждения (в моем случае, на портале создан запрос на который пользователб должен ответить &#8220;Подтверждаю&#8221;) Вам необходимо настроить &#8220;Link&#8221; от шага &#8220;Initialize Data&#8221; до шага &#8220;Get SR&#8221;, где переменная &#8220;Confirm&#8221; равняется &#8220;Подтверждаю&#8221;
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/pc-del-02.png" rel="attachment wp-att-5250"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/pc-del-02-300x205.png" alt="pc-del-02" width="300" height="205" /></a>

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

**9. Remove VM**
  
Action: Remove VM
  
Connector: VMM Connector
  
Filter: VM ID equals {VM ID from &#8220;Get VM&#8221;}

Остальные шаги опциональны.