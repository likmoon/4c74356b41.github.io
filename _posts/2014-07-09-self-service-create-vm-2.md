---
id: 1227
title: Self Service виртуальных машин. Создание виртуальной машины с портала SCSM. Часть 2
date: 2014-07-09T00:03:17+00:00
author: rootilo
layout: post
guid: http://4c74356b41.com/post1227
permalink: /post1227
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
Во второй части гайда мы обратим взоры на дочерние runbook&#8217;и.
  
1. [Создание Runbook&#8217;а](http://4c74356b41.com/post1176); (часть 1)
  
2. Создание дочерних Runbook&#8217;ов; (часть 2)
  
3. [Импорт Runbook&#8217;ов в SCSM](http://4c74356b41.com/post1261); (часть 3)
  
4. [Создание шаблонов для публикации на портале](http://4c74356b41.com/post1261); (часть 3)
  
5. [Подготовка Request Offering к публикации](http://4c74356b41.com/post1284); (часть 4)
  
6. [Публикация и проверка](http://4c74356b41.com/post1284). (часть 4)

Вначале я поясню, зачем нужны дочерние runbook&#8217;и. Изначально все мои ранбуки использовали просто шаг запуск виртуальной машины из интеграционного пакета VMM, но потом я понял, что данный шаг используется в половине runbook&#8217;ов, кроме того, мне показалось, что мои runbook&#8217;и &#8220;раздуваются&#8221; до неприличных размеров, и захотелось добавить в них модульность, так как модули по отдельности проще тестировать и можно использовать заново (советую использовать тот же подход для PowerShell скриптов 😉 ).

Подробное описание всей системы я выпишу в отдельный пост, потому сразу к делу.
  
Я использую всего 2 дочерних runbook&#8217;а. Это запуск виртуальной машины и запуск синхронизации коннектора SCSM.

**Запуск виртуальной машины**
  
Runbook выглядит вот так
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/module-runbooks-00.png" rel="attachment wp-att-5129"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/module-runbooks-00-300x175.png" alt="module-runbooks-00" width="300" height="175" /></a>
  
Начинаем, как обычно, с шага Initialize Data.
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/module-runbooks-02.png" rel="attachment wp-att-5136"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/module-runbooks-02-300x206.png" alt="module-runbooks-02" width="300" height="206" /></a>
  
Parent Runbook - будет получать значение {Runbook Process ID from &#8220;Initialize Data&#8221;} родительского Runbook&#8217;а
  
Reserved - получит значение &#8220;NewVM&#8221; из runbook&#8217;а &#8220;Create VM from Template&#8221;
  
VM ID - получит значение {VM ID from &#8220;Get VM&#8221;} родительского Runbook&#8217;а

Далее, как Вы заметили, runbook может &#8220;выполнить&#8221; два сценария. Для новой виртуальной машины или для существующей. Для этого в линке к следующему событию нужно указать условие Reserved from Initialize Data equals &#8220;NewVM&#8221; для сценария запуска новой виртуальной машины и Reserved from Initialize Data does not equals &#8220;NewVM&#8221; для сценария модификация VM.<a href="http://4c74356b41.com/wp-content/uploads/2016/02/module-runbooks-03.png" rel="attachment wp-att-5139"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/module-runbooks-03-300x196.png" alt="module-runbooks-03" width="300" height="196" /></a>

Следующими шагами будет получить данные о виртуальной машине и запустить её
  
**2. Get VM**
  
Action: Get VM
  
Connector: VMM Connector
  
Filter: {VM ID from &#8220;Initialize Data&#8221;}

**3. Start VM**
  
Action: Start VM
  
Connector: VMM Connector
  
Filter: {VM ID from &#8220;Get VM&#8221;}
  
Интересная ремарка, когда я пытался сделать тоже самое без шага &#8220;Get VM&#8221; шаг &#8220;Start VM&#8221; генерировал ошибку, у меня нет объяснения этому феномену, так как значения совпадают.

**4. Delay 90**
  
Action: Run .Net Script
  
Type: PowerShell
  
Script: Sleep 90
  
Ждем пока виртуальная машина загрузится и получит IP.

**5. Run VMM PowerShell Script**
  
Action: Run VMM PowerShell Script
  
Connector: VMM Connector
  
PowerShell: $a = Get-SCVirtualMachine {VM Name from &#8220;Get VM&#8221;} | Get-SCVirtualNetworkAdapter | %{$_.ipv4addresses}
  
Output: $a

**6. Return Data**
  
Action: Return Data (модуль Runbook Control)
  
IP: {Output Variable 01 from &#8220;Run VMM PowerShell Script&#8221;}
  
Для того чтобы вернуть данные в родительский runbook нужно настроить свойства дочернего runbook&#8217;а и добавить туда переменную &#8220;IP&#8221; и после этого создать данный шаг.
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/module-runbooks-04.png" rel="attachment wp-att-5142"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/module-runbooks-04-300x122.png" alt="module-runbooks-04" width="300" height="122" /></a>
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/module-runbooks-05.png" rel="attachment wp-att-5145"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/module-runbooks-05-300x205.png" alt="module-runbooks-05" width="300" height="205" /></a>
  
Значение переданное в родительский runbook можно использовать в родительском runbook&#8217;е в любом шаге после шага запуска дочернего runbook&#8217;а
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/module-runbooks-06.png" rel="attachment wp-att-5148"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/module-runbooks-06-300x261.png" alt="module-runbooks-06" width="300" height="261" /></a>
  
Вот шаг Launch VM из родительского runbook&#8217;а, укажите переменные и runbook (обратите внимание на галочку). Данные переменные будут доступны только после того как Вы создадите шаг Initialize Data в дочернем runbook&#8217;е, создадите в нем переменные и сохраните дочерний runbook.
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/module-runbooks-07.png" rel="attachment wp-att-5151"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/module-runbooks-07-300x206.png" alt="module-runbooks-07" width="300" height="206" /></a>

Если Вы все сделали правильно, родительский runbook будет запускать дочерний, ждать окончания runbook&#8217;а, получать из него IP адрес виртуальной машины и обновлять Service Request на портале этими данными.

**Запуск синхронизации коннектора SCSM-SCOM**
  
Runbook состоит из 1 действия &#8220;Run Program&#8221;, для того, чтобы это работало, необходим Service Manager Shell на сервере Orchestrator
  
cmd.exe /c | C:\Windows\system32\WindowsPowerShell\v1.0\powershell.exe –c  c:Start-SCOM-Sync.ps1
  
Содержимое скрипта (скрипт должен сам себя &#8220;убивать&#8221;)
  
Get-SCSMConnector -ComputerName lab-scsm-01 | where {$\_.displayname -Match &#8220;имя\_коннектора_scom&#8221;} | Start-SCSMConnector
  
$objCurrentPSProcess = [System.Diagnostics.Process]::GetCurrentProcess();
  
Stop-Process -Id $objCurrentPSProcess.ID;
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/module-runbooks-08.png" rel="attachment wp-att-5154"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/module-runbooks-08-300x206.png" alt="module-runbooks-08" width="300" height="206" /></a>
  
Тут необходимо пояснить: я создал данный runbook, так как хотел, чтобы в тестовой среде изменения отображались сразу после создания, изменения или удаления виртуальной машины. В &#8220;боевой&#8221; среде я бы не рекомендовал запускать синхронизацию после каждого запроса пользователя, так как это очень ресурсоемкая операция.
  
Вероятно, есть варианты заполнять CMDB &#8220;точечно&#8221;, т.е. брать сущность из VMM или SCOM и напрямую передавать в CMDB. Этот вариант был бы идеален, но по этому вопросу я Вам ничего поведать не могу, по крайней мере пока.