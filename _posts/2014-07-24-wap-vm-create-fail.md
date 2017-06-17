---
id: 1415
title: '&#8220;Failed to create virtual machine&#8221; при создании виртуальной машины в Windows Azure Pack'
date: 2014-07-24T23:00:38+00:00
author: rootilo
layout: post
guid: http://4c74356b41.com/post1415
permalink: /post1415
categories:
  - Azure Pack
tags:
  - Copypaste
  - Service Provider Foundation
  - SPF
  - Virtual Machine Manager
  - Windows Azure Pack
---
Failed to create virtual machine&#8217;vmname&#8217;.
  
Failed to submit operation request.

Ошибка подобная данной в логе Event ViewerApplications and Service LogsMicrosoft Windows ManagementODataServiceOperations

Log Name:   Microsoft-Windows-ManagementOdataService/Operational channel
  
EventID: 20006
  
Level: Error
  
Description: Web Service has got a callback from OData framework about an error.
  
Exception message = An error occurred while processing this request.
  
Inner exception message = The property &#8220;DynamicMemoryMinimumMB&#8221; does not exist on type &#8220;VMM.VirtualMachine&#8221;. Make sure to only use property names that are defined by the type.
  
Response status code = 400
  
Response content type = application/xml;charset=utf-8
  
Response written = false
  
Use verbose error = true

Это происходит потому что в WAP Rollup 2 добавили поддержку динамической памяти для виртуальных машин. То есть, Вы обновили WAP до Rollup 2 (или поставили его с Rollup 2), а VMM без Rollup 2. Необходимо поставить Rollup 2 на VMM.