---
id: 1284
title: Self Service виртуальных машин. Создание виртуальной машины с портала SCSM. Часть 4
date: 2014-07-09T17:12:25+00:00
author: rootilo
layout: post
guid: http://4c74356b41.com/post1284
permalink: /post1284
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
Последняя статья из серии &#8220;Создание виртуальной машины с портала SCSM&#8221;.
  
1. [Создание Runbook&#8217;а](http://4c74356b41.com/post1176); (часть 1)
  
2. [Создание дочерних Runbook&#8217;ов](http://4c74356b41.com/post1227); (часть 2)
  
3. [Импорт Runbook&#8217;ов в SCSM](http://4c74356b41.com/post1261); (часть 3)
  
4. [Создание шаблонов для публикации на портале](http://4c74356b41.com/post1261); (часть 3)
  
5. Подготовка Request Offering к публикации; (часть 4)
  
6. Публикация и проверка. (часть 4)

**Публикуем Request Offering на портале SCSM**
  
После создания шаблона Service Request необходимо сделать Service Offering и Request Offering и опубликовать их на портале.
  
Для начала создайте Service Offering и заполните данные так как Вам необходимо (не забудьте опубликовать его)
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-00.png" rel="attachment wp-att-5375"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-00-300x134.png" alt="conf-scsm-00" width="300" height="134" /></a>
  
По сути, все что Вы там заполните не важно (в рамках теста), в целом эта информация не на что не влияет, но отображается на портале, для пользователей.
  
После этого создаем Request Offering
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-01.png" rel="attachment wp-att-5037"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-01-300x175.png" alt="conf-scsm-01" width="300" height="175" /></a>
  
На первом экране заполните название, описание и выберите шаблон Service Request&#8217;а, который мы создали в предыдущей статье
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-02.png" rel="attachment wp-att-5040"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-02-300x206.png" alt="conf-scsm-02" width="300" height="206" /></a>
  
На втором экране необходимо создать запросы, на которые должен ответить пользователь, чтобы создать виртуальную машину. Мы запросим у пользователя имя машины, шаблон, из которого будет создана машина, и облако, в котором она будет создана. Запрос имени должен иметь тип &#8220;Text&#8221;, а два других запроса должны иметь тип &#8220;Query Results&#8221;.
  
Вы можете так же запросить пароль администратора, к примеру, или делать ли виртуальную машину высокодоступной. Используйте фантазию!<a href="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-03.png" rel="attachment wp-att-5044"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-03-300x233.png" alt="conf-scsm-03" width="300" height="233" /></a>

Третий экран нужен для конфигурации запросов, к примеру, для первого запроса, рекомендую ограничить количество символов в имени виртуальной машины (15 максимум).
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-04.png" rel="attachment wp-att-5048"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-04-300x244.png" alt="conf-scsm-04" width="300" height="244" /></a>
  
Запрос шаблона и облака настроить сложнее, откройте свойства запроса двойным шелчком. В открывшемся окне выберите &#8220;All Basic Classes&#8221;, найдите и укажите &#8220;Virtual Machine Template&#8221; класс
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-05.png" rel="attachment wp-att-5052"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-05-300x260.png" alt="conf-scsm-05" width="300" height="260" /></a>
  
Далее Вам нужно будет указать фильтры для шаблона, к примеру у меня на стенде, у всех шаблонов для Self Service есть пометка &#8220;usage: SSP Deployments&#8221;
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-06.png" rel="attachment wp-att-5056"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-06-300x300.png" alt="conf-scsm-06" width="300" height="300" /></a>
  
На следующем экране Вы можете настроить информацию о шаблонах, которая будет предоставлена конечному пользователю. Имя, описание, количество ЦПУ, памяти и так далее. На последнем шаге Вы должны указать что делать с полученным объектом. В данном случае его необходимо прикрепить к Service Request&#8217;у как &#8220;Related Item&#8221;.
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-07.png" rel="attachment wp-att-5060"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-07-300x300.png" alt="conf-scsm-07" width="300" height="300" /></a>
  
Точно такую же процедуру нужно повторить с третьим запросом. Выбираете класс &#8220;Private Cloud&#8221;, фильтры, настраиваете представление и прикрепляете к Service Request&#8217;у как &#8220;Related Item&#8221;.

Шаг &#8220;Map Prompts&#8221; позволяет связать переменные Runbook&#8217;а с переменными, которые выбрал пользователь
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-09.png" rel="attachment wp-att-5064"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-09-300x233.png" alt="conf-scsm-09" width="300" height="233" /></a>

И осталось только опубликовать Request Offering и связать его с Service Offering
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-10.png" rel="attachment wp-att-5068"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-10-300x159.png" alt="conf-scsm-10" width="300" height="159" /></a>

Проверяем работоспособность
  
Заходим на портал и выбираем &#8220;наш&#8221; Service Offering
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-11.png" rel="attachment wp-att-5073"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-11-300x141.png" alt="conf-scsm-11" width="300" height="141" /></a>
  
Выбираем Request Offering
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-12.png" rel="attachment wp-att-5078"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-12-300x140.png" alt="conf-scsm-12" width="300" height="140" /></a>
  
Создаем Service Request
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-13.png" rel="attachment wp-att-5083"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-13-300x140.png" alt="conf-scsm-13" width="300" height="140" /></a>
  
Заполняем и отправляем
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-14.png" rel="attachment wp-att-5088"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-14-300x114.png" alt="conf-scsm-14" width="300" height="114" /></a>
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-15.png" rel="attachment wp-att-5093"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/conf-scsm-15-300x107.png" alt="conf-scsm-15" width="300" height="107" /></a>
  
Теперь осталось только дождаться исполнения runbook&#8217;а.
  
<a href="http://4c74356b41.com/wp-content/uploads/2016/02/success-newvm-00.png" rel="attachment wp-att-5352"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/success-newvm-00-300x226.png" alt="success-newvm-00" width="300" height="226" /></a>

<a href="http://4c74356b41.com/wp-content/uploads/2016/02/success-newvm-01.png" rel="attachment wp-att-5355"><img src="http://4c74356b41.com/wp-content/uploads/2016/02/success-newvm-01-300x114.png" alt="success-newvm-01" width="300" height="114" /></a>

Удачных разворачиваний!