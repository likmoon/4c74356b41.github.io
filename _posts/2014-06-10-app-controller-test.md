---
id: 399
title: Тестовая установка App Controller 2012 R2
date: 2014-06-10T22:10:59+00:00
author: rootilo
layout: post
guid: http://4c74356b41.com/post399
permalink: /post399
categories:
  - Virtualization and Cloud
tags:
  - App Controller
  - Guide
  - Quick Deploy
  - System Center 2012 R2
---
В данной статье я кратко опишу тестовую установку App Controller 2012 R2.

**Что произойдет:**
  
1. Установка зависимостей
  
2. Установка SQL Server 2012 SP1
  
3. Установка App Controller 2012 R2

**Что Вам потребуется:**
  
1. Сервер с установленным на него Windows Server 2012 R2
  
2. Дистрибутив VMM 2012 R2
  
3. Дистрибутив App Controller 2012 R2
  
4. Дистрибутив SQL Server 2012 SP1;
  
5. Сертификат для веб портала App Controller.

**Подготовка сервера к установке App Controller:**
  
Необходимо установить консоль VMM, путем запуска Setup.exe с дистрибутива VMM и нажатия на кнопку далее некое количество раз. После этого можно подготовить сертификат для веб портала (если у вас есть PKI). Заходим в IIS Manager >> открываем Server Certificates локального компьютера >> запросить доменный сертификат. В качестве Common Name рекомендуется использовать FQDN веб портала App Controller.

**Установка SQL Server 2012 SP1:**
  
Установите роль Database Engine Services (опционально Management Tools – Basic and Complete для работы с базой). Можно использовать любой поддерживаемый collation (SQL\_Latin1\_General\_CP1\_CI_AS по умолчанию). Сервисы SQL Agent и SQL Engine должны запускаться автоматически и использовать доменный RunAs Account. Настройки на экранах, не упомянутых в гайде, Вы можете выставить такие, какие Вам необходимы. Далее-далее-далее, Готово.

**Установка App Controller 2012 R2:**
  
Данное действие необходимо выпольнять пользователем с правами VMM Admin.
  
Установка данного продукта тривиальна и Вам фактически не нужно ничего изменять. Укажите лицензионный ключ, сервисный аккаунт (желательно доменный), в качестве сервера баз данных укажите FQDN компьютера на котором происходит установка, поле порт оставьте пустым. Далее-далее-далее, Готово.

**Настройка App Controller:**
  
Данное действие необходимо выпольнять пользователем с правами VMM Admin.
  
Зайдите не веб портал App Controller (<https://fqdn>) после авторизации нажмите Settings > Connection > Connect. Укажите FQDN сервера VMM.

Идем в облака - <http://technet.microsoft.com/en-us/library/hh221344.aspx>