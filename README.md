# Домашнее задание к занятию "08.02 Работа с Playbook"

## Подготовка к выполнению

1. (Необязательно) Изучите, что такое [clickhouse](https://www.youtube.com/watch?v=fjTNS2zkeBs) и [vector](https://www.youtube.com/watch?v=CgEhyffisLY)  
2. Создайте свой собственный (или используйте старый) публичный репозиторий на github с произвольным именем.  
3. Скачайте [playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.  
4. Подготовьте хосты в соответствии с группами из предподготовленного playbook.  
**Сделано**  

## Основная часть

1. Приготовьте свой собственный inventory файл `prod.yml`.
```
---
clickhouse:
  hosts:
    clickhouse-01:
      ansible_host: 10.50.41.139
vector:
  hosts:
    vector-01:
      ansible_host: 10.50.41.139
```

2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает [vector](https://vector.dev).
Есть. Несколько моментов:  
- Добавил таску рестарта хендлера раньше времени:
```
    - name: Flush handlers
      meta: flush_handlers
```

Добавил ожидание, т.к. видимо сервис не успевал поднятся и сваливался в ошибку:
```
   - name: wait 10 sec
      ansible.builtin.pause:
        seconds: 10
```

3. При создании tasks рекомендую использовать модули: `get_url`, `template`, `unarchive`, `file`.
4. Tasks должны: скачать нужной версии дистрибутив, выполнить распаковку в выбранную директорию, установить vector.
```
root@ansible-test:~/08-ansible-02-playbook-1/playbook# ansible-playbook -i inventory/prod.yml site.yml -vv
ansible-playbook 2.10.8
  config file = None
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible-playbook
  python version = 3.10.6 (main, Nov 14 2022, 16:10:14) [GCC 11.3.0]
No config file found; using defaults
Skipping callback 'default', as we already have a stdout callback.
Skipping callback 'minimal', as we already have a stdout callback.
Skipping callback 'oneline', as we already have a stdout callback.

PLAYBOOK: site.yml **************************************************************************************************************************************************************************************************************************
2 plays in site.yml

PLAY [Install Clickhouse] *******************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook-1/playbook/site.yml:2
ok: [clickhouse-01]
META: ran handlers

TASK [Get clickhouse distrib] ***************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook-1/playbook/site.yml:14
changed: [clickhouse-01] => (item=clickhouse-client) => {"ansible_loop_var": "item", "changed": true, "checksum_dest": null, "checksum_src": "f6f57a5e156c972f68a6fb983cc29ab7189cb14a", "dest": "./clickhouse-client-22.3.3.44.rpm", "elapsed": 0, "gid": 0, "group": "root", "item": "clickhouse-client", "md5sum": "e8b2834873a9f7cfaaf9f59761aa68bc", "mode": "0644", "msg": "OK (38090 bytes)", "owner": "root", "secontext": "unconfined_u:object_r:admin_home_t:s0", "size": 38090, "src": "/root/.ansible/tmp/ansible-tmp-1677487660.6200006-27763-117882767550904/tmp85zf1P", "state": "file", "status_code": 200, "uid": 0, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-client-22.3.3.44.noarch.rpm"}
changed: [clickhouse-01] => (item=clickhouse-server) => {"ansible_loop_var": "item", "changed": true, "checksum_dest": null, "checksum_src": "153ece14c0fab3caa0442f0fcb0175e717e498e1", "dest": "./clickhouse-server-22.3.3.44.rpm", "elapsed": 0, "gid": 0, "group": "root", "item": "clickhouse-server", "md5sum": "23484986857a1632bb3003b1fd56a4d0", "mode": "0644", "msg": "OK (61151 bytes)", "owner": "root", "secontext": "unconfined_u:object_r:admin_home_t:s0", "size": 61151, "src": "/root/.ansible/tmp/ansible-tmp-1677487662.129157-27763-139950689243101/tmp1w3WBU", "state": "file", "status_code": 200, "uid": 0, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-server-22.3.3.44.noarch.rpm"}
failed: [clickhouse-01] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static-22.3.3.44.rpm", "elapsed": 0, "item": "clickhouse-common-static", "msg": "Request failed", "response": "HTTP Error 404: Not Found", "status_code": 404, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-22.3.3.44.noarch.rpm"}

TASK [Get clickhouse distrib] ***************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook-1/playbook/site.yml:20
changed: [clickhouse-01] => {"changed": true, "checksum_dest": null, "checksum_src": "e9c3bdddfc9dd66b170903231d11d0afc598d9e1", "dest": "./clickhouse-common-static-22.3.3.44.rpm", "elapsed": 49, "gid": 0, "group": "root", "md5sum": "6a5edc2173d44d505f47a4522819810d", "mode": "0644", "msg": "OK (246310036 bytes)", "owner": "root", "secontext": "unconfined_u:object_r:admin_home_t:s0", "size": 246310036, "src": "/root/.ansible/tmp/ansible-tmp-1677487664.7517128-27786-140641610886589/tmpE_n8AG", "state": "file", "status_code": 200, "uid": 0, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-22.3.3.44.x86_64.rpm"}

TASK [Install clickhouse packages] **********************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook-1/playbook/site.yml:24
NOTIFIED HANDLER Start clickhouse service for clickhouse-01
changed: [clickhouse-01] => {"changed": true, "changes": {"installed": ["clickhouse-common-static-22.3.3.44.rpm", "clickhouse-client-22.3.3.44.rpm", "clickhouse-server-22.3.3.44.rpm"]}, "msg": "", "rc": 0, "results": ["Loaded plugins: fastestmirror, product-id, search-disabled-repos, subscription-\n              : manager\n\nThis system is not registered with an entitlement server. You can use subscription-manager to register.\n\nExamining clickhouse-common-static-22.3.3.44.rpm: clickhouse-common-static-22.3.3.44-1.x86_64\nMarking clickhouse-common-static-22.3.3.44.rpm to be installed\nExamining clickhouse-client-22.3.3.44.rpm: clickhouse-client-22.3.3.44-1.noarch\nMarking clickhouse-client-22.3.3.44.rpm to be installed\nExamining clickhouse-server-22.3.3.44.rpm: clickhouse-server-22.3.3.44-1.noarch\nMarking clickhouse-server-22.3.3.44.rpm to be installed\nResolving Dependencies\n--> Running transaction check\n---> Package clickhouse-client.noarch 0:22.3.3.44-1 will be installed\n---> Package clickhouse-common-static.x86_64 0:22.3.3.44-1 will be installed\n---> Package clickhouse-server.noarch 0:22.3.3.44-1 will be installed\n--> Finished Dependency Resolution\n\nDependencies Resolved\n\n================================================================================\n Package           Arch   Version     Repository                           Size\n================================================================================\nInstalling:\n clickhouse-client noarch 22.3.3.44-1 /clickhouse-client-22.3.3.44        123 k\n clickhouse-common-static\n                   x86_64 22.3.3.44-1 /clickhouse-common-static-22.3.3.44 725 M\n clickhouse-server noarch 22.3.3.44-1 /clickhouse-server-22.3.3.44        195 k\n\nTransaction Summary\n================================================================================\nInstall  3 Packages\n\nTotal size: 725 M\nInstalled size: 725 M\nDownloading packages:\nRunning transaction check\nRunning transaction test\nTransaction test succeeded\nRunning transaction\nCannot set 'net_admin' or 'ipc_lock' or 'sys_nice' capability for clickhouse binary. This is optional. Taskstats accounting will be disabled. To enable taskstats accounting you may add the required capability later manually.\nClickHouse binary is already located at /usr/bin/clickhouse\nSymlink /usr/bin/clickhouse-server already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.\nCreating symlink /usr/bin/clickhouse-server to /usr/bin/clickhouse.\nCreating symlink /usr/bin/clickhouse-client to /usr/bin/clickhouse.\nCreating symlink /usr/bin/clickhouse-local to /usr/bin/clickhouse.\nCreating symlink /usr/bin/clickhouse-benchmark to /usr/bin/clickhouse.\nSymlink /usr/bin/clickhouse-copier already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.\nCreating symlink /usr/bin/clickhouse-copier to /usr/bin/clickhouse.\nCreating symlink /usr/bin/clickhouse-obfuscator to /usr/bin/clickhouse.\nCreating symlink /usr/bin/clickhouse-git-import to /usr/bin/clickhouse.\nCreating symlink /usr/bin/clickhouse-compressor to /usr/bin/clickhouse.\nCreating symlink /usr/bin/clickhouse-format to /usr/bin/clickhouse.\nSymlink /usr/bin/clickhouse-extract-from-config already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.\nCreating symlink /usr/bin/clickhouse-extract-from-config to /usr/bin/clickhouse.\nSymlink /usr/bin/clickhouse-keeper already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.\nCreating symlink /usr/bin/clickhouse-keeper to /usr/bin/clickhouse.\nCreating symlink /usr/bin/clickhouse-keeper-converter to /usr/bin/clickhouse.\nCreating clickhouse group if it does not exist.\n groupadd -r clickhouse\nCreating clickhouse user if it does not exist.\n useradd -r --shell /bin/false --home-dir /nonexistent -g clickhouse clickhouse\nWill set ulimits for clickhouse user in /etc/security/limits.d/clickhouse.conf.\nCreating config directory /etc/clickhouse-server/config.d that is used for tweaks of main server configuration.\nCreating config directory /etc/clickhouse-server/users.d that is used for tweaks of users configuration.\nConfig file /etc/clickhouse-server/config.xml already exists, will keep it and extract path info from it.\n/etc/clickhouse-server/config.xml has /var/lib/clickhouse/ as data path.\n/etc/clickhouse-server/config.xml has /var/log/clickhouse-server/ as log path.\nUsers config file /etc/clickhouse-server/users.xml already exists, will keep it and extract users info from it.\nCreating log directory /var/log/clickhouse-server/.\nCreating data directory /var/lib/clickhouse/.\nCreating pid directory /var/run/clickhouse-server.\n chown -R clickhouse:clickhouse '/var/log/clickhouse-server/'\n chown -R clickhouse:clickhouse '/var/run/clickhouse-server'\n chown  clickhouse:clickhouse '/var/lib/clickhouse/'\n groupadd -r clickhouse-bridge\n useradd -r --shell /bin/false --home-dir /nonexistent -g clickhouse-bridge clickhouse-bridge\n chown -R clickhouse-bridge:clickhouse-bridge '/usr/bin/clickhouse-odbc-bridge'\n chown -R clickhouse-bridge:clickhouse-bridge '/usr/bin/clickhouse-library-bridge'\n\u001b[1mPassword for default user is empty string. See /etc/clickhouse-server/users.xml and /etc/clickhouse-server/users.d to change it.\u001b[0m\nSetting capabilities for clickhouse binary. This is optional.\n chown -R clickhouse:clickhouse '/etc/clickhouse-server'\n\nClickHouse has been successfully installed.\n\nStart clickhouse-server with:\n sudo clickhouse start\n\nStart clickhouse-client with:\n clickhouse-client\n\n\n  Verifying  : clickhouse-server-22.3.3.44-1.noarch                         1/3 \n  Verifying  : clickhouse-common-static-22.3.3.44-1.x86_64                  2/3 \n  Verifying  : clickhouse-client-22.3.3.44-1.noarch                         3/3 \n\nInstalled:\n  clickhouse-client.noarch 0:22.3.3.44-1                                        \n  clickhouse-common-static.x86_64 0:22.3.3.44-1                                 \n  clickhouse-server.noarch 0:22.3.3.44-1                                        \n\nComplete!\n"]}

RUNNING HANDLER [Start clickhouse service] **************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook-1/playbook/site.yml:5
changed: [clickhouse-01] => {"changed": true, "name": "clickhouse-server", "state": "started", "status": {"ActiveEnterTimestampMonotonic": "0", "ActiveExitTimestampMonotonic": "0", "ActiveState": "inactive", "After": "system.slice basic.target time-sync.target network-online.target systemd-journald.socket", "AllowIsolate": "no", "AmbientCapabilities": "0", "AssertResult": "no", "AssertTimestampMonotonic": "0", "Before": "shutdown.target", "BlockIOAccounting": "no", "BlockIOWeight": "18446744073709551615", "CPUAccounting": "no", "CPUQuotaPerSecUSec": "infinity", "CPUSchedulingPolicy": "0", "CPUSchedulingPriority": "0", "CPUSchedulingResetOnFork": "no", "CPUShares": "18446744073709551615", "CanIsolate": "no", "CanReload": "no", "CanStart": "yes", "CanStop": "yes", "CapabilityBoundingSet": "8409088", "CollectMode": "inactive", "ConditionResult": "no", "ConditionTimestampMonotonic": "0", "Conflicts": "shutdown.target", "ControlPID": "0", "DefaultDependencies": "yes", "Delegate": "no", "Description": "ClickHouse Server (analytic DBMS for big data)", "DevicePolicy": "auto", "EnvironmentFile": "/etc/default/clickhouse (ignore_errors=yes)", "ExecMainCode": "0", "ExecMainExitTimestampMonotonic": "0", "ExecMainPID": "0", "ExecMainStartTimestampMonotonic": "0", "ExecMainStatus": "0", "ExecStart": "{ path=/usr/bin/clickhouse-server ; argv[]=/usr/bin/clickhouse-server --config=/etc/clickhouse-server/config.xml --pid-file=/run/clickhouse-server/clickhouse-server.pid ; ignore_errors=no ; start_time=[n/a] ; stop_time=[n/a] ; pid=0 ; code=(null) ; status=0/0 }", "FailureAction": "none", "FileDescriptorStoreMax": "0", "FragmentPath": "/usr/lib/systemd/system/clickhouse-server.service", "Group": "clickhouse", "GuessMainPID": "yes", "IOScheduling": "0", "Id": "clickhouse-server.service", "IgnoreOnIsolate": "no", "IgnoreOnSnapshot": "no", "IgnoreSIGPIPE": "yes", "InactiveEnterTimestampMonotonic": "0", "InactiveExitTimestampMonotonic": "0", "JobTimeoutAction": "none", "JobTimeoutUSec": "0", "KillMode": "control-group", "KillSignal": "15", "LimitAS": "18446744073709551615", "LimitCORE": "18446744073709551615", "LimitCPU": "18446744073709551615", "LimitDATA": "18446744073709551615", "LimitFSIZE": "18446744073709551615", "LimitLOCKS": "18446744073709551615", "LimitMEMLOCK": "65536", "LimitMSGQUEUE": "819200", "LimitNICE": "0", "LimitNOFILE": "500000", "LimitNPROC": "15063", "LimitRSS": "18446744073709551615", "LimitRTPRIO": "0", "LimitRTTIME": "18446744073709551615", "LimitSIGPENDING": "15063", "LimitSTACK": "18446744073709551615", "LoadState": "loaded", "MainPID": "0", "MemoryAccounting": "no", "MemoryCurrent": "18446744073709551615", "MemoryLimit": "18446744073709551615", "MountFlags": "0", "Names": "clickhouse-server.service", "NeedDaemonReload": "no", "Nice": "0", "NoNewPrivileges": "no", "NonBlocking": "no", "NotifyAccess": "none", "OOMScoreAdjust": "0", "OnFailureJobMode": "replace", "PermissionsStartOnly": "no", "PrivateDevices": "no", "PrivateNetwork": "no", "PrivateTmp": "no", "ProtectHome": "no", "ProtectSystem": "no", "RefuseManualStart": "no", "RefuseManualStop": "no", "RemainAfterExit": "no", "Requires": "basic.target system.slice network-online.target", "Restart": "always", "RestartUSec": "30s", "Result": "success", "RootDirectoryStartOnly": "no", "RuntimeDirectory": "clickhouse-server", "RuntimeDirectoryMode": "0755", "SameProcessGroup": "no", "SecureBits": "0", "SendSIGHUP": "no", "SendSIGKILL": "yes", "Slice": "system.slice", "StandardError": "inherit", "StandardInput": "null", "StandardOutput": "journal", "StartLimitAction": "none", "StartLimitBurst": "5", "StartLimitInterval": "10000000", "StartupBlockIOWeight": "18446744073709551615", "StartupCPUShares": "18446744073709551615", "StatusErrno": "0", "StopWhenUnneeded": "no", "SubState": "dead", "SyslogLevelPrefix": "yes", "SyslogPriority": "30", "SystemCallErrorNumber": "0", "TTYReset": "no", "TTYVHangup": "no", "TTYVTDisallocate": "no", "TasksAccounting": "no", "TasksCurrent": "18446744073709551615", "TasksMax": "18446744073709551615", "TimeoutStartUSec": "1min 30s", "TimeoutStopUSec": "1min 30s", "TimerSlackNSec": "50000", "Transient": "no", "Type": "simple", "UMask": "0022", "UnitFilePreset": "disabled", "UnitFileState": "disabled", "User": "clickhouse", "Wants": "time-sync.target", "WatchdogTimestampMonotonic": "0", "WatchdogUSec": "0"}}
META: ran handlers

TASK [wait 10 sec] **************************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook-1/playbook/site.yml:36
Pausing for 10 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [clickhouse-01] => {"changed": false, "delta": 10, "echo": true, "rc": 0, "start": "2023-02-27 08:49:09.304758", "stderr": "", "stdout": "Paused for 10.0 seconds", "stop": "2023-02-27 08:49:19.305365", "user_input": ""}

TASK [Create database] **********************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook-1/playbook/site.yml:40
changed: [clickhouse-01] => {"changed": true, "cmd": ["clickhouse-client", "-q", "create database logs;"], "delta": "0:00:00.149489", "end": "2023-02-27 08:55:50.597456", "failed_when_result": false, "rc": 0, "start": "2023-02-27 08:55:50.447967", "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}
META: ran handlers
META: ran handlers

PLAY [Install Vector] ***********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook-1/playbook/site.yml:46
ok: [vector-01]
META: ran handlers

TASK [Vector Install package] ***************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook-1/playbook/site.yml:57
NOTIFIED HANDLER restart vector service for vector-01
changed: [vector-01] => {"changed": true, "changes": {"installed": ["/root/.ansible/tmp/ansible-tmp-1677487762.577346-27838-183700954787451/vector-0.21.1-1.x86_64R5NTZZ.rpm"]}, "msg": "", "rc": 0, "results": ["Loaded plugins: fastestmirror, product-id, search-disabled-repos, subscription-\n              : manager\n\nThis system is not registered with an entitlement server. You can use subscription-manager to register.\n\nExamining /root/.ansible/tmp/ansible-tmp-1677487762.577346-27838-183700954787451/vector-0.21.1-1.x86_64R5NTZZ.rpm: vector-0.21.1-1.x86_64\nMarking /root/.ansible/tmp/ansible-tmp-1677487762.577346-27838-183700954787451/vector-0.21.1-1.x86_64R5NTZZ.rpm to be installed\nResolving Dependencies\n--> Running transaction check\n---> Package vector.x86_64 0:0.21.1-1 will be installed\n--> Finished Dependency Resolution\n\nDependencies Resolved\n\n================================================================================\n Package     Arch        Version       Repository                          Size\n================================================================================\nInstalling:\n vector      x86_64      0.21.1-1      /vector-0.21.1-1.x86_64R5NTZZ      154 M\n\nTransaction Summary\n================================================================================\nInstall  1 Package\n\nTotal size: 154 M\nInstalled size: 154 M\nDownloading packages:\nRunning transaction check\nRunning transaction test\nTransaction test succeeded\nRunning transaction\n  Installing : vector-0.21.1-1.x86_64                                       1/1 \n  Verifying  : vector-0.21.1-1.x86_64                                       1/1 \n\nInstalled:\n  vector.x86_64 0:0.21.1-1                                                      \n\nComplete!\n"]}

TASK [Configure Service | Template systemd unit] ********************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook-1/playbook/site.yml:64
changed: [vector-01] => {"changed": true, "checksum": "7628541ee1ca342db39db772cd6aa730aae0374e", "dest": "/etc/systemd/system/vector.service", "gid": 0, "group": "root", "md5sum": "1965c7068200ab49d4222c99ec0cc10d", "mode": "0644", "owner": "root", "secontext": "system_u:object_r:systemd_unit_file_t:s0", "size": 248, "src": "/root/.ansible/tmp/ansible-tmp-1677487781.4904106-27847-2639372831485/source", "state": "file", "uid": 0}

RUNNING HANDLER [restart vector service] ****************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook-1/playbook/site.yml:49
changed: [vector-01] => {"changed": true, "name": "vector", "state": "started", "status": {"ActiveEnterTimestampMonotonic": "0", "ActiveExitTimestampMonotonic": "0", "ActiveState": "inactive", "After": "systemd-journald.socket system.slice basic.target network.target", "AllowIsolate": "no", "AmbientCapabilities": "0", "AssertResult": "no", "AssertTimestampMonotonic": "0", "Before": "shutdown.target", "BlockIOAccounting": "no", "BlockIOWeight": "18446744073709551615", "CPUAccounting": "no", "CPUQuotaPerSecUSec": "infinity", "CPUSchedulingPolicy": "0", "CPUSchedulingPriority": "0", "CPUSchedulingResetOnFork": "no", "CPUShares": "18446744073709551615", "CanIsolate": "no", "CanReload": "no", "CanStart": "yes", "CanStop": "yes", "CapabilityBoundingSet": "18446744073709551615", "CollectMode": "inactive", "ConditionResult": "no", "ConditionTimestampMonotonic": "0", "Conflicts": "shutdown.target", "ControlPID": "0", "DefaultDependencies": "yes", "Delegate": "no", "Description": "Vector Service", "DevicePolicy": "auto", "ExecMainCode": "0", "ExecMainExitTimestampMonotonic": "0", "ExecMainPID": "0", "ExecMainStartTimestampMonotonic": "0", "ExecMainStatus": "0", "ExecStart": "{ path=/usr/bin/vector ; argv[]=/usr/bin/vector --config-yaml /root/vector_config/vector.yml --watch-config true ; ignore_errors=no ; start_time=[n/a] ; stop_time=[n/a] ; pid=0 ; code=(null) ; status=0/0 }", "FailureAction": "none", "FileDescriptorStoreMax": "0", "FragmentPath": "/etc/systemd/system/vector.service", "Group": "0", "GuessMainPID": "yes", "IOScheduling": "0", "Id": "vector.service", "IgnoreOnIsolate": "no", "IgnoreOnSnapshot": "no", "IgnoreSIGPIPE": "yes", "InactiveEnterTimestampMonotonic": "0", "InactiveExitTimestampMonotonic": "0", "JobTimeoutAction": "none", "JobTimeoutUSec": "0", "KillMode": "control-group", "KillSignal": "15", "LimitAS": "18446744073709551615", "LimitCORE": "18446744073709551615", "LimitCPU": "18446744073709551615", "LimitDATA": "18446744073709551615", "LimitFSIZE": "18446744073709551615", "LimitLOCKS": "18446744073709551615", "LimitMEMLOCK": "65536", "LimitMSGQUEUE": "819200", "LimitNICE": "0", "LimitNOFILE": "4096", "LimitNPROC": "15063", "LimitRSS": "18446744073709551615", "LimitRTPRIO": "0", "LimitRTTIME": "18446744073709551615", "LimitSIGPENDING": "15063", "LimitSTACK": "18446744073709551615", "LoadState": "loaded", "MainPID": "0", "MemoryAccounting": "no", "MemoryCurrent": "18446744073709551615", "MemoryLimit": "18446744073709551615", "MountFlags": "0", "Names": "vector.service", "NeedDaemonReload": "no", "Nice": "0", "NoNewPrivileges": "no", "NonBlocking": "no", "NotifyAccess": "none", "OOMScoreAdjust": "0", "OnFailureJobMode": "replace", "PermissionsStartOnly": "no", "PrivateDevices": "no", "PrivateNetwork": "no", "PrivateTmp": "no", "ProtectHome": "no", "ProtectSystem": "no", "RefuseManualStart": "no", "RefuseManualStop": "no", "RemainAfterExit": "no", "Requires": "basic.target network-online.target system.slice", "Restart": "always", "RestartUSec": "100ms", "Result": "success", "RootDirectoryStartOnly": "no", "RuntimeDirectoryMode": "0755", "SameProcessGroup": "no", "SecureBits": "0", "SendSIGHUP": "no", "SendSIGKILL": "yes", "Slice": "system.slice", "StandardError": "inherit", "StandardInput": "null", "StandardOutput": "journal", "StartLimitAction": "none", "StartLimitBurst": "5", "StartLimitInterval": "10000000", "StartupBlockIOWeight": "18446744073709551615", "StartupCPUShares": "18446744073709551615", "StatusErrno": "0", "StopWhenUnneeded": "no", "SubState": "dead", "SyslogLevelPrefix": "yes", "SyslogPriority": "30", "SystemCallErrorNumber": "0", "TTYReset": "no", "TTYVHangup": "no", "TTYVTDisallocate": "no", "TasksAccounting": "no", "TasksCurrent": "18446744073709551615", "TasksMax": "18446744073709551615", "TimeoutStartUSec": "1min 30s", "TimeoutStopUSec": "1min 30s", "TimerSlackNSec": "50000", "Transient": "no", "Type": "simple", "UMask": "0022", "UnitFilePreset": "disabled", "UnitFileState": "static", "User": "root", "WatchdogTimestampMonotonic": "0", "WatchdogUSec": "0"}}
META: ran handlers
META: ran handlers

PLAY RECAP **********************************************************************************************************************************************************************************************************************************
clickhouse-01              : ok=6    changed=4    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0
vector-01                  : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.
Были лишние пробелы (исправлено):
```
yaml: wrong indentation: expected 8 but found 10 (indentation)
site.yml:52
```

6. Попробуйте запустить playbook на этом окружении с флагом `--check`.
7. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.
8. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.
```

root@ansible-test:~/08-ansible-02-playbook-1/playbook# ansible-playbook -i inventory/prod.yml site.yml -vv --diff
ansible-playbook 2.10.8
  config file = None
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible-playbook
  python version = 3.10.6 (main, Nov 14 2022, 16:10:14) [GCC 11.3.0]
No config file found; using defaults
Skipping callback 'default', as we already have a stdout callback.
Skipping callback 'minimal', as we already have a stdout callback.
Skipping callback 'oneline', as we already have a stdout callback.

PLAYBOOK: site.yml **************************************************************************************************************************************************************************************************************************
2 plays in site.yml

PLAY [Install Clickhouse] *******************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook-1/playbook/site.yml:2
ok: [clickhouse-01]
META: ran handlers

TASK [Get clickhouse distrib] ***************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook-1/playbook/site.yml:14
ok: [clickhouse-01] => (item=clickhouse-client) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-client-22.3.3.44.rpm", "elapsed": 0, "gid": 0, "group": "root", "item": "clickhouse-client", "mode": "0644", "msg": "HTTP Error 304: Not Modified", "owner": "root", "secontext": "unconfined_u:object_r:admin_home_t:s0", "size": 38090, "state": "file", "status_code": 304, "uid": 0, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-client-22.3.3.44.noarch.rpm"}
ok: [clickhouse-01] => (item=clickhouse-server) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-server-22.3.3.44.rpm", "elapsed": 0, "gid": 0, "group": "root", "item": "clickhouse-server", "mode": "0644", "msg": "HTTP Error 304: Not Modified", "owner": "root", "secontext": "unconfined_u:object_r:admin_home_t:s0", "size": 61151, "state": "file", "status_code": 304, "uid": 0, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-server-22.3.3.44.noarch.rpm"}
failed: [clickhouse-01] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static-22.3.3.44.rpm", "elapsed": 0, "gid": 0, "group": "root", "item": "clickhouse-common-static", "mode": "0644", "msg": "Request failed", "owner": "root", "response": "HTTP Error 404: Not Found", "secontext": "unconfined_u:object_r:admin_home_t:s0", "size": 246310036, "state": "file", "status_code": 404, "uid": 0, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-22.3.3.44.noarch.rpm"}

TASK [Get clickhouse distrib] ***************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook-1/playbook/site.yml:20
ok: [clickhouse-01] => {"changed": false, "dest": "./clickhouse-common-static-22.3.3.44.rpm", "elapsed": 0, "gid": 0, "group": "root", "mode": "0644", "msg": "HTTP Error 304: Not Modified", "owner": "root", "secontext": "unconfined_u:object_r:admin_home_t:s0", "size": 246310036, "state": "file", "status_code": 304, "uid": 0, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-22.3.3.44.x86_64.rpm"}

TASK [Install clickhouse packages] **********************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook-1/playbook/site.yml:24
ok: [clickhouse-01] => {"changed": false, "msg": "", "rc": 0, "results": ["clickhouse-common-static-22.3.3.44-1.x86_64 providing clickhouse-common-static-22.3.3.44.rpm is already installed", "clickhouse-client-22.3.3.44-1.noarch providing clickhouse-client-22.3.3.44.rpm is already installed", "clickhouse-server-22.3.3.44-1.noarch providing clickhouse-server-22.3.3.44.rpm is already installed"]}
META: ran handlers

TASK [wait 10 sec] **************************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook-1/playbook/site.yml:36
Pausing for 10 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [clickhouse-01] => {"changed": false, "delta": 10, "echo": true, "rc": 0, "start": "2023-02-27 10:47:28.291191", "stderr": "", "stdout": "Paused for 10.0 seconds", "stop": "2023-02-27 10:47:38.292207", "user_input": ""}

TASK [Create database] **********************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook-1/playbook/site.yml:40
ok: [clickhouse-01] => {"changed": false, "cmd": ["clickhouse-client", "-q", "create database logs;"], "delta": "0:00:00.159782", "end": "2023-02-27 09:06:58.273875", "failed_when_result": false, "msg": "non-zero return code", "rc": 82, "start": "2023-02-27 09:06:58.114093", "stderr": "Received exception from server (version 22.3.3):\nCode: 82. DB::Exception: Received from localhost:9000. DB::Exception: Database logs already exists.. (DATABASE_ALREADY_EXISTS)\n(query: create database logs;)", "stderr_lines": ["Received exception from server (version 22.3.3):", "Code: 82. DB::Exception: Received from localhost:9000. DB::Exception: Database logs already exists.. (DATABASE_ALREADY_EXISTS)", "(query: create database logs;)"], "stdout": "", "stdout_lines": []}
META: ran handlers
META: ran handlers

PLAY [Install Vector] ***********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook-1/playbook/site.yml:46
ok: [vector-01]
META: ran handlers

TASK [Vector Install package] ***************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook-1/playbook/site.yml:57
ok: [vector-01] => {"changed": false, "msg": "", "rc": 0, "results": ["vector-0.21.1-1.x86_64 providing /root/.ansible/tmp/ansible-tmp-1677494861.5802689-29712-234394909484202/vector-0.21.1-1.x86_64u68Zcs.rpm is already installed"]}

TASK [Configure Vector | ensure what directory exists] **************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook-1/playbook/site.yml:64
ok: [vector-01] => {"changed": false, "gid": 0, "group": "root", "mode": "0644", "owner": "root", "path": "/root/vector_config/", "secontext": "unconfined_u:object_r:admin_home_t:s0", "size": 24, "state": "directory", "uid": 0}

TASK [Configure Vector | Template config] ***************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook-1/playbook/site.yml:70
ok: [vector-01] => {"changed": false, "checksum": "73809b9097ffb445e1bb79b309365cb082a109d9", "dest": "/root/vector_config//vector.yml", "gid": 0, "group": "root", "mode": "0644", "owner": "root", "path": "/root/vector_config//vector.yml", "secontext": "system_u:object_r:admin_home_t:s0", "size": 13, "state": "file", "uid": 0}

TASK [Configure Service | Template systemd unit] ********************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook-1/playbook/site.yml:76
ok: [vector-01] => {"changed": false, "checksum": "ab51618d25f807192bbde7598341b6ed9df09847", "dest": "/etc/systemd/system/vector.service", "gid": 0, "group": "root", "mode": "0644", "owner": "root", "path": "/etc/systemd/system/vector.service", "secontext": "system_u:object_r:systemd_unit_file_t:s0", "size": 249, "state": "file", "uid": 0}
META: ran handlers
META: ran handlers

PLAY RECAP **********************************************************************************************************************************************************************************************************************************
clickhouse-01              : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0
vector-01                  : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

9. Подготовьте README.md файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.
Сделано  

10. Готовый playbook выложите в свой репозиторий, поставьте тег `08-ansible-02-playbook` на фиксирующий коммит, в ответ предоставьте ссылку на него.  
https://github.com/aleks-sh-devops/08-ansible-02-playbook

## Доработка  
**Про теги**  
Добавил в плейбук теги. Они позволяют более гибко пользоваться ансиблом. Например, -skip-tags позволяет нам не выполнять определенные части плейбука (в моем случае я скипнул создание logs и вектор:
```
root@ansible-test:~/08-ansible-02-playbook/playbook# ansible-playbook -i inventory/prod.yml site.yml -vv --skip-tags tag2,tag3
ansible-playbook 2.10.8
  config file = None
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible-playbook
  python version = 3.10.6 (main, Nov 14 2022, 16:10:14) [GCC 11.3.0]
No config file found; using defaults
Skipping callback 'default', as we already have a stdout callback.
Skipping callback 'minimal', as we already have a stdout callback.
Skipping callback 'oneline', as we already have a stdout callback.

PLAYBOOK: site.yml **************************************************************************************************************************************************************************************************************************
2 plays in site.yml

PLAY [Install Clickhouse] *******************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:2
ok: [clickhouse-01]
META: ran handlers

TASK [Get clickhouse distrib] ***************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:14
changed: [clickhouse-01] => (item=clickhouse-client) => {"ansible_loop_var": "item", "changed": true, "checksum_dest": null, "checksum_src": "f6f57a5e156c972f68a6fb983cc29ab7189cb14a", "dest": "./clickhouse-client-22.3.3.44.rpm", "elapsed": 1, "gid": 0, "group": "root", "item": "clickhouse-client", "md5sum": "e8b2834873a9f7cfaaf9f59761aa68bc", "mode": "0644", "msg": "OK (38090 bytes)", "owner": "root", "secontext": "unconfined_u:object_r:admin_home_t:s0", "size": 38090, "src": "/root/.ansible/tmp/ansible-tmp-1677501211.471111-30293-47055735499740/tmpT335o_", "state": "file", "status_code": 200, "uid": 0, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-client-22.3.3.44.noarch.rpm"}
changed: [clickhouse-01] => (item=clickhouse-server) => {"ansible_loop_var": "item", "changed": true, "checksum_dest": null, "checksum_src": "153ece14c0fab3caa0442f0fcb0175e717e498e1", "dest": "./clickhouse-server-22.3.3.44.rpm", "elapsed": 0, "gid": 0, "group": "root", "item": "clickhouse-server", "md5sum": "23484986857a1632bb3003b1fd56a4d0", "mode": "0644", "msg": "OK (61151 bytes)", "owner": "root", "secontext": "unconfined_u:object_r:admin_home_t:s0", "size": 61151, "src": "/root/.ansible/tmp/ansible-tmp-1677501214.3699749-30293-204287424963597/tmpKTjdwN", "state": "file", "status_code": 200, "uid": 0, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-server-22.3.3.44.noarch.rpm"}
failed: [clickhouse-01] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static-22.3.3.44.rpm", "elapsed": 0, "item": "clickhouse-common-static", "msg": "Request failed", "response": "HTTP Error 404: Not Found", "status_code": 404, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-22.3.3.44.noarch.rpm"}

TASK [Get clickhouse distrib] ***************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:22
changed: [clickhouse-01] => {"changed": true, "checksum_dest": null, "checksum_src": "e9c3bdddfc9dd66b170903231d11d0afc598d9e1", "dest": "./clickhouse-common-static-22.3.3.44.rpm", "elapsed": 61, "gid": 0, "group": "root", "md5sum": "6a5edc2173d44d505f47a4522819810d", "mode": "0644", "msg": "OK (246310036 bytes)", "owner": "root", "secontext": "unconfined_u:object_r:admin_home_t:s0", "size": 246310036, "src": "/root/.ansible/tmp/ansible-tmp-1677501217.8890707-30316-502225855840/tmpglGQnI", "state": "file", "status_code": 200, "uid": 0, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-22.3.3.44.x86_64.rpm"}

TASK [Install clickhouse packages] **********************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:29
NOTIFIED HANDLER Start clickhouse service for clickhouse-01
changed: [clickhouse-01] => {"changed": true, "changes": {"installed": ["clickhouse-common-static-22.3.3.44.rpm", "clickhouse-client-22.3.3.44.rpm", "clickhouse-server-22.3.3.44.rpm"]}, "msg": "", "rc": 0, "results": ["Loaded plugins: fastestmirror, product-id, search-disabled-repos, subscription-\n              : manager\n\nThis system is not registered with an entitlement server. You can use subscription-manager to register.\n\nExamining clickhouse-common-static-22.3.3.44.rpm: clickhouse-common-static-22.3.3.44-1.x86_64\nMarking clickhouse-common-static-22.3.3.44.rpm to be installed\nExamining clickhouse-client-22.3.3.44.rpm: clickhouse-client-22.3.3.44-1.noarch\nMarking clickhouse-client-22.3.3.44.rpm to be installed\nExamining clickhouse-server-22.3.3.44.rpm: clickhouse-server-22.3.3.44-1.noarch\nMarking clickhouse-server-22.3.3.44.rpm to be installed\nResolving Dependencies\n--> Running transaction check\n---> Package clickhouse-client.noarch 0:22.3.3.44-1 will be installed\n---> Package clickhouse-common-static.x86_64 0:22.3.3.44-1 will be installed\n---> Package clickhouse-server.noarch 0:22.3.3.44-1 will be installed\n--> Finished Dependency Resolution\n\nDependencies Resolved\n\n================================================================================\n Package           Arch   Version     Repository                           Size\n================================================================================\nInstalling:\n clickhouse-client noarch 22.3.3.44-1 /clickhouse-client-22.3.3.44        123 k\n clickhouse-common-static\n                   x86_64 22.3.3.44-1 /clickhouse-common-static-22.3.3.44 725 M\n clickhouse-server noarch 22.3.3.44-1 /clickhouse-server-22.3.3.44        195 k\n\nTransaction Summary\n================================================================================\nInstall  3 Packages\n\nTotal size: 725 M\nInstalled size: 725 M\nDownloading packages:\nRunning transaction check\nRunning transaction test\nTransaction test succeeded\nRunning transaction\nCannot set 'net_admin' or 'ipc_lock' or 'sys_nice' capability for clickhouse binary. This is optional. Taskstats accounting will be disabled. To enable taskstats accounting you may add the required capability later manually.\nClickHouse binary is already located at /usr/bin/clickhouse\nSymlink /usr/bin/clickhouse-server already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.\nCreating symlink /usr/bin/clickhouse-server to /usr/bin/clickhouse.\nCreating symlink /usr/bin/clickhouse-client to /usr/bin/clickhouse.\nCreating symlink /usr/bin/clickhouse-local to /usr/bin/clickhouse.\nCreating symlink /usr/bin/clickhouse-benchmark to /usr/bin/clickhouse.\nSymlink /usr/bin/clickhouse-copier already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.\nCreating symlink /usr/bin/clickhouse-copier to /usr/bin/clickhouse.\nCreating symlink /usr/bin/clickhouse-obfuscator to /usr/bin/clickhouse.\nCreating symlink /usr/bin/clickhouse-git-import to /usr/bin/clickhouse.\nCreating symlink /usr/bin/clickhouse-compressor to /usr/bin/clickhouse.\nCreating symlink /usr/bin/clickhouse-format to /usr/bin/clickhouse.\nSymlink /usr/bin/clickhouse-extract-from-config already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.\nCreating symlink /usr/bin/clickhouse-extract-from-config to /usr/bin/clickhouse.\nSymlink /usr/bin/clickhouse-keeper already exists but it points to /clickhouse. Will replace the old symlink to /usr/bin/clickhouse.\nCreating symlink /usr/bin/clickhouse-keeper to /usr/bin/clickhouse.\nCreating symlink /usr/bin/clickhouse-keeper-converter to /usr/bin/clickhouse.\nCreating clickhouse group if it does not exist.\n groupadd -r clickhouse\nCreating clickhouse user if it does not exist.\n useradd -r --shell /bin/false --home-dir /nonexistent -g clickhouse clickhouse\nWill set ulimits for clickhouse user in /etc/security/limits.d/clickhouse.conf.\nCreating config directory /etc/clickhouse-server/config.d that is used for tweaks of main server configuration.\nCreating config directory /etc/clickhouse-server/users.d that is used for tweaks of users configuration.\nConfig file /etc/clickhouse-server/config.xml already exists, will keep it and extract path info from it.\n/etc/clickhouse-server/config.xml has /var/lib/clickhouse/ as data path.\n/etc/clickhouse-server/config.xml has /var/log/clickhouse-server/ as log path.\nUsers config file /etc/clickhouse-server/users.xml already exists, will keep it and extract users info from it.\nCreating log directory /var/log/clickhouse-server/.\nCreating data directory /var/lib/clickhouse/.\nCreating pid directory /var/run/clickhouse-server.\n chown -R clickhouse:clickhouse '/var/log/clickhouse-server/'\n chown -R clickhouse:clickhouse '/var/run/clickhouse-server'\n chown  clickhouse:clickhouse '/var/lib/clickhouse/'\n groupadd -r clickhouse-bridge\n useradd -r --shell /bin/false --home-dir /nonexistent -g clickhouse-bridge clickhouse-bridge\n chown -R clickhouse-bridge:clickhouse-bridge '/usr/bin/clickhouse-odbc-bridge'\n chown -R clickhouse-bridge:clickhouse-bridge '/usr/bin/clickhouse-library-bridge'\n\u001b[1mPassword for default user is empty string. See /etc/clickhouse-server/users.xml and /etc/clickhouse-server/users.d to change it.\u001b[0m\nSetting capabilities for clickhouse binary. This is optional.\n chown -R clickhouse:clickhouse '/etc/clickhouse-server'\n\nClickHouse has been successfully installed.\n\nStart clickhouse-server with:\n sudo clickhouse start\n\nStart clickhouse-client with:\n clickhouse-client\n\n\n  Verifying  : clickhouse-server-22.3.3.44-1.noarch                         1/3 \n  Verifying  : clickhouse-common-static-22.3.3.44-1.x86_64                  2/3 \n  Verifying  : clickhouse-client-22.3.3.44-1.noarch                         3/3 \n\nInstalled:\n  clickhouse-client.noarch 0:22.3.3.44-1                                        \n  clickhouse-common-static.x86_64 0:22.3.3.44-1                                 \n  clickhouse-server.noarch 0:22.3.3.44-1                                        \n\nComplete!\n"]}

RUNNING HANDLER [Start clickhouse service] **************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:5
changed: [clickhouse-01] => {"changed": true, "name": "clickhouse-server", "state": "started", "status": {"ActiveEnterTimestampMonotonic": "0", "ActiveExitTimestampMonotonic": "0", "ActiveState": "inactive", "After": "network-online.target time-sync.target systemd-journald.socket basic.target system.slice", "AllowIsolate": "no", "AmbientCapabilities": "0", "AssertResult": "no", "AssertTimestampMonotonic": "0", "Before": "shutdown.target", "BlockIOAccounting": "no", "BlockIOWeight": "18446744073709551615", "CPUAccounting": "no", "CPUQuotaPerSecUSec": "infinity", "CPUSchedulingPolicy": "0", "CPUSchedulingPriority": "0", "CPUSchedulingResetOnFork": "no", "CPUShares": "18446744073709551615", "CanIsolate": "no", "CanReload": "no", "CanStart": "yes", "CanStop": "yes", "CapabilityBoundingSet": "8409088", "CollectMode": "inactive", "ConditionResult": "no", "ConditionTimestampMonotonic": "0", "Conflicts": "shutdown.target", "ControlPID": "0", "DefaultDependencies": "yes", "Delegate": "no", "Description": "ClickHouse Server (analytic DBMS for big data)", "DevicePolicy": "auto", "EnvironmentFile": "/etc/default/clickhouse (ignore_errors=yes)", "ExecMainCode": "0", "ExecMainExitTimestampMonotonic": "0", "ExecMainPID": "0", "ExecMainStartTimestampMonotonic": "0", "ExecMainStatus": "0", "ExecStart": "{ path=/usr/bin/clickhouse-server ; argv[]=/usr/bin/clickhouse-server --config=/etc/clickhouse-server/config.xml --pid-file=/run/clickhouse-server/clickhouse-server.pid ; ignore_errors=no ; start_time=[n/a] ; stop_time=[n/a] ; pid=0 ; code=(null) ; status=0/0 }", "FailureAction": "none", "FileDescriptorStoreMax": "0", "FragmentPath": "/usr/lib/systemd/system/clickhouse-server.service", "Group": "clickhouse", "GuessMainPID": "yes", "IOScheduling": "0", "Id": "clickhouse-server.service", "IgnoreOnIsolate": "no", "IgnoreOnSnapshot": "no", "IgnoreSIGPIPE": "yes", "InactiveEnterTimestampMonotonic": "0", "InactiveExitTimestampMonotonic": "0", "JobTimeoutAction": "none", "JobTimeoutUSec": "0", "KillMode": "control-group", "KillSignal": "15", "LimitAS": "18446744073709551615", "LimitCORE": "18446744073709551615", "LimitCPU": "18446744073709551615", "LimitDATA": "18446744073709551615", "LimitFSIZE": "18446744073709551615", "LimitLOCKS": "18446744073709551615", "LimitMEMLOCK": "65536", "LimitMSGQUEUE": "819200", "LimitNICE": "0", "LimitNOFILE": "500000", "LimitNPROC": "15063", "LimitRSS": "18446744073709551615", "LimitRTPRIO": "0", "LimitRTTIME": "18446744073709551615", "LimitSIGPENDING": "15063", "LimitSTACK": "18446744073709551615", "LoadState": "loaded", "MainPID": "0", "MemoryAccounting": "no", "MemoryCurrent": "18446744073709551615", "MemoryLimit": "18446744073709551615", "MountFlags": "0", "Names": "clickhouse-server.service", "NeedDaemonReload": "no", "Nice": "0", "NoNewPrivileges": "no", "NonBlocking": "no", "NotifyAccess": "none", "OOMScoreAdjust": "0", "OnFailureJobMode": "replace", "PermissionsStartOnly": "no", "PrivateDevices": "no", "PrivateNetwork": "no", "PrivateTmp": "no", "ProtectHome": "no", "ProtectSystem": "no", "RefuseManualStart": "no", "RefuseManualStop": "no", "RemainAfterExit": "no", "Requires": "basic.target system.slice network-online.target", "Restart": "always", "RestartUSec": "30s", "Result": "success", "RootDirectoryStartOnly": "no", "RuntimeDirectory": "clickhouse-server", "RuntimeDirectoryMode": "0755", "SameProcessGroup": "no", "SecureBits": "0", "SendSIGHUP": "no", "SendSIGKILL": "yes", "Slice": "system.slice", "StandardError": "inherit", "StandardInput": "null", "StandardOutput": "journal", "StartLimitAction": "none", "StartLimitBurst": "5", "StartLimitInterval": "10000000", "StartupBlockIOWeight": "18446744073709551615", "StartupCPUShares": "18446744073709551615", "StatusErrno": "0", "StopWhenUnneeded": "no", "SubState": "dead", "SyslogLevelPrefix": "yes", "SyslogPriority": "30", "SystemCallErrorNumber": "0", "TTYReset": "no", "TTYVHangup": "no", "TTYVTDisallocate": "no", "TasksAccounting": "no", "TasksCurrent": "18446744073709551615", "TasksMax": "18446744073709551615", "TimeoutStartUSec": "1min 30s", "TimeoutStopUSec": "1min 30s", "TimerSlackNSec": "50000", "Transient": "no", "Type": "simple", "UMask": "0022", "UnitFilePreset": "disabled", "UnitFileState": "disabled", "User": "clickhouse", "Wants": "time-sync.target", "WatchdogTimestampMonotonic": "0", "WatchdogUSec": "0"}}
META: ran handlers
META: ran handlers
META: ran handlers

PLAY [Install Vector] ***********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:57
ok: [vector-01]
META: ran handlers
META: ran handlers
META: ran handlers

PLAY RECAP **********************************************************************************************************************************************************************************************************************************
clickhouse-01              : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0
vector-01                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Здесь я указал выполнить только создание БД logs:
```
root@ansible-test:~/08-ansible-02-playbook/playbook# ansible-playbook -i inventory/prod.yml site.yml -vv --tags tag2
ansible-playbook 2.10.8
  config file = None
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible-playbook
  python version = 3.10.6 (main, Nov 14 2022, 16:10:14) [GCC 11.3.0]
No config file found; using defaults
Skipping callback 'default', as we already have a stdout callback.
Skipping callback 'minimal', as we already have a stdout callback.
Skipping callback 'oneline', as we already have a stdout callback.

PLAYBOOK: site.yml **************************************************************************************************************************************************************************************************************************
2 plays in site.yml

PLAY [Install Clickhouse] *******************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:2
ok: [clickhouse-01]
META: ran handlers
META: ran handlers

TASK [wait 10 sec] **************************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:43
Pausing for 10 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [clickhouse-01] => {"changed": false, "delta": 10, "echo": true, "rc": 0, "start": "2023-02-27 12:36:01.004673", "stderr": "", "stdout": "Paused for 10.0 seconds", "stop": "2023-02-27 12:36:11.005445", "user_input": ""}

TASK [Create database] **********************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:49
changed: [clickhouse-01] => {"changed": true, "cmd": ["clickhouse-client", "-q", "create database logs;"], "delta": "0:00:00.236340", "end": "2023-02-27 08:57:24.273175", "failed_when_result": false, "rc": 0, "start": "2023-02-27 08:57:24.036835", "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}
META: ran handlers
META: ran handlers

PLAY [Install Vector] ***********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:57
ok: [vector-01]
META: ran handlers
META: ran handlers
META: ran handlers

PLAY RECAP **********************************************************************************************************************************************************************************************************************************
clickhouse-01              : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
vector-01                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
Ну и ставим вектор:  
```
root@ansible-test:~/08-ansible-02-playbook/playbook# ansible-playbook -i inventory/prod.yml site.yml -vv --tags tag3
ansible-playbook 2.10.8
  config file = None
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible-playbook
  python version = 3.10.6 (main, Nov 14 2022, 16:10:14) [GCC 11.3.0]
No config file found; using defaults
Skipping callback 'default', as we already have a stdout callback.
Skipping callback 'minimal', as we already have a stdout callback.
Skipping callback 'oneline', as we already have a stdout callback.

PLAYBOOK: site.yml **************************************************************************************************************************************************************************************************************************
2 plays in site.yml

PLAY [Install Clickhouse] *******************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:2
ok: [clickhouse-01]
META: ran handlers
META: ran handlers
META: ran handlers
META: ran handlers

PLAY [Install Vector] ***********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:57
ok: [vector-01]
META: ran handlers

TASK [Vector Install package] ***************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:68
NOTIFIED HANDLER restart vector service for vector-01
changed: [vector-01] => {"changed": true, "changes": {"installed": ["/root/.ansible/tmp/ansible-tmp-1677501395.86628-30422-80000601773544/vector-0.21.1-1.x86_64coazjW.rpm"]}, "msg": "", "rc": 0, "results": ["Loaded plugins: fastestmirror, product-id, search-disabled-repos, subscription-\n              : manager\n\nThis system is not registered with an entitlement server. You can use subscription-manager to register.\n\nExamining /root/.ansible/tmp/ansible-tmp-1677501395.86628-30422-80000601773544/vector-0.21.1-1.x86_64coazjW.rpm: vector-0.21.1-1.x86_64\nMarking /root/.ansible/tmp/ansible-tmp-1677501395.86628-30422-80000601773544/vector-0.21.1-1.x86_64coazjW.rpm to be installed\nResolving Dependencies\n--> Running transaction check\n---> Package vector.x86_64 0:0.21.1-1 will be installed\n--> Finished Dependency Resolution\n\nDependencies Resolved\n\n================================================================================\n Package     Arch        Version       Repository                          Size\n================================================================================\nInstalling:\n vector      x86_64      0.21.1-1      /vector-0.21.1-1.x86_64coazjW      154 M\n\nTransaction Summary\n================================================================================\nInstall  1 Package\n\nTotal size: 154 M\nInstalled size: 154 M\nDownloading packages:\nRunning transaction check\nRunning transaction test\nTransaction test succeeded\nRunning transaction\n  Installing : vector-0.21.1-1.x86_64                                       1/1 \n  Verifying  : vector-0.21.1-1.x86_64                                       1/1 \n\nInstalled:\n  vector.x86_64 0:0.21.1-1                                                      \n\nComplete!\n"]}

TASK [Configure Vector | ensure what directory exists] **************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:77
changed: [vector-01] => {"changed": true, "gid": 0, "group": "root", "mode": "0644", "owner": "root", "path": "/root/vector_config/", "secontext": "unconfined_u:object_r:admin_home_t:s0", "size": 6, "state": "directory", "uid": 0}

TASK [Configure Vector | Template config] ***************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:85
changed: [vector-01] => {"changed": true, "checksum": "73809b9097ffb445e1bb79b309365cb082a109d9", "dest": "/root/vector_config//vector.yml", "gid": 0, "group": "root", "md5sum": "820ee48967091ab8a6a762a7413ff214", "mode": "0644", "owner": "root", "secontext": "system_u:object_r:admin_home_t:s0", "size": 13, "src": "/root/.ansible/tmp/ansible-tmp-1677501419.2308059-30440-228223312297369/source", "state": "file", "uid": 0}

TASK [Configure Service | Template systemd unit] ********************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:93
changed: [vector-01] => {"changed": true, "checksum": "ab51618d25f807192bbde7598341b6ed9df09847", "dest": "/etc/systemd/system/vector.service", "gid": 0, "group": "root", "md5sum": "afbca4d7edaace4e32c9db9bbfb7a452", "mode": "0644", "owner": "root", "secontext": "system_u:object_r:systemd_unit_file_t:s0", "size": 249, "src": "/root/.ansible/tmp/ansible-tmp-1677501421.116802-30456-145917332075211/source", "state": "file", "uid": 0}

RUNNING HANDLER [restart vector service] ****************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:60
changed: [vector-01] => {"changed": true, "name": "vector", "state": "started", "status": {"ActiveEnterTimestampMonotonic": "0", "ActiveExitTimestampMonotonic": "0", "ActiveState": "inactive", "After": "systemd-journald.socket basic.target system.slice network.target", "AllowIsolate": "no", "AmbientCapabilities": "0", "AssertResult": "no", "AssertTimestampMonotonic": "0", "Before": "shutdown.target", "BlockIOAccounting": "no", "BlockIOWeight": "18446744073709551615", "CPUAccounting": "no", "CPUQuotaPerSecUSec": "infinity", "CPUSchedulingPolicy": "0", "CPUSchedulingPriority": "0", "CPUSchedulingResetOnFork": "no", "CPUShares": "18446744073709551615", "CanIsolate": "no", "CanReload": "no", "CanStart": "yes", "CanStop": "yes", "CapabilityBoundingSet": "18446744073709551615", "CollectMode": "inactive", "ConditionResult": "no", "ConditionTimestampMonotonic": "0", "Conflicts": "shutdown.target", "ControlPID": "0", "DefaultDependencies": "yes", "Delegate": "no", "Description": "Vector Service", "DevicePolicy": "auto", "ExecMainCode": "0", "ExecMainExitTimestampMonotonic": "0", "ExecMainPID": "0", "ExecMainStartTimestampMonotonic": "0", "ExecMainStatus": "0", "ExecStart": "{ path=/usr/bin/vector ; argv[]=/usr/bin/vector --config-yaml /root/vector_config//vector.yml --watch-config true ; ignore_errors=no ; start_time=[n/a] ; stop_time=[n/a] ; pid=0 ; code=(null) ; status=0/0 }", "FailureAction": "none", "FileDescriptorStoreMax": "0", "FragmentPath": "/etc/systemd/system/vector.service", "Group": "0", "GuessMainPID": "yes", "IOScheduling": "0", "Id": "vector.service", "IgnoreOnIsolate": "no", "IgnoreOnSnapshot": "no", "IgnoreSIGPIPE": "yes", "InactiveEnterTimestampMonotonic": "0", "InactiveExitTimestampMonotonic": "0", "JobTimeoutAction": "none", "JobTimeoutUSec": "0", "KillMode": "control-group", "KillSignal": "15", "LimitAS": "18446744073709551615", "LimitCORE": "18446744073709551615", "LimitCPU": "18446744073709551615", "LimitDATA": "18446744073709551615", "LimitFSIZE": "18446744073709551615", "LimitLOCKS": "18446744073709551615", "LimitMEMLOCK": "65536", "LimitMSGQUEUE": "819200", "LimitNICE": "0", "LimitNOFILE": "4096", "LimitNPROC": "15063", "LimitRSS": "18446744073709551615", "LimitRTPRIO": "0", "LimitRTTIME": "18446744073709551615", "LimitSIGPENDING": "15063", "LimitSTACK": "18446744073709551615", "LoadState": "loaded", "MainPID": "0", "MemoryAccounting": "no", "MemoryCurrent": "18446744073709551615", "MemoryLimit": "18446744073709551615", "MountFlags": "0", "Names": "vector.service", "NeedDaemonReload": "no", "Nice": "0", "NoNewPrivileges": "no", "NonBlocking": "no", "NotifyAccess": "none", "OOMScoreAdjust": "0", "OnFailureJobMode": "replace", "PermissionsStartOnly": "no", "PrivateDevices": "no", "PrivateNetwork": "no", "PrivateTmp": "no", "ProtectHome": "no", "ProtectSystem": "no", "RefuseManualStart": "no", "RefuseManualStop": "no", "RemainAfterExit": "no", "Requires": "basic.target system.slice network-online.target", "Restart": "always", "RestartUSec": "100ms", "Result": "success", "RootDirectoryStartOnly": "no", "RuntimeDirectoryMode": "0755", "SameProcessGroup": "no", "SecureBits": "0", "SendSIGHUP": "no", "SendSIGKILL": "yes", "Slice": "system.slice", "StandardError": "inherit", "StandardInput": "null", "StandardOutput": "journal", "StartLimitAction": "none", "StartLimitBurst": "5", "StartLimitInterval": "10000000", "StartupBlockIOWeight": "18446744073709551615", "StartupCPUShares": "18446744073709551615", "StatusErrno": "0", "StopWhenUnneeded": "no", "SubState": "dead", "SyslogLevelPrefix": "yes", "SyslogPriority": "30", "SystemCallErrorNumber": "0", "TTYReset": "no", "TTYVHangup": "no", "TTYVTDisallocate": "no", "TasksAccounting": "no", "TasksCurrent": "18446744073709551615", "TasksMax": "18446744073709551615", "TimeoutStartUSec": "1min 30s", "TimeoutStopUSec": "1min 30s", "TimerSlackNSec": "50000", "Transient": "no", "Type": "simple", "UMask": "0022", "UnitFilePreset": "disabled", "UnitFileState": "static", "User": "root", "WatchdogTimestampMonotonic": "0", "WatchdogUSec": "0"}}
META: ran handlers
META: ran handlers

PLAY RECAP **********************************************************************************************************************************************************************************************************************************
clickhouse-01              : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
vector-01                  : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

А теперь выполним все теги:
```
root@ansible-test:~/08-ansible-02-playbook/playbook# ansible-playbook -i inventory/prod.yml site.yml -vv --tags all
ansible-playbook 2.10.8
  config file = None
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible-playbook
  python version = 3.10.6 (main, Nov 14 2022, 16:10:14) [GCC 11.3.0]
No config file found; using defaults
Skipping callback 'default', as we already have a stdout callback.
Skipping callback 'minimal', as we already have a stdout callback.
Skipping callback 'oneline', as we already have a stdout callback.

PLAYBOOK: site.yml **************************************************************************************************************************************************************************************************************************
2 plays in site.yml

PLAY [Install Clickhouse] *******************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:2
ok: [clickhouse-01]
META: ran handlers

TASK [Get clickhouse distrib] ***************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:14
ok: [clickhouse-01] => (item=clickhouse-client) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-client-22.3.3.44.rpm", "elapsed": 0, "gid": 0, "group": "root", "item": "clickhouse-client", "mode": "0644", "msg": "HTTP Error 304: Not Modified", "owner": "root", "secontext": "unconfined_u:object_r:admin_home_t:s0", "size": 38090, "state": "file", "status_code": 304, "uid": 0, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-client-22.3.3.44.noarch.rpm"}
ok: [clickhouse-01] => (item=clickhouse-server) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-server-22.3.3.44.rpm", "elapsed": 0, "gid": 0, "group": "root", "item": "clickhouse-server", "mode": "0644", "msg": "HTTP Error 304: Not Modified", "owner": "root", "secontext": "unconfined_u:object_r:admin_home_t:s0", "size": 61151, "state": "file", "status_code": 304, "uid": 0, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-server-22.3.3.44.noarch.rpm"}
failed: [clickhouse-01] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static-22.3.3.44.rpm", "elapsed": 0, "gid": 0, "group": "root", "item": "clickhouse-common-static", "mode": "0644", "msg": "Request failed", "owner": "root", "response": "HTTP Error 404: Not Found", "secontext": "unconfined_u:object_r:admin_home_t:s0", "size": 246310036, "state": "file", "status_code": 404, "uid": 0, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-22.3.3.44.noarch.rpm"}

TASK [Get clickhouse distrib] ***************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:22
ok: [clickhouse-01] => {"changed": false, "dest": "./clickhouse-common-static-22.3.3.44.rpm", "elapsed": 0, "gid": 0, "group": "root", "mode": "0644", "msg": "HTTP Error 304: Not Modified", "owner": "root", "secontext": "unconfined_u:object_r:admin_home_t:s0", "size": 246310036, "state": "file", "status_code": 304, "uid": 0, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-22.3.3.44.x86_64.rpm"}

TASK [Install clickhouse packages] **********************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:29
ok: [clickhouse-01] => {"changed": false, "msg": "", "rc": 0, "results": ["clickhouse-common-static-22.3.3.44-1.x86_64 providing clickhouse-common-static-22.3.3.44.rpm is already installed", "clickhouse-client-22.3.3.44-1.noarch providing clickhouse-client-22.3.3.44.rpm is already installed", "clickhouse-server-22.3.3.44-1.noarch providing clickhouse-server-22.3.3.44.rpm is already installed"]}
META: ran handlers

TASK [wait 10 sec] **************************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:43
Pausing for 10 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [clickhouse-01] => {"changed": false, "delta": 10, "echo": true, "rc": 0, "start": "2023-02-27 12:37:28.145475", "stderr": "", "stdout": "Paused for 10.0 seconds", "stop": "2023-02-27 12:37:38.146411", "user_input": ""}

TASK [Create database] **********************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:49
ok: [clickhouse-01] => {"changed": false, "cmd": ["clickhouse-client", "-q", "create database logs;"], "delta": "0:00:00.178328", "end": "2023-02-27 08:58:51.460407", "failed_when_result": false, "msg": "non-zero return code", "rc": 82, "start": "2023-02-27 08:58:51.282079", "stderr": "Received exception from server (version 22.3.3):\nCode: 82. DB::Exception: Received from localhost:9000. DB::Exception: Database logs already exists.. (DATABASE_ALREADY_EXISTS)\n(query: create database logs;)", "stderr_lines": ["Received exception from server (version 22.3.3):", "Code: 82. DB::Exception: Received from localhost:9000. DB::Exception: Database logs already exists.. (DATABASE_ALREADY_EXISTS)", "(query: create database logs;)"], "stdout": "", "stdout_lines": []}
META: ran handlers
META: ran handlers

PLAY [Install Vector] ***********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:57
ok: [vector-01]
META: ran handlers

TASK [Vector Install package] ***************************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:68
ok: [vector-01] => {"changed": false, "msg": "", "rc": 0, "results": ["vector-0.21.1-1.x86_64 providing /root/.ansible/tmp/ansible-tmp-1677501461.673935-30560-278785122933015/vector-0.21.1-1.x86_64gJeGvn.rpm is already installed"]}

TASK [Configure Vector | ensure what directory exists] **************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:77
ok: [vector-01] => {"changed": false, "gid": 0, "group": "root", "mode": "0644", "owner": "root", "path": "/root/vector_config/", "secontext": "unconfined_u:object_r:admin_home_t:s0", "size": 24, "state": "directory", "uid": 0}

TASK [Configure Vector | Template config] ***************************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:85
ok: [vector-01] => {"changed": false, "checksum": "73809b9097ffb445e1bb79b309365cb082a109d9", "dest": "/root/vector_config//vector.yml", "gid": 0, "group": "root", "mode": "0644", "owner": "root", "path": "/root/vector_config//vector.yml", "secontext": "system_u:object_r:admin_home_t:s0", "size": 13, "state": "file", "uid": 0}

TASK [Configure Service | Template systemd unit] ********************************************************************************************************************************************************************************************
task path: /root/08-ansible-02-playbook/playbook/site.yml:93
ok: [vector-01] => {"changed": false, "checksum": "ab51618d25f807192bbde7598341b6ed9df09847", "dest": "/etc/systemd/system/vector.service", "gid": 0, "group": "root", "mode": "0644", "owner": "root", "path": "/etc/systemd/system/vector.service", "secontext": "system_u:object_r:systemd_unit_file_t:s0", "size": 249, "state": "file", "uid": 0}
META: ran handlers
META: ran handlers

PLAY RECAP **********************************************************************************************************************************************************************************************************************************
clickhouse-01              : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0
vector-01                  : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Поскольку мы потегово выполнили плейбук ничего не изменилось. И это отлично, поскольку работает идемпотентность!
