---
id: 167
title: Восстановление доступа к SCCM в случае если удален Full Adminstrator
date: 2013-10-10T22:23:20+00:00
author: rootilo
layout: post
guid: http://4c74356b41.com/post167
permalink: /post167
categories:
  - SCCM
tags:
  - Configuration Manager
---
Да, это случилось со мной 🙂 Я стер учетную запись Full Administrator SCCM из AD DS 🙂
  
Немного помучавшись с LDP.exe я понял что восстановить не получится, чтож будем искать другой путь.

И их есть у меня. Для начала нужен полный доступ на SQL базу данных, если нет получаем его как-то так

```
net stop MSSQLSERVER
net start MSSQLSERVER /m
osql -E -S .InstanceName -Q "EXEC sp_addsrvrolemember 'DOMUser', 'sysadmin'"
net stop MSSQLSERVER
net start MSSQLSERVER
```

За точность "скрипта"; не ручаюсь (выглядит правильно), но речь не об этом, как получить доступ к базе если его нет легко найти в интернете. Вопрос что делать когда доступ естьуже получен. Вводим в SQL следующую команду:

use CM_XXX - где XXX имя вашего сайта SCCM
  
select AdminID,AdminSID,LogonName,DisplayName from RBAC_Admins

В появившихся результатах находим удаленный аккаунт и запоминаем его AdminID, после чего идем и с помощью ADSIedit получаем hexadecimal значение ObjectSID пользователя который призван заменить данного индивидуама (я создал нового пользователя который назывался один в один как старый) и выполняем следующую команду:

```
use CM_XXX – где XXX имя вашего сайта SCCM
update dbo.RBAC_Admins
set AdminSID=0x010500000000000515000000F094C85F9A7CD63643170A3234200000 – 0x + шестандцатеричное значение атрибута ObjectSID нового пользователя
where AdminID=Значение AdminID стертой учетной записи
```

Чудо!