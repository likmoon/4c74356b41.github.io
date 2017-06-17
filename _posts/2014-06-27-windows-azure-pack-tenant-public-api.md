---
id: 864
title: Windows Azure Pack Tenant Public API
date: 2014-06-27T11:27:08+00:00
author: rootilo
layout: post
guid: http://4c74356b41.com/post864
permalink: /post864
categories:
  - Azure Pack
tags:
  - Guide
  - WAPack API
  - Windows Azure Pack
---
Windows Azure Pack предоставляет поддержку Powershell. Это позволяет тенантам использовать не портал, а PowerShell. Для чего это может понадобиться? Например, тестирование билдов или просто автоматизация процессов.

**Нам потребуется:**
  
1. [Web Platform Installer](http://www.microsoft.com/web/downloads/platform.aspx);
  
2. [Работающий WAP](http://4c74356b41.com/post422).

Роль Tenant Public API предоставляется непосредственно тенанту и должна находиться в DMZ. Обычно данную роль устанавливают на сервер WAP Tenant Site. Tenant Public API использует SLL и нуждается в отдельном endpoint'е. Для расположения её на сервере с Tenant Site необходимо воспользоваться функцией Host Headers (новая функция IIS 8.5). В [посте](http://4c74356b41.com/post835) про ADFS и WAP я уже описывал эту процедуру.

**Подготовка сервиса**
  
По умолчанию Tenant Public API "висит" на порте 30006, это, конечно, не очень удобно.
  
Для начала перенастроим привязку сайта в IIS Manager, при желании так же можно создать в DNS запись и сделать отдельный endpoint для API.
  
После этого необходимо изменить запись в базе данных WAP
  
Set-MgmtSvcFqdn –Namespace TenantPublicAPI –FQDN “api.tailspintoys.com” –Connectionstring “Data Source=wap-sql-01;Initial Catalog=Microsoft.MgmtSvc.Store;User Id=sa;Password=пароль” –Port 443

**Подготовка клиента**
  
Запустите WPI и установите "Windows Azure PowerShell". На моей системе (8.1 enterprise) WPI скачал 111мб, в интернете говорят что на серверные системы он качает всего ~10 мб. 😉 Оставим это на совести MSFT.
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/WAP_Tenant_API_00.png" rel="attachment wp-att-4987"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/WAP_Tenant_API_00-300x204.png" alt="WAP_Tenant_API_00" width="300" height="204" /></a>

WAP API использует аутентификацию по сертификатам. Портал генерирует сертификат для каждой подписки и хранит данные в БД. При аутентификации он сверяет ключи сертификатовю. Проверить сгенерированы ли ключи можно на портале.

<a href="http://4c74356b41.com/wp-content/uploads/2016/02/WAP_Tenant_API_01.png" rel="attachment wp-att-4991"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/WAP_Tenant_API_01-300x90.png" alt="WAP_Tenant_API_01" width="300" height="90" /></a>

Если у Вас нет сгенерированных сертификатов необходимо пройти по ссылке https://manage.tailspintoys.com/publishsettings
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/WAP_Tenant_API_02.png" rel="attachment wp-att-4996"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/WAP_Tenant_API_02-300x183.png" alt="WAP_Tenant_API_02" width="300" height="183" /></a>

Проверяем портал, на портале появился наш сертификат
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/WAP_Tenant_API_03.png" rel="attachment wp-att-5001"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/WAP_Tenant_API_03-300x78.png" alt="WAP_Tenant_API_03" width="300" height="78" /></a>

После этого необходимо импортировать сертификат (запустив Windows Azure Powershell)
  
Import-WAPackPublishSettingsFile "путь к скачанному файлу"
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/WAP_Tenant_API_04.png" rel="attachment wp-att-5006"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/WAP_Tenant_API_04-300x92.png" alt="WAP_Tenant_API_04" width="300" height="92" /></a>

На данный момент все настроено.

**Примеры использования**
  
Get-WAPackVM - "получить" виртуальные машины;
  
Get-WAPackVNet -  "получить" виртуальные сети;
  
Get-WAPackVMTemplate - "получить" шаблоны standalone виртуальных машин;

Ну и в целом Get-Command \*WAPack\* очень помогает. 😉

**Создаем Standalone виртуальную машину**
  
$VM = "Имя виртуальной машины"
  
$Template = Get-WAPackVMTemplate -Name "Имя шаблона виртуальной машины"
  
$VNet = Get-WAPackVNet -Name "hypervnu_network"
  
$pwd = Get-Credential
  
New-WAPackVM -Name $VM -Template $Template -VNet $VNet -VMCredential $pwd -Windows

**Управляем машиной**
  
Get-WAPackVM -Name "Имя виртуальной машины" | Start-WAPackVM
  
Get-WAPackVM -Name "Имя виртуальной машины" | Stop-WAPackVM
  
Get-WAPackVM -Name "Имя виртуальной машины" | Remove-WAPackVM

В WAPack Powershell пока еще нет командлетов для VM Roles, это можно обойти конвертировав запрос в JSON и передав на портал. Подробнее описано на [технете](http://blogs.technet.com/b/privatecloud/archive/2014/03/12/automation-the-new-world-of-tenant-provisioning-with-windows-azure-pack-part-3-automated-deployment-of-the-identity-workload-as-a-tenant-admin.aspx) сотрудником MSFT.

**Troubleshooting**
  
Для того чтобы "обнулить" конфигурацию
  
1. Сотрите сертификаты на портале
  
2. Удалите сертификаты с локальной машины из хранилища пользователя
  
3. Удалите WindowsAzureProfile.xml в папке %appdata%Windows Azure PowerShell
  
4. Заново сгенерируйте сертификаты и импортируйте их.

Если при попытке выполнить команду Вы получаете сообщение в духе "The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure  channel." Проверьте сертификат endpoint'а Tenant API (он должен быть доверенным).

Если Вы не можете развернуть машину, проверьте работает ли API в принципе, выполните какую-либо Get команду, например Get-WAPackVM;
  
Проверьте не совпадает ли имя разворачиваемой машины с уже существующей на портале машиной (в вашей подписке);
  
Если Вы разворачиваете роль виртуальной машины проверьте правильно ли Вы указали переменные для роли;
  
Если Вы разворачиваете роль виртуальной машины проверьте правильно ли Вы мапите параметры из powershell в JSON.