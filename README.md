# Мониторинг процесса `test`

Скрипт `monitor-test.sh` проверяет наличие процесса `test`, стучится на `https://test.com/monitoring/test/api` и пишет диагностические сообщения в `/var/log/monitoring.log`. Запуск осуществляется через `systemd` при старте системы и далее каждые 60 секунд.

## Возможности
- периодический запуск через `systemd`-таймер;
- фиксация перезапуска процесса по времени старта `/proc/<pid>`;
- проверка доступности внешнего мониторинга по HTTPS;
- логирование перезапусков процесса и недоступности мониторинга.

## Установка
1. Установите скрипт:
   ```bash
   sudo install -m 0755 monitor-test.sh /usr/local/bin/monitor-test.sh
   ```
2. Скопируйте unit и timer:
   ```bash
   sudo install -m 0644 monitor-test.service /etc/systemd/system/monitor-test.service
   sudo install -m 0644 monitor-test.timer /etc/systemd/system/monitor-test.timer
   ```
3. Перечитайте конфигурацию и включите таймер:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable --now monitor-test.timer
   ```

## Проверка
- Разовый запуск: `sudo systemctl start monitor-test.service`.
- Расписание: `systemctl list-timers monitor-test.timer`.
- Логи: `sudo tail -f /var/log/monitoring.log`.

## Принцип работы
- `pgrep -xo test` ищет текущий PID процесса.
- Если процесс отсутствует, скрипт завершается без действий.
- PID и время старта сохраняются в `/var/run/test-monitor.state`.
- Изменение времени старта считается перезапуском и логируется.
- Ошибки `curl` к `https://test.com/monitoring/test/api` фиксируются в журнале.

> Запускайте скрипт от пользователя с правами записи в `/var/log/monitoring.log` и `/var/run/` (обычно `root`).

