[root@aap25 09_NetworkManager_Auto_Manage]# ansible-playbook networkmanager_management_site.yml -i inventory -t view

PLAY [Automated NetworkManager Management] ****************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************
ok: [Test-RHEL-7.9-1]
ok: [Test-RHEL-7.9-3]
ok: [Test-RHEL-7.9-2]
ok: [ansible25.example.com]
ok: [TEST-RHEL-8.9-1]

TASK [networkmanager_cli : 1.1 View all network device status (nmcli dev status)] *************************************************************************************
ok: [Test-RHEL-7.9-1]
ok: [Test-RHEL-7.9-2]
ok: [Test-RHEL-7.9-3]
ok: [TEST-RHEL-8.9-1]
ok: [ansible25.example.com]

TASK [networkmanager_cli :    -> Display device status] ***************************************************************************************************************
ok: [Test-RHEL-7.9-1] => {
    "dev_status.stdout_lines": [
        "DEVICE  TYPE      STATE      CONNECTION ",
        "ens192  ethernet  connected  ens192     ",
        "lo      loopback  unmanaged  --         "
    ]
}
ok: [Test-RHEL-7.9-2] => {
    "dev_status.stdout_lines": [
        "DEVICE  TYPE      STATE      CONNECTION    ",
        "ens192  ethernet  connected  static-ens192 ",
        "lo      loopback  unmanaged  --            "
    ]
}
ok: [Test-RHEL-7.9-3] => {
    "dev_status.stdout_lines": [
        "DEVICE  TYPE      STATE      CONNECTION ",
        "ens192  ethernet  connected  ens192     ",
        "lo      loopback  unmanaged  --         "
    ]
}
ok: [TEST-RHEL-8.9-1] => {
    "dev_status.stdout_lines": [
        "DEVICE  TYPE      STATE                   CONNECTION ",
        "ens192  ethernet  connected               ens192     ",
        "virbr0  bridge    connected (externally)  virbr0     ",
        "lo      loopback  unmanaged               --         "
    ]
}
ok: [ansible25.example.com] => {
    "dev_status.stdout_lines": [
        "DEVICE  TYPE      STATE         CONNECTION ",
        "ens33   ethernet  已连接        ens33      ",
        "lo      loopback  连接（外部）  lo         "
    ]
}

TASK [networkmanager_cli : 1.2 View all network connections (nmcli con show)] *****************************************************************************************
ok: [Test-RHEL-7.9-1]
ok: [Test-RHEL-7.9-2]
ok: [Test-RHEL-7.9-3]
ok: [TEST-RHEL-8.9-1]
ok: [ansible25.example.com]

TASK [networkmanager_cli :    -> Display connection information] ******************************************************************************************************
ok: [Test-RHEL-7.9-1] => {
    "con_show.stdout_lines": [
        "NAME         UUID                                  TYPE      DEVICE ",
        "ens192       b527e11f-4bbc-4eec-9bb4-74f3d88ecb8d  ethernet  ens192 ",
        "dhcp-eth1    2bd288a0-4e0f-4439-b992-4347fde3a914  ethernet  --     ",
        "dhcp-eth2    2484c835-4fc3-4db5-b06e-5c8680e89e54  ethernet  --     ",
        "dhcp-eth2    c2b15bcf-2c48-4d2b-8908-568a3f816568  ethernet  --     ",
        "dhcp-eth2    eded205f-74ea-416a-867f-269442ab4411  ethernet  --     ",
        "dhcp-eth2    50eb6b60-5532-478d-82ee-dd236dfa5ece  ethernet  --     ",
        "static-eth1  4d96766b-8b15-425b-b9ac-dfbe7dfb7cc7  ethernet  --     ",
        "static-eth1  5be37a8b-2354-4dfc-9aae-f1ed67c768a1  ethernet  --     ",
        "static-eth1  c366d293-d7fd-4d4a-ae7e-343e2e0a8550  ethernet  --     ",
        "static-eth1  1b114d82-3546-4abb-8092-233d44b5cf7f  ethernet  --     ",
        "static-eth1  34c5a7fe-3d48-496d-a082-7eeb545ab10c  ethernet  --     "
    ]
}
ok: [Test-RHEL-7.9-2] => {
    "con_show.stdout_lines": [
        "NAME           UUID                                  TYPE      DEVICE ",
        "static-ens192  e53d7147-c602-414f-8ed5-2244d9099aaa  ethernet  ens192 ",
        "ens192         e85e9a8c-a553-48a7-bdbf-838c77bf27e0  ethernet  --     "
    ]
}
ok: [Test-RHEL-7.9-3] => {
    "con_show.stdout_lines": [
        "NAME    UUID                                  TYPE      DEVICE ",
        "ens192  73f0d8b1-4f5e-4005-93a9-aa4ab33902b1  ethernet  ens192 "
    ]
}
ok: [TEST-RHEL-8.9-1] => {
    "con_show.stdout_lines": [
        "NAME    UUID                                  TYPE      DEVICE ",
        "ens192  91116322-e5ac-430a-acda-672cf6a134e1  ethernet  ens192 ",
        "virbr0  7668db02-aff1-472b-b693-b5e2a9a91768  bridge    virbr0 "
    ]
}
ok: [ansible25.example.com] => {
    "con_show.stdout_lines": [
        "NAME   UUID                                  TYPE      DEVICE ",
        "ens33  c14ca192-9dcd-35d9-a747-d6c290927639  ethernet  ens33  ",
        "lo     21613a67-324a-42eb-b61a-e7f4287536f9  loopback  lo     "
    ]
}

PLAY RECAP ************************************************************************************************************************************************************
TEST-RHEL-8.9-1            : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
Test-RHEL-7.9-1            : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
Test-RHEL-7.9-2            : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
Test-RHEL-7.9-3            : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ansible25.example.com      : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
