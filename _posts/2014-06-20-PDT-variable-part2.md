---
id: 593
title: '"Сердце" PDT - Variable.xml (часть 2)'
date: 2014-06-20T12:44:17+00:00
author: rootilo
layout: post
guid: http://4c74356b41.com/post593
permalink: /post593
categories:
  - Azure Pack
  - Virtualization and Cloud
tags:
  - Guide
  - PDT
  - Powershell Deployment Toolkit
  - System Center 2012 R2
  - Windows Azure Pack
---
**Продолжим изучение Variable.xml.**
  
В комплекте PDT идут 2 Variable.xml файла, один для установки в существующее AD, другой для создания разворачивания с нуля.
  
На практике, чтобы превратить один в другой достаточно добавить (или убрать) кусок xml файла где описывается домен:

<Domain>
  
<Name>contoso.com</Name>
  
<ServiceAccountOU>Users</ServiceAccountOU>
  
<GroupOU>Groups</GroupOU>
  
</Domain>

Как происходит кастомизация VM мы вкратце рассмотрели в [предыдущей статье](http://4c74356b41.com/post546). Общая идея такова, все что Вы не укажите в свойствах конкретной виртуальной машины (после тега <VM>) будет взято из тега <Default>. Пару примеров.
  
Если какую-то машину необходимо сделать машиной первого поколения внутри тега машины необходимо вставить "<VMGeneration>1</VMGeneration>". Не забудьте указать ему отдельный родительский диск (тк виртуальные машины первого поколения не могут грузиться с iSCSI):
  
<VM>
  
<VM Count="номер">
  
<VMName>Имя</VMName>
  
<VMGeneration>1</VMGeneration>
  
<OSDisk>
  
<Parent>путь до vhd(x)</Parent>
  
<Type>Differencing</Type>
  
</OSDisk>
  
</VM>

Как разместить виртуальную машину на другом хосте:
  
<VM>
  
<VM Count="номер">
  
<Host>имя хоста</Host>
  
</VM>

Словом \_все\_ что идет внутри тега <Default> можно изменить для каждой конкретной виртуальной машины.

**Немного мыслей про SQL:**
  
Если Вы хотите развернуть какую-то роль на существующем SQL сервере:
  
<Role Name="System Center 2012 R2 Virtual Machine Manager Database Server" Server="VMMDB.contoso.com" Instance="MSSQLSERVER" Existing="True">

Если Installer.ps1 или VMCreator.ps1 при проверке ругаются, что порт не задан, его можно задать добавив Port="number" в описание инстанса:
  
<Instance Server="SQL-01.contoso.com" Instance="MSSQLSERVER" Version="SQL Server 2012&#8243; Port="number">

Если Вы хотите развернуть какую-то роль на кластере SQL Вам необходимо добавить к описанию роли SQLCluster="True":
  
<Role Name="System Center 2012 R2 Virtual Machine Manager Database Server" Server="SQL-01.contoso.com" Instance="MSSQLSERVER" SQLCluster="True"></Role>
  
<Role Name="SQL Server 2012 Management Tools" Server="SQL-01.contoso.com" SQLCluster="True"></Role>

И соответственно заполнить описание компонентов роли SQL:
  
<Cluster Cluster="SQL-01.contoso.com" Version="SQL Server 2012&#8243;
  
<Variable Name="SQLUserDBDir" Value="D:MSSQL11.$InstanceMSSQLData" /> - и поправить эти пути на кластерные
  
<Variable Name="SQLUserDBLogDir" Value="E:MSSQL11.$InstanceMSSQLData" />
  
<Variable Name="SQLTempDBDir" Value="F:MSSQL11.$InstanceMSSQLData" />
  
<Variable Name="SQLTempDBLogDir" Value="F:MSSQL11.$InstanceMSSQLData" />
  
<Variable Name="SQLClusterIPAddress"
  
<Variable Name="SQLClusterIPAddress" Value=""IP />
  
<Variable Name="SQLClusterNetwork" Value="Cluster Network 1&#8243; />
  
<Variable Name="SQLClusterSubnet" Value="Subnet"/>
  
<Node Server="FQDN" Preferred "1"></Node>
  
<Node Server="FQDN" Preferred "2"></Node>

Но это только пол пути 😉
  
После создания виртуальных машин, Вам нужно остановить Installer.ps1 на домен контроллере,создать руками Failover Cluster, создать сетевые шары для него, настроить file share witness, дать кластерному аккаунту доступ Full Control в OU кластера и объекты в этом OU (или Full Control на OU и VCOCNO), дать ему и аккаунту инсталлера полный доступ на сетевые и кластерные "шары" и после этого запустить Installer.ps1 заново.

В случае установки SCOM DB и SCOM DW DB на кластер необходимо устанавливать их на одну и ту же ноду кластера (это ограничение SCOM), а для установки репортинга на этот же SCOM, необходимо ставить его на ту же ноду где уже находится SCOM Management сервер. Те указать просто тот же сервер (не ноду и не кластер):
  
<Role Name="System Center 2012 R2 Operations Manager Management Server" Server="OM1.contoso.com"></Role>
  
<Role Name="System Center 2012 R2 Operations Manager Reporting Server" Server="OM1.contoso.com" Instance="MSSQLSERVER"></Role>

**Несколько случайных мыслей:**
  
- Ключ "WindowsAzurePack2013ConfigStorePassphrase", который используется для задания фразы пароля в Windows Azure Pack, не понимает символа _;
  
- SCCM не может быть установлен на SQL с включенными динамическими портами;
  
- В случае если при валидации конфигурации выдается ошибка можно проверить Variable.xml загрузив его в Powershell $Variable = \[XML\] (Get-Content .Variable.xml). Powershell сообщит на какой строке ошибка;
  
- При указании NETBIOS имени домена для добавления машин, машины могут не проходить фазу Specialize бесконечно пытаясь вступить в домен;
  
- В случае проблем с unattend.xml проще всего искать причины на самой виртуалке нажав Shift+F10, и далее через блокнот. %windir%Panther и %windir%PantherUnnatendGC файлы setupact.log и setuperr.log. Unnattend.xml в папке %windir%Panther;
  
- При создании виртуальных машин из файла Variable.xml с не доменной рабочей станции и без создания домена виртуальные машины могут не проходить фазу OOBE из-за того, что в unnattend.xml, в качестве одного из администраторов операционной системы, будет указан пользователь запустивший скрипт, и, естественн,о он не будет добавлен, так как не будет найден. Workaround - затереть кусок кода, который добавляет это в unattend.xml в скрипте VMCreator.ps1 (1437 строка, кусок после $env:UserDomain -ne $InstallerServiceAccountDomain, необходимо стереть все после @" и до "@):
  
If ($env:UserDomain -ne $InstallerServiceAccountDomain) {
  
@"
  
"@ | Out-File "$Drive\`:unattend.xml" -Append -Encoding ASCII

В последнем из запланированных постов я расскажу в общем о процессе развертывания и подкину еще пару мыслей на тему PDT.

ПС. Наткнулся тут на интересный пост о PDT. [Создание среды для лабораторных работ](http://social.technet.microsoft.com/wiki/contents/articles/22048.creating-system-center-vms-using-pdt-for-hybrid-scenario.aspx) по гибридному облаку (<a href="http://aka.ms/hybridlabs" target="_blank">http://aka.ms/hybridlabs</a>).