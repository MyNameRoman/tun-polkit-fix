(Если вы говорите по-русски, тут так же есть [README.RU.md](README.RU.md))

```markdown
# Solution for `polkit` and `tun` interface problem on Linux

## Problem
When trying to configure DNS, routes, or MTU for the `neko-tun` interface in Linux, the system asks for a password, even when using the command with `sudo`. The error might look like this:

```
Operator of unix-session:c1 FAILED to authenticate to gain authorization for action org.freedesktop.resolve1.set-dns-servers for system-bus-name::1.106 [/usr/bin/resolvectl dns neko-tun 172.19.0.2] (owned by unix-user:user)
```

This error occurs because certain network operations, such as changing DNS or routes, require additional access configuration through **polkit**.

## Cause
**Polkit** (PolicyKit) is an access control system in Linux that determines who can perform specific actions on the system. For certain operations related to network interfaces, the policy must be explicitly allowed to avoid constant password prompts.

## Solution
To eliminate constant password prompts during network configuration, you need to create a special rule for **polkit** that will allow users in the `sudo` group to perform the specified actions without being prompted for a password.

### Steps to resolve the issue

1. **Create a `polkit` rule**

   Open a terminal and create the file `/etc/polkit-1/rules.d/99-tun.rules` with the following content:

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

2. **Set file permissions**

   After the file is created, set the correct permissions for the file:

   ```bash
   sudo chmod 644 /etc/polkit-1/rules.d/99-tun.rules
   ```

3. **Test the solution**

   Now, check if the issue is resolved. Try running the command to configure DNS or routes:

   ```bash
   sudo resolvectl dns neko-tun 172.19.0.2
   sudo resolvectl domain neko-tun ~.
   sudo resolvectl default-route neko-tun true
   sudo resolvectl mtu neko-tun 1500
   ```

## Additional notes

- In some cases, changes might not take effect immediately. Try restarting your system or logging out and back in.
- If you want the rule to apply only to specific commands or interfaces, you can adjust the code in the rules file.

## Conclusion

This method allows you to configure the system so that actions with the `neko-tun` interface settings no longer require constant password input.

## License

This repository is licensed under the **MIT License**. You may use, copy, modify, and distribute it under the terms of this license.
