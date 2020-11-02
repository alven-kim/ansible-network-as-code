# Ansible Network as Code
Network-Engineer를 위한 Ansible-playbook을 이용하여 네트워크 각 벤더별 네트워크 자동화 Sample

## 들어가기 앞서...
네트워크 장비들에 대한 자동화 스크립트를 몇 번 작성하고 사용해본 네트워크 엔지니어 입장에서 앤서블은 추천드립니다.
다만, 자동화에 대한 호기심으로 접근하시거나 앤서블로 정점을 찍겠다는 분들에게는 추천드리지만.. 본인이 파이썬/각종 쉘/이외 스크립팅이 가능하다면, 앤서
블을 사용하지 말고 본인이 가장 빠르고 정확하게 할 수 있는 스크립트를 사용하는게 좋다고 생각합니다.

### Ansible 특징
1. 멱등성 : Ansible은 멱등성이 존재한다.
2. 인터프리터 : 앤서블은 파이썬으로 만들어졌기에 파이썬과 같이 인터프리터로 동작한다.
 - 처리방식에서 인터프리터는 코드 한줄 한줄을 일일히 읽는다. ( 컴파일러는 코드 전체를 읽은 다음에 실행한다. ) 때문에, failed_when 같은 변수를 사용하는 경우가 종종 발생한다.

### Ansible에 대해 알아야할 중요한 사항들
 - Ansible에서 인터프리터로 동작하는 부분은 매우 중요하다.
왜냐하면 어떤 설정들을 넣었을 때, 중복이 있다면 default로 fail을 시키기 때문에 그 이후의 설정을 진행하지 않는다. 따라서, 중복에 대한 옵션은 true로 설정한 뒤에 진행하는 것이 좋다.
 - 현재 Python2는 지원을 중단하는 추세이며, 앤서블도 되도록 python3으로 설정/사용하자.

### Ansible 이외 참고 사항들
Network장비를 Ansible로 사용하는 경우에는 gather_facts의 쓰임새를 적절히 활용하자.
gather_facts에는 Network장비에 대한 Model, Version, OS, Interface, Serialnum 등, 각종 정보들에 대하여 이미 변수로 만들어져 있기 때문에 gather_facts만 true로 설정한 뒤에 해당 변수들만 가져다가 쓰면 간단하다. 다만, 선언된 변수를 읽는 과정이 각 플레이마다 실행하여 반복되기 때문에 안그래도 느린 앤서블이 더 느려지게 되므로 빠르게 처리되어야 하는 환경에서는 잘 고려하여 사용하자.
 - gather_facts 주의사항  
gather_facts의 default는 true로 활성화다. juniper의 경우에는 현재 지원하지 않는다.
느리다. 왠만한 reserved variable들을 제외하여도 그 하위의 변수들이 많기때문에 느리다. 따라서, 여러 장비들을 할때에는 잘 고려하여 사용하자.
 - Vault를 이용하여 패스워드는 암호화하자.  
 - Duplicate : https://docs.ansible.com/ansible/latest/reference_appendices/config.html#duplicate-yaml-dict-key

### Library Module(requirements.txt)
Dependency Module(Requirements.txt)
```shell
root$ pip3 install -r requirements.txt
```


## How to use each network-vendors
각 벤더별 Ansible-Playbook 단독 예제

### Case1) icx-ansible
ICX 스위치 장비들에 대한 Ansible-Playbook 예제
 - https://github.com/alven-kim/ansible-network-as-code/tree/master/ansible-icx

### Case2) junos-ansible
Juniper 스위치 장비들에 대한 Ansbiel-Playbook 예제
 - https://github.com/alven-kim/ansible-network-as-code/tree/master/ansible-junos

### Case2-2) junos-ansible-os-upgrade(nssu, issu)
Juniper 스위치 장비들의 OS Upgrade 대한 Ansbiel-Playbook 예제(nssu, issu)
 - https://github.com/alven-kim/ansible-network-as-code/tree/master/ansible-junos-os-upgrade-Xssu

### Case3) tmos-ansible
F5 장비들에 대한 Ansible-Playbook 예제  
 - https://github.com/alven-kim/ansible-network-as-code/tree/master/ansible-tmos
 - node
 - pool
 - pool-member
 - virtual-Server
 - virtual-Address
 - monitor
 - profile
 - snat / snat_pool
 - etc

### Case4) ansible-vault
Ansible Vault를 이용한 패스워드 암호화
 - https://github.com/alven-kim/ansible-network-as-code/tree/master/ansible-vault


## How to use Ansible-Playbook Standard
Ansible-Playbook에 대한 사용 방법(표준화)

### YAML 디렉토리 구조
 - current_dirctory - role_dir - vendor_dir - tasks_dir - main.yml
 - tasks 폴더없이 실행 시, main.yml의 tasks 활성화
 - meta 데이터 없이 실행 버전

### inventory 구조
 - switch > os(icx,junos,ios) > os_vars(inventory_hostname, ansible_host, ansible_...os_model)
 - loadbalancer > os(tmos, alteonos) > os_vars(inventory_hostname, ansible_host, ansible_...os_model)
 - network > os(all) > os_vars(inventory_hostname, ansible_host, ansible_...os_model)

### Example yaml
 - network-role.yml

### PLAY
 - normal : ansible-playbook -i [inventory] [playbook yaml]
 - valut : ansible-playbook --ask-vault-pass -i [vault-inventory] [playbook yaml]
  + vault 방법은 링크 참조(https://github.com/alven-kim/ansible-network)

## Result
 - Play Result
```bash
root# ansible-playbook -i hosts network-role.yml

PLAY [Get Device Facts] *********************************************************************************************************************************************************

TASK [icx : ICX6000, FWS624G Commands Config] ***********************************************************************************************************************************
skipping: [icx6(test-sw-06)]
skipping: [junos4(test-sw-04)]

TASK [icx : ICX7000 Commands Config] *******************************************************************************************************************************************$
skipping: [junos4(test-sw-04)]
changed: [icx6(test-sw-06)]

TASK [icx : ICX6000 - Copy to Output] ******************************************************************************************************************************************$
skipping: [icx6(test-sw-06)]
skipping: [junos4(test-sw-04)]

TASK [icx : ICX7000 - Copy to Output] ******************************************************************************************************************************************$
skipping: [junos4(test-sw-04)]
changed: [icx6(test-sw-06)]

TASK [junos : JUNIPER - CMD - CONFIG] ******************************************************************************************************************************************$
skipping: [icx6(test-sw-06)]
[WARNING]: arguments wait_for, match, rpcs are not supported when using transport=cli
ok: [junos4(test-sw-04)]

TASK [junos : JUNIPER - Copy to output "/etc/ansible"] *************************************************************************************************************************$
skipping: [icx6(test-sw-06)]
changed: [junos4(test-sw-04)]

TASK [junos : JUNIPER Print Output] ********************************************************************************************************************************************$
skipping: [icx6(test-sw-06)]
ok: [junos4(test-sw-04)] => {
    "msg": [
(이하 생략)
}

PLAY RECAP *********************************************************************************************************************************************************************$
icx6(test-sw-06)            : ok=2    changed=2    unreachable=0    failed=0    skipped=5    rescued=0    ignored=0
junos4(test-sw-04)      : ok=3    changed=1    unreachable=0    failed=0    skipped=4    rescued=0    ignored=0
```
 - Result
<img width="734" alt="ansible-network-all-result" src="https://user-images.githubusercontent.com/41031835/96415321-5f7e3d80-1229-11eb-89d8-700c4465cfde.png">

## To-Do
미정

## See also
Ansible for Network Automation
 - https://docs.ansible.com/ansible/latest/network
Network Advanced Topics
 - https://docs.ansible.com/ansible/latest/network/user_guide/index.html
Ansible-Galaxy
 - https://galaxy.ansible.com/home
Create a directory if it does not exist
 - https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html#file-module
Match the string or regex
 - https://docs.ansible.com/ansible/latest/user_guide/playbooks_tests.html#testing-strings
Playbook handling(error, failed, changed ...)
 - https://docs.ansible.com/ansible/latest/user_guide/playbooks_error_handling.html
gather_facts
 - ICX gather_facts - https://docs.ansible.com/ansible/latest/modules/icx_facts_module.html
 - IOS gather_facts - https://docs.ansible.com/ansible/latest/modules/ios_facts_module.html
 - junos gather_facts - https://docs.ansible.com/ansible/latest/modules/junos_facts_module.html
Ansible-junos
 - ansible-galaxy-junos - https://galaxy.ansible.com/juniper/device
 - ansible-junos-collection - https://github.com/Juniper/ansible-junos-stdlib
 - NSSU 미사용- https://www.juniper.net/documentation/en_US/release-independent/nce/topics/example/nce-171-qfx-vc-upgrade.html
Ansible-tmos
 - https://galaxy.ansible.com/f5networks/f5_modules
 - https://galaxy.ansible.com/f5devcentral/f5ansible
 - https://github.com/ansible/workshops/tree/master/exercises/ansible_f5
 - https://clouddocs.f5.com/products/orchestration/ansible/devel
 - https://clouddocs.f5.com/products/orchestration/ansible/devel/modules/module_index.html
 - https://docs.ansible.com/ansible/latest/collections/f5networks/f5_modules/bigip_node_module.html
 - group_vars 적용하여 호스트 그룹별 변수 적용 - https://ansible-tips-and-tricks.readthedocs.io/en/latest/ansible/inventory
