---
id: 618
title: 'PXE Boot зависает после получения IP адреса от DHCP, &#8220;MTFTP&#8230;.&#8221;'
date: 2014-06-20T14:59:43+00:00
author: rootilo
layout: post
guid: http://4c74356b41.com/post618
permalink: /post618
categories:
  - SCCM
tags:
  - Configuration Manager
  - Copypaste
  - Guide
---
Если у вас не включен мультикаст в настройках PXE:
  
Administration >> Site >> Site Settings >> Site Systems >> Distribution Point - проверяем выключен ли мультикаст.
  
Software Library >> >> Operating System >> Operating System Images >> образ который используется для PXE >> убрана ли галочка &#8220;allow this package to be transferred via multicast&#8221;.

Если это так, проверяем DHCP scope и убираем опцию 43. Voila!