---
id: 3761
title: Не ставится Update Rollup на Exchange 2010\13\16
date: 2015-11-15T22:32:11+00:00
author: rootilo
layout: post
guid: http://4c74356b41.com/post3761
permalink: /post3761
categories:
  - random
tags:
  - Copypaste
  - Guide
---
Запускать от имена администратора (msiexec /update путь\_к\_пакету), выключить UAG, поставить execution policy unrestricted на время патча.
  
Еще вспомнил забавную проблему, обновление не ставилось, а в логе были ошибки про невозможность верифицировать керберос тикет от контроллера домена&#8230; поправил днс на сервере - все встало на свои места 🙂