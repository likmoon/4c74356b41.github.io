---
id: 532
title: Установка Service Management Automation для Windows Azure Pack
date: 2014-06-18T16:35:55+00:00
author: rootilo
layout: post
guid: http://4c74356b41.com/post532
permalink: /post532
categories:
  - Azure Pack
  - Virtualization and Cloud
tags:
  - Guide
  - Service Management Automation
  - SMA
  - System Center 2012 R2
  - WAPack Express
  - Windows Azure Pack
---
В данной статье я кратко опишу процесс установки Service Management Automation (SMA) для Windows Azure Pack.

**Что произойдет:**
  
1. Установка зависимостей;
  
2. Установка SQL Server 2012 SP1;
  
3. Установка SMA.

**Что Вам потребуется:**
  
1. Сервер с установленным на него Windows Server 2012 R2;
  
2. Дистрибутив SQL Server 2012 SP1;
  
3. Дистрибутив Orchestrator 2012 R2;
  
4. Доменная учетные записи sma-app-pool и sma-book-worker, доменная группы sma-admins;
  
5. Сертификат для веб-сервиса (опционально).

**Подготовка сервера к установке SMA:**
  
Перед установкой SMA необходимо установить зависимости, можно воспользоваться командой:
  
Install-WindowsFeature NET-WCF-HTTP-Activation45, Web-Windows-Auth, Web-Url-Auth, Web-Basic-Auth -IncludeAllSubFeature

**Установка SQL Server 2012 SP1 (или используйте уже существующую установку):**
  
Установите роль Database Engine Services (опционально Management Tools – Basic and Complete для работы с базой). Можно использовать любой поддерживаемый collation (SQL\_Latin1\_General\_CP1\_CI_AS по умолчанию). Сервисы SQL Agent и SQL Engine должны запускаться автоматически и использовать доменный RunAs Account. Далее-далее-далее, Готово.

**Установка SMA:**
  
С дистрибутива Ochestrator 2012 R2 запустите setup.exe, выберите установку Web Service в разделе Service Management Automation.
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/sma_install_02.png" rel="attachment wp-att-4871"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/sma_install_02-300x282.png" alt="sma_install_02" width="300" height="282" /></a>
  
Пройдите проверку зависимостей, если Вы что-то забыли, исправьте это. На следующем шаге укажите FQDN SQL сервера, где будет храниться база данных SMA. На следующем экране Вам необходимо будет сконфигурировать аккаунт для Application Pool&#8217;а IIS и группу для управления SMA.
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/sma_install_00.png" rel="attachment wp-att-4863"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/sma_install_00-300x227.png" alt="sma_install_00" width="300" height="227" /></a>
  
В качестве аккаунта для Application Pool&#8217;а укажите sma-app-pool, так же укажите группу для &#8220;Domain security group or users with access&#8221;, члены этой группы будет иметь доступ к SMA. Кроме того, Вам нужно будет указать сертификат и порт (9090 по умолчанию). Далее-далее-далее, Готово.
  
Но не совсем готово. 😉 Теперь Вам нужно еще раз запустить setup.exe и выбрать установку Runbook Worker в разделе Service Management Automation. Далее-далее-далее.
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/sma_install_01.png" rel="attachment wp-att-4867"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/sma_install_01-300x228.png" alt="sma_install_01" width="300" height="228" /></a>
  
Далее-далее-далее, Готово.
  
Можете так же установить Powershell.

О действиях необходимых для подключения SMA к Windows Azure Pack в [следующем посте](http://4c74356b41.com/post678).