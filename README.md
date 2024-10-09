# Jmeter-dashboard-influxdb
Дашборд Grafana для мониторинга запусков сценариев НТ написанных с помощью инструмента нагрузочного
тестирования [Jmeter](https://jmeter.apache.org/).  
Отправка метрик осуществляется с помощью 
[Backend listener](https://jmeter.apache.org/usermanual/component_reference.html#Backend_Listener)
в InfluxDB (Query Language: InfluxQL).

Дашборд загружен в [Grafana Labs](https://grafana.com/grafana/dashboards/21818-jmeter-dashboard-influxdb/).

---
# Оглавление
* [С чего начать?](#begin)
* [Описание дашборда](#dashboardDescription)
  * [Run Test](#runTest)
  * [Summary](#summary)
  * [RPS / Response Times](#rps)
  * [Error](#error)
  * [Transaction Table](#transactionTable)
  * [Network Traffic](#network)
  * [Correlation](#correlation)
* [Пример запуска в Non-GUI mode](#example)

---
## С чего начать? <a id="begin"></a>
Инструкция перед началом эксплуатации:
1. Скачать и импортировать дашборд --> jmeter-dashboard-influxdb.json
2. Во время импорта указать параметры: InfluxDB, UID, dashboardName 
   * UID должен быть равен Unique identifier (UID)
   * dashboardName должен быть равен Name
4. Скачать пример скрипта Jmeter из репозитория --> jmeter-script/scriptExample.jmx
4. Внести несколько изменений в Backend Listener
   * **Backend Listener implementation = InfluxdbBackendListenerClient** - класс реализации BackendListenerClient
   * **application = ${__TestPlanName}** - название jmx-файла
   * **testTitle = ${__P(testTitle)}** - название теста (либо другая ~~полезная~~ информация)
   * **eventTags = ${__time()}** - уникальный RunID
   * **influxdbToken = {influxdbToken}** -
    influxdb [API Token](https://docs.influxdata.com/influxdb/cloud/admin/tokens/create-token/)

![Backend Listener - картинка](https://raw.githubusercontent.com/promokk/jmeter-dashboard-influxdb/main/data/Backend_Listener.png)

---
## Описание дашборда <a id="dashboardDescription"></a>
**ПРИМЕЧАНИЕ**
* На всех панелях одновременно отображается информация о запросах и транзакциях. Учитывайте это при
просмотре **_Summary_**.  
* Параметр _transaction_ позволяет выбрать необходимые запросы / транзакции для просмотра в **_RPS / Response Times_**.
Также во всех таблицах есть фильтры, которые позволяют сделать то же самое.  
* Использование _Assertion_ влияет на отображение ошибок 4xx и 5xx на панелях. В случае неудачной проверки
вместо кода ошибки будет отображаться сообщение - Assertion errors.
* Тест обязательно должен завершаться корректно. Либо по истечении времени, 
либо должен быть остановлен с помощью /jmeter/bin/stoptest.sh или shutdown.sh. 
Влияет на корректное отображение Status в таблице Run Test.

---
### Run Test <a id="runTest"></a>
Таблица Run Test - статус запущенных / завершенных тестов. Для работы данной таблицы нужно обязательно передавать
уникальное значение eventTags в Backend listener.
* Status - отображает состояние запуска, RUN или END. Запуск переходит в статус END при завершении теста.
* RunID - выводит значение eventTags из Backend listener. Значение RunID кликабельно. Открывает временной
интервал проводимого теста.
* Start Time - время начала теста.
* Application - наименование jmx-файла.
* Test - выводит значение testTitle из Backend listener.
* Duration - продолжительность теста.

![Run Test - гифка](https://raw.githubusercontent.com/promokk/jmeter-dashboard-influxdb/main/data/Run_Test.gif)

---
### Summary <a id="summary"></a>
Содержит основную информацию для первичного анализа:
* Активные / Завершившиеся потоки 
* Пропускная способность 
* Кол-во / Процент ошибок 
* Кол-во запросов 
* Сеть

![Summary - картинка](https://raw.githubusercontent.com/promokk/jmeter-dashboard-influxdb/main/data/Summary.png)

---
### RPS / Response Times <a id="rps"></a>
Отображает успешные и неудачные запросы, времена отклика. 
* Кол-во успешных / неудачных запросов в секунду 
* Время отклика успешных / неудачных запросов 
* Кол-во запросов 4xx и 5xx в секунду 
* Количество / Процент неудачных запросов по кодам

![RPS/Response Time - картинка](https://raw.githubusercontent.com/promokk/jmeter-dashboard-influxdb/main/data/RPS_Response_Time.png)

---
### Error <a id="error"></a>
Таблица Error Info - краткая информация об ошибках.  
Поле Response Message отображает наименование ошибки. Полное сообщение об ошибке (тело ответа) **не выводится**.

![Error - картинка](https://raw.githubusercontent.com/promokk/jmeter-dashboard-influxdb/main/data/Error.png)

---
### Transaction Table <a id="transactionTable"></a>
Таблица Transaction Table - статистика запросов / транзакций.
* Наименование запроса / транзакции 
* Кол-во запросов (общее / успешное / неудачное)
* Кол-во запросов в секунду (среднее / максимальное)
* Время отклика (95th percentile / 99th percentile)
* Процент неудачных запросов

![Transaction Table - картинка](https://raw.githubusercontent.com/promokk/jmeter-dashboard-influxdb/main/data/Transaction_Table.png)

---
### Network Traffic <a id="network"></a>
Полученные байты / отправленные байты

![Transaction Table - картинка](https://raw.githubusercontent.com/promokk/jmeter-dashboard-influxdb/main/data/Network_Traffic.png)

---
### Correlation <a id="correlation"></a>
Аннотация _Сorrelation_ позволяет поставить маркер на панели. Данный маркер будет отображаться на всех панелях.  
**Обязательное условие** - при создании аннотации необходимо указать tags = lt.

![Correlation - гифка](https://raw.githubusercontent.com/promokk/jmeter-dashboard-influxdb/main/data/Correlation.gif)

---
## Пример запуска в Non-GUI mode <a id="example"></a>
Пример для [распределенного запуска](https://jmeter.apache.org/usermanual/remote-test.html).  
Для распределенного запуска желательно использовать [mode](https://jmeter.apache.org/usermanual/remote-test.html#sendermode)
StrippedAsynch или Asynch - позволяет собирать данные в асинхронном режиме. Выборка данных будет точнее.  
С параметрами командной строки можно ознакомиться по [ссылке](https://jmeter.apache.org/usermanual/get-started.html#non_gui).

~~~shell
nohup /opt/jmeter/bin/jmeter -n -t scriptExample.jmx -X -R server01,server02 -Dmode=StrippedAsynch -DtestTitle=example > /dev/null 2>&1&
~~~
