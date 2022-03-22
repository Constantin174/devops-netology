# devops-netology

## Домашнее задание к занятию "6.6. Troubleshooting"

1) Для начала определим текущую операцию командой db.currentOp(). Затем, чтобы завершить ее, воспользуемся командой db.killOp(opid), где opid - идентификатор процесса.
Чтобы не было долгих запросов, возможно есть смысл выполнить перестроение соответствующего индекса.
2) Вероятно, тот момент, что Redis блокирует операции записи, может означать нехватку оперативной памяти, возможно, стоит масштабировать инстанс, добавив дополнительных ресурсов.
3) Вероятно, данная проблема связана со сбоями в сетевой инфраструктуре, высокой нагрузкой на сервер или с небольшим числом доступных соединений. Основные решения, позволяющие исправить данную:
    * слишком маленький таймаут операции, необходимо увеличить значения параметров connect_timeout, net_read_timeout, wait_timeout;
    * изменить параметр max_allowed_packet, т.к. на сервер могут поступать слишком большие коммуникационные пакеты (значения больше параметра max_allowed_packet);
    * добавить индексы для оптимизации выполнения запросов;
    * добавить ресурсов машине;
    * увеличить максимальное число соединений, используя параметр max_connections.
4) OOM-Killer - процесс, который необходим для завершения работы приложения в случае переполнения в нем оперативной памяти, чтобы спасти ОС от падения. Решением данной проблемы будет выставление в настройках ограничений на использование ресурсов хоста.