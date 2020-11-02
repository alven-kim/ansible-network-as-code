Ansible-Vault
=============

## inventory(hosts)
### 아래 순번들에 대한 inventory 예제 샘플
 - Vault 미사용 inventory
Vault로 암호화 되지 않은 상태.
```shell
[all:vars]
ansible_connection=network_cli
ansible_user=admin
ansible_become=yes
ansible_become_method=enable
ansible_password="Your Password"
ansible_become_pass="Your enable Password"

[icx]
icx01 ansible_host=10.10.1.2
icx02 ansible_host=10.10.1.3
icx03 ansible_host=10.10.1.4
icx04 ansible_host=10.10.1.5

[icx:vars]
ansible_network_os=icx

[junos]
junos01 ansible_host=10.20.1.2

[junos:vars]
ansible_network_os=junos

이하 생략
```

 - Vault 사용 inventory
Vault로 인하여 AES256으로 암호화된 값이 출력
```shell
$ANSIBLE_VAULT;1.1;AES256
3436633062313763633636623965373530633533316132656665383636363261663634336130
이하 생략
```

## Vault Commands
### Vault 사용 시, Command 예제
```shell
# 암호화된 파일을 생성하기
root$ ansible-vault create hosts

# 암호화된 파일의 내용 보기
root$ ansible-vault view hosts

# 암호화된 파일 수정하기
root$ ansible-vault edit hosts

# 암호화된 파일의 패스워드 변경하기
root$ ansible-vault rekey hosts

# 암호화되지 않은 파일을 암호화
root$ ansible-vault encrypt hosts

# 암호화된 파일의 복호화
root$ ansible-vault decrypt hosts
```

### Vault로 암호화 된 파일 Playbook 사용 예제
ansible-playbook --ask-vault-pass -i [inventory] [playbook.yml]
```shell
root# ansible-playbook --ask-vault-pass -i hosts icx.yml
Vault password:

PLAY [Get Device Facts] ***************************************************************************************************************************************

TASK [Gather facts (icx)] *************************************************************************************************************************************
ok: [icx02]
ok: [icx04]
ok: [icx01]
ok: [icx03]

TASK [Gather facts (eos)] *************************************************************************************************************************************
skipping: [icx01]
skipping: [icx02]
(이하 생략)
```

### hosts파일에 대한 vault 패스워드를 vault_pass.txt에 저장하여 패스워드 입력없이 Playbook 실행 예제
 1. vault_pass.txt 파일명에 패스워드 입력. ( 패스워드는 무조건 1줄이어야 하고 특수문자 사용 가능 )  
 2. vault_pass.txt 파일은 vault로 암호화하면 안됨. ( 암호화가 안되니 쓰는 이유가 없지만, 뭔가 방법이 있을듯 함 )  
 3. 아래 명령어 실행.  
 참고) ansible.cfg 파일에 vault_password_file=/etc/ansible/vault_pass.txt 정의하면 --vault 옵션없이 실행 가능.
```shell
# ansible-playbook -i inventory icx.yml --vault-password-file vault_pass.txt

PLAY [Get Device Facts] ***************************************************************************************************************************************

TASK [Gather facts (icx)] *************************************************************************************************************************************
ok: [icx02]
ok: [icx04]
ok: [icx01]
ok: [icx03]

TASK [Gather facts (eos)] *************************************************************************************************************************************
skipping: [icx01]
skipping: [icx02]
(이하 생략)
```

## See Also
 - https://docs.ansible.com/ansible/latest/cli/ansible-vault.html
