Ansible Juniper OS Upgrade
==========================
Ansible-Playbook을 이용한 junos OS 업그레이드  
NSSU / ISSU 업그레이드 추가

Library Module(requirements.txt)
-----------------------------------
 - Dependency Module
 - Python >= 3.5
 - Ansible 2.9 or later
 - Junos py-junos-eznc 2.5.0 or later
 - jxmlease 1.0.1 or later
 - Juniper.junos.software 2.4.0

참고사항
--------
 - 여러개의 RE을 사용할 경우(VC, VCF) Routing-Engine마다 버전이 다른 상태이면 junos_software module로는 업그레이드 불가하므로 각각의 RE를 모두 동일한 OS로 맞춘 뒤, 업그레이드를 해야 한다.(물론, network_cli로 바꿔서 Command를 넣으면 가능은 하지만, 매번 YAML설정을 변경해야 하니 불편하게 됨)
 - all_re 옵션을 false로 할 경우 Master인 Routing-Engine만 업그레이드가 되므로 다음 실행 시에 위의 제약사항이 생긴다.
 - connection local은 netconf와 동일(local = legacy)


주의사항
--------
 - 꼭... Ansible로 OS Upgarde하기 전에 현재 Juniper 장비의 OS가 업그레이드가 정상적으로 되는지 확인 후에 하자...  
처음에 무작정 앤서블로 했는데 막힌 요소가 너무나 많았다. 현재 OS가 ISSU를 지원하지 않는 OS인 경우도 있었고, 앤서블의 변수가 지원되지 않는 변수를 쓰기도 했었고.. QFX5110에 QFX5100 OS를 입력해서 안되기도 했었고.. 정말 많은 에러가 있었다.  
**+꼭! 현재 OS에서 테스트 완료 후에 앤서블로 해보자**  

 - NSSU를 수행 시, Master RE를 설치 후, Backup으로 전환 할 때 netconf 세션이 중단 되는 이슈 존재  
 - https://forums.juniper.net/t5/Junos-Automation-Scripting/How-to-do-NSSU-with-Ansible-via-SSH-NetConf-as-session/td-p/462509
 - reboot 변수를 no로 하거나 wait_for를 주어 netconf 세션이 중단되지 않도록 설정해야 함.


inventory(hosts)
----------------
 - hosts-junos
```bash
[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_connection=(netconf or local)
ansible_become=yes
ansible_become_method=config
ansible_user="Your UserID"
ansible_password="Your Password"
ansible_become_pass="Your enable Password"

[qfx5100]
nssu01 ansible_host=10.20.1.2

[qfx5100:vars]
ansible_network_os=junos
ansible_network_model=qfx5100

[junos:children]
qfx5100
(Your More Item)


이하 생략
```  

Playbook Config
---------------
 - Normal OS Upgrade : junos-os.yml  
 - [N/I]SSU OS Upgrade : junos-os-Xssu.yml



NSSU
====
 - Virtual-Chassis와 같이 Routing-Engine이 2개 이상인 상태에서만 NSSU 사용 가능.
 - TEST Device : QFX5100 / QFX5110
 - TEST OS : V18 ~ 20  

NSSU 특징
---------
1. NSSU는 데이터 플레인 트래픽 중단을 최소화하면서 서로 다른 두 Junos OS 릴리스 간의 업그레이드를 지원합니다. NSSU는 중복 라우팅 엔진이있는 EX 시리즈 가상 섀시 시스템 또는 EX 시리즈 독립 실행 형 시스템에만 해당됩니다.  
2. VC / VCF에서 사용(즉, Routing-Engine이 2개 이상인 상태에서 사용 가능)  
3. 제어 플레인을 중단하지 않으며, 인터페이스/커널/라우팅 프로토콜 정보가 보존 됨.  
4. STP에 대한 re-convergence가 없음.  
5. NSSU 사용 시, 롤백 및 다운그레이드에는 제약이 있음.  
6. ISSU 및 NSSU는 상호 배타적  
7. NSSU/ISSU 업그레이드 중에는 rollback이 안됨. ( 사전에 commit confirm으로 제한된 시간이 지나도 OS업그레이드를 완료한 뒤에 rollback이 즉시 적용됨 )  
8. 소요시간 약 10분xRE개수

NSSU 제약 사항
--------------
1. Master RE와 Backup RE는 같은 OS 버전이어야 함.  
2. NSSU는 Routing-Engine이 여러개인 장비에서만 사용 가능하다.  
  + CLI: set virtual-chassis preprovisioned  

**3. virtual-chassis를 pre-provisioning으로 구성하여야만 NSSU가 사용될 수 있음. ( 가장 중요 )**  
4. VC members should be connected in ring topology  
5. chassis should be configured with no-split-detection  
  + CLI: set virtual-chassis no-split-detection
6. enable GRES (Graceful routing engine switchover)  
  + CLI: set chassis redundancy graceful-switchover
7. enable NSR (Non-stop active routing)  
  + CLI: set routing-options nonstop-routing
8. enable NSB (Non-stop brigding) - 선택 사항  
  + CLI: set protocols layer2-control nonstop-bridging
9. request system snapshot 명령 을 사용하여 각 라우팅 엔진의 시스템 소프트웨어를 외부 저장 장치에 백업(USB 백업) - 선택 사항  
  + no-split-detection은 Routing-Engine을 3개 이상으로 할 경우 설정하면 안됨

NSSU 동작 방식
--------------
1. VC Master는 아래 상태들을 먼저 확인
2. 백업이 온라인 상태이며 동일한 OS 체크
3. GRES (Graceful Routing Engine Switchover) 및 NSR (nonstop active routing)가 활성화 되었는지 체크
4. VC preprovisioning 활성화 체크
5. 마스터는 백업에 신규 OS를 설치하고 재부팅
6. 마스터는 백업을 재 동기화
7. 마스터는 라인 카드 역할에있는 구성원 스위치에 새 소프트웨어 이미지를 설치하고 한 번에 하나씩 재부팅합니다. 마스터는 다음 멤버에서 소프트웨어 업그레이드를 시작하기 전에 각 멤버가 온라인 상태가되고 활성화 될 때까지 기다립니다.
8. 라인 카드 역할에있는 모든 구성원이 업그레이드되면 마스터가 단계적 라우팅 엔진 전환을 수행하고 업그레이드 된 백업이 마스터가됩니다.
9. 원래 마스터의 소프트웨어가 업그레이드되고 원래 마스터가 자동으로 재부팅됩니다. 원래 마스터가 Virtual Chassis에 다시 가입 한 후에는 선택적으로 우아한 라우팅 엔진 전환을 요청하여 제어권을 되돌릴 수 있습니다.  
**10. 즉, NSSU가 정상적으로 마치게 되면, 기존에 backup이었던 1번 장비가 Master로 변환된 상태**  

Virtual-Chassis 구성 방안 2가지
-------------------------------
### VC - Preprovisioning 미사용
```shell
set virtual-chassis member 0 mastership-priority 255
set virtual-chassis member 1 mastership-priority 255
```

### VC - Preprovisioning 사용
```shell
set virtual-chassis preprovisioned
set virtual-chassis member 0 role routing-engine
set virtual-chassis member 0 serial-number AB1234567890     # 1번 장비 serial 번호 입력.
set virtual-chassis member 1 role routing-engine
set virtual-chassis member 1 serial-number BC1234567890       # 2번 장비 serial 번호 입력.
```

See Also
--------
 - ansible-galaxy-junos - https://galaxy.ansible.com/juniper/device
 - ansible-junos-collection - https://github.com/Juniper/ansible-junos-stdlib
 - NSSU 미사용 - https://www.juniper.net/documentation/en_US/release-independent/nce/topics/example/nce-171-qfx-vc-upgrade.html
