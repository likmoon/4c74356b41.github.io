---
id: 3501
title: Nested Virtualization в Hyper-V, ура товарищи!
date: 2015-10-14T10:52:09+00:00
author: rootilo
layout: post
guid: http://4c74356b41.com/post3501
permalink: /post3501
categories:
  - Virtualization and Cloud
tags:
  - Copypaste
  - Hyper-V
---
http://blogs.technet.com/b/virtualization/archive/2015/10/13/windows-insider-preview-nested-virtualization.aspx

Пока еще с &#8220;конскими&#8221; ограничениями, но что же делать 😉 

Like I said earlier – this is still just a “preview” of this feature. Obviously, this feature should not be used in production environments.  Below is a list of known issues:

  * **Both hypervisors need to be the latest versions of Hyper-V. Other hypervisors will not work. Windows Server 2012R2, or even builds prior to 10565 will not work.**
  * Once nested virtualization is enabled in a VM, the following features are no longer compatible with that VM. These actions will either fail, or cause the VM not to start: 
      * Dynamic memory must be OFF. This will prevent the VM from booting.
      * Runtime memory resize will fail.
      * Applying checkpoints to a running VM will fail.
      * Live migration will fail.
      * Save/restore will fail.
  * Once nested virtualization is enabled in a VM, MAC spoofing must be enabled for networking to work in its guests.
  * Hosts with Virtualization Based Security (VBS) enabled cannot expose virtualization extensions to guests. You must first disable VBS in order to preview nested virtualization.
  * This feature is currently Intel-only. Intel VT-x is required.
  * Beware: nested virtualization requires a good amount of memory. I managed to run a VM in a VM with 4 GB of host RAM, but things were tight.