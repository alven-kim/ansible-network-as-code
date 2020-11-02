# Ansible-TMOS
Ansible-Playbook을 활용한 F5(tmos) 생성
 - node
 - pool
 - pool-member
 - virtual-Server
 - virtual-Address
 - traffic-group
 - partition
 - monitor
 - profile
 - snat / snat_pool
 - etc

## 사용 방식
Provider를 입력하여 사용하는 형태

## 참고 사항
validate_certs는 provider에서만 사용하고 Node, Pool, VServer 등에는 사용할 수 없는 모듈임. ( 사용 시, 에러 출력 )  
state에서 present는 생성을 정의하고 absent는 삭제를 정의 함. ( bigip_virtual_address에서는 traffic-group을 변경하는 것이므로 생성도 아닌 삭제도 아니므로 state를 빼야됨 )

## Result
```bash
root@ansible1:/etc/ansible# ansible-playbook -i hosts f5_ssl_dist.yml
/usr/local/lib/python2.7/dist-packages/requests/__init__.py:83: RequestsDependencyWarning: Old version of cryptography ([1, 2, 3]) may cause slowdown.
  warnings.warn(warning, RequestsDependencyWarning)

PLAY [Connection-TEST - Create a VServer, Pool, Pool-Members, Nodes] ***************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************************************************
ok: [f5]

TASK [Create node-ALL] *************************************************************************************************************************************************************************************
changed: [f5 -> localhost] => (item=test_node01a)
changed: [f5 -> localhost] => (item=test_node02a)

TASK [Create a POOL] ***************************************************************************************************************************************************************************************
changed: [f5] => (item=test_node01a)
ok: [f5] => (item=test_node02a)

TASK [Add to pool-members] *********************************************************************************************************************************************************************************
changed: [f5] => (item=test_node01a)
changed: [f5] => (item=test_node02a)

TASK [Add Virtual-Address(Traffic-Group-1to3)] *************************************************************************************************************************************************************
changed: [f5 -> localhost] => (item=ansible.alvenkim.com_80_1)
changed: [f5 -> localhost] => (item=ansible.alvenkim.com_80_2)
changed: [f5 -> localhost] => (item=ansible.alvenkim.com_80_3)

TASK [Create VServers(Traffic-Group-1to3)] *****************************************************************************************************************************************************************
changed: [f5] => (item=ansible.alvenkim.com_80_1)
changed: [f5] => (item=ansible.alvenkim.com_80_2)
changed: [f5] => (item=ansible.alvenkim.com_80_3)

TASK [SAVE RUNNING CONFIG ON BIG-IP] ***********************************************************************************************************************************************************************
changed: [f5]

PLAY RECAP *************************************************************************************************************************************************************************************************
f5                         : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

root@ansible1:/etc/ansible#
```

## To-Do
Provider 방식을 사용하지 않고, inventory를 활용

## See Also
ansible-galaxy-f5
 - https://galaxy.ansible.com/f5networks/f5_modules
 - https://galaxy.ansible.com/f5devcentral/f5ansible

ansible-f5-ref
 - https://github.com/ansible/workshops/tree/master/exercises/ansible_f5

ansible-f5-clouddocs  
 - https://clouddocs.f5.com/products/orchestration/ansible/devel

ansible-f5-clouddocs-module  
 - https://clouddocs.f5.com/products/orchestration/ansible/devel/modules/module_index.html

ansible-f5-module  
 - https://docs.ansible.com/ansible/latest/collections/f5networks/f5_modules/bigip_node_module.html
