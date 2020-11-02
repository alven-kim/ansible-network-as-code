Ansible-junos
=============
Ansible-playbook을 이용한 junos 네트워크 자동화

Example yaml
------------
 - juniper.yml 
 - juniper-config.yml

PLAY
----
 - normal : ansible-playbook -i [inventory] [playbook yaml]  
 - valut : ansible-playbook --ask-vault-pass -i [vault-inventory] [playbook yaml]  
 - vault 방법은 링크 참조(https://github.com/alven-kim/ansible-network-as-code/tree/master/ansible-vault)

Result
------
 - Play Result
```
root# ansible-playbook -i hosts juniper-config.yml

PLAY [Get Device Facts] *********************************************************************************************************************************************************

TASK [run show version on remote devices] ***************************************************************************************************************************************
ok: [junos2(qa-l3)]
ok: [junos3(l2-1)]
TASK [Debug Print Output] *******************************************************************************************************************************************************
ok: [junos2(qa-l3] => {
(이하 생략)
```
 - Result
<img width="734" alt="junos-result" src="https://user-images.githubusercontent.com/41031835/94407961-1f3e1900-01af-11eb-8129-947b03b24bf9.png">


See also
--------
Ansible-junos
 - https://www.juniper.net/documentation/en_US/junos-ansible/topics/reference/general/junos-ansible-modules-overview.html  
Ansible-junos-Command
 - https://junos-ansible-modules.readthedocs.io/en/2.4.0/juniper_junos_command.html
