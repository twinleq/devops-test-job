# Мониторинг процесса `test`

Скрипт `monitor-test.sh` проверяет наличие процесса `test`, стучится на `https://test.com/monitoring/test/api` и пишет диагностические сообщения в `/var/log/monitor-test/monitoring.log`. Запуск осуществляется через `systemd` при старте системы и далее каждые 60 секунд.

## Возможности
- периодический запуск через `systemd`-таймер;
- фиксация перезапуска процесса по времени старта `/proc/<pid>`;
- проверка доступности внешнего мониторинга по HTTPS;
- логирование перезапусков процесса и недоступности мониторинга.

## Требования
- системный пользователь `monitoring` без интерактивного шелла;
- каталоги `/var/lib/monitor-test` и `/var/log/monitor-test`, принадлежащие пользователю `monitoring`.

> Имена можно изменить, но не забудьте скорректировать unit-файл и переменные окружения для скрипта.

## Установка
1. Создайте пользователя и каталоги (однократно):
   ```bash
   sudo useradd --system --home /var/lib/monitor-test --shell /usr/sbin/nologin monitoring
   sudo install -d -o monitoring -g monitoring -m 0750 /var/lib/monitor-test /var/log/monitor-test
   ```
2. Установите скрипт:
   ```bash
   sudo install -m 0755 monitor-test.sh /usr/local/bin/monitor-test.sh
   ```
3. Скопируйте unit и timer:
   ```bash
   sudo install -m 0644 monitor-test.service /etc/systemd/system/monitor-test.service
   sudo install -m 0644 monitor-test.timer /etc/systemd/system/monitor-test.timer
   ```
4. Перечитайте конфигурацию и включите таймер:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable --now monitor-test.timer
   ```

## Проверка
- Разовый запуск: `sudo systemctl start monitor-test.service`.
- Расписание: `systemctl list-timers monitor-test.timer`.
- Логи: `sudo tail -f /var/log/monitoring.log`.

## Принцип работы и безопасность
- `pgrep -xo test` ищет текущий PID процесса.
- Если процесс отсутствует, скрипт завершается без действий.
- PID и время старта сохраняются в `STATE_FILE` (по умолчанию `/var/lib/monitor-test/test-monitor.state`).
- Изменение времени старта считается перезапуском и логируется.
- Ошибки `curl` к `https://test.com/monitoring/test/api` фиксируются в журнале.
- Скрипт работает от имени пользователя `monitoring`, поэтому root не требуется.
- Переменные `PROCESS_NAME`, `MONITOR_URL`, `STATE_FILE`, `LOG_FILE`, `CURL_TIMEOUT` могут быть переопределены через `Environment=` или `EnvironmentFile=` в unit-файле.
- При создании лог-файла используется `umask 077` и принудительный `chmod 600`, что исключает посторонний доступ к данным мониторинга.

