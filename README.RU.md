```markdown
# Решение проблемы с `polkit` и интерфейсом `tun` на Linux

## Проблема
При попытке настроить DNS, маршруты или MTU для интерфейса `neko-tun` в Linux система требует пароль, даже при использовании команды с `sudo`. Ошибка может выглядеть так:

```
Operator of unix-session:c1 FAILED to authenticate to gain authorization for action org.freedesktop.resolve1.set-dns-servers for system-bus-name::1.106 [/usr/bin/resolvectl dns neko-tun 172.19.0.2] (owned by unix-user:user)
```

Эта ошибка происходит из-за того, что для некоторых сетевых операций, например, изменения DNS или маршрутов, требуется дополнительная настройка прав доступа через **polkit**.

## Причина
**Polkit** (PolicyKit) — это система управления доступом в Linux, которая решает, кто и какие действия может выполнять на системе. Для некоторых операций, связанных с сетевыми интерфейсами, политику нужно явно разрешить, чтобы избежать постоянных запросов пароля.

## Решение
Чтобы устранить постоянные запросы пароля при выполнении сетевых настроек, необходимо создать специальное правило для **polkit**, которое позволит пользователю в группе `sudo` выполнять указанные действия без запроса пароля.

### Шаги для решения проблемы

1. **Создание правила для `polkit`**

   Откройте терминал и создайте файл `/etc/polkit-1/rules.d/99-tun.rules` с следующим содержимым:

   ```javascript
   polkit.addRule(function(action, subject) {
       if (action.id == "org.freedesktop.resolve1.set-dns-servers" ||
           action.id == "org.freedesktop.resolve1.set-domains" ||
           action.id == "org.freedesktop.resolve1.set-default-route" ||
           action.id == "org.freedesktop.network1.set-mtu") {
           if (subject.isInGroup("sudo")) {
               return polkit.Result.YES;
           }
       }
   });
   ```

   Этот код создает правило, разрешающее выполнение команд для изменения DNS, доменов, маршрутов и MTU интерфейса для пользователей, входящих в группу `sudo`.

2. **Установка прав на файл**

   После того как файл будет создан, установите правильные права для файла:

   ```bash
   sudo chmod 644 /etc/polkit-1/rules.d/99-tun.rules
   ```

   Это гарантирует, что файл будет доступен для чтения системой, но не позволит редактировать его без прав суперпользователя.

3. **Проверка работы**

   Теперь можно проверить, решена ли проблема. Попробуйте выполнить команду для настройки DNS или маршрутов:

   ```bash
   sudo resolvectl dns neko-tun 172.19.0.2
   sudo resolvectl domain neko-tun ~.
   sudo resolvectl default-route neko-tun true
   sudo resolvectl mtu neko-tun 1500
   ```

   Если все настроено правильно, система больше не должна запрашивать пароль.

## Дополнительные замечания

- В некоторых случаях изменения могут не вступить в силу сразу. Попробуйте перезагрузить систему или выйти и снова войти в сессию.
- Если вы хотите, чтобы правило применялось только для конкретных команд или интерфейсов, вы можете адаптировать код в файле правил.

## Заключение

Этот метод позволяет настроить систему так, чтобы действия с настройками интерфейса `neko-tun` не требовали постоянного ввода пароля. Это полезно, если нужно часто изменять сетевые настройки или работать с виртуальными интерфейсами.

## Лицензия

Этот репозиторий распространяется под лицензией **MIT**. Вы можете использовать, копировать, изменять и распространять его в рамках разрешений данной лицензии.
