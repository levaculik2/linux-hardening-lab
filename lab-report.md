Hardening Lab - Ubuntu Server
Autor: Leandro Vaculik
Objetivo: Implementar práticas de hardening em um servidor Linux para reduzir a superfície de ataque e melhorar a segurança do sistema.

---

1. Ambiente

Hipervisor: Oracle VirtualBox
Servidor: Ubuntu Server 24.04.4
Máquina de ataque: Kali Linux

Topologia de rede (Lab interno):

Kali Linux (Attacker) → 192.168.100.10
Ubuntu Server (Target) → 192.168.100.20

Rede configurada em modo Host-Only para permitir comunicação entre as máquinas virtuais.

---

2. Reconhecimento inicial

Ferramenta utilizada: Nmap

Comando executado no Kali Linux:

nmap -sS -sV 192.168.100.20

Objetivo:

Identificar portas abertas no servidor antes da aplicação das medidas de segurança.

Resultado:

PORT    SERVICE
22/tcp  SSH

Conclusão:

O servidor possui apenas o serviço SSH exposto na rede.

---

3. Hardening do serviço SSH

Arquivo de configuração:

/etc/ssh/sshd_config

Alterações realizadas:

PermitRootLogin no
PasswordAuthentication no
MaxAuthTries 3
X11Forwarding no

Objetivo:

Reduzir riscos de acesso indevido ao servidor através do serviço SSH.

Medidas aplicadas:

* Desabilitado login direto do usuário root
* Desabilitada autenticação por senha
* Reduzido número máximo de tentativas de login
* Desabilitado encaminhamento X11

Após alteração foi executado:

sudo systemctl restart ssh

Resultado:

Tentativas de login direto com root foram bloqueadas.

---

4. Firewall

Ferramenta utilizada: UFW (Uncomplicated Firewall)

Instalação:

sudo apt install ufw

Configuração aplicada:

sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw enable

Verificação do status:

sudo ufw status

Resultado:

Status: active

Portas permitidas:

22/tcp ALLOW

Conclusão:

Todo tráfego de entrada é bloqueado por padrão, exceto conexões SSH autorizadas.

---

5. Proteção contra brute force

Ferramenta utilizada: Fail2ban

Instalação:

sudo apt install fail2ban

Configuração criada em:

/etc/fail2ban/jail.local

Configuração aplicada:

[sshd]
enabled = true
port = ssh
maxretry = 3
bantime = 600
findtime = 600

Objetivo:

Bloquear automaticamente endereços IP que realizem múltiplas tentativas de autenticação falhadas.

Reinicialização do serviço:

sudo systemctl restart fail2ban

---

6. Teste de ataque (simulação de brute force)

Ferramenta utilizada: Hydra

Ataque executado no Kali Linux:

hydra -l cyberlab -P /usr/share/wordlists/rockyou.txt -t 4 ssh://192.168.100.20

Objetivo:

Simular um ataque de força bruta contra o serviço SSH.

Resultado observado:

Após múltiplas tentativas de autenticação falhadas, o endereço IP do atacante foi bloqueado automaticamente pelo Fail2ban.

Verificação do bloqueio:

sudo fail2ban-client status sshd

Saída observada:

Banned IP list:
192.168.100.10

---

7. Validação do bloqueio

Tentativa manual de conexão SSH após o bloqueio:

ssh cyberlab@192.168.100.20

Resultado:

Connection refused / connection timed out

Conclusão:

O mecanismo de proteção contra brute force está funcionando corretamente.

---

8. Auditoria de segurança do sistema

Ferramenta utilizada: Lynis

Instalação:

sudo apt install lynis

Execução da auditoria:

sudo lynis audit system

Objetivo:

Avaliar a configuração de segurança do sistema e identificar possíveis melhorias adicionais.

O Lynis realizou diversas verificações incluindo:

* configuração do SSH
* políticas de autenticação
* permissões de arquivos
* configuração do firewall
* pacotes desatualizados

Resultado:

Foram apresentadas recomendações adicionais para fortalecimento do sistema.

---

9. Conclusão

Após a aplicação das técnicas de hardening, o servidor apresentou melhorias significativas na sua postura de segurança.

Principais medidas implementadas:

* Configuração segura do serviço SSH
* Implementação de firewall com política restritiva
* Proteção contra ataques de força bruta com Fail2ban
* Auditoria de segurança com Lynis

Essas medidas reduziram a superfície de ataque e aumentaram a resiliência do servidor contra acessos não autorizados.

---

10. Ferramentas utilizadas

Kali Linux
Nmap
Hydra
Fail2ban
UFW
Lynis
Ubuntu Server 24.04

---

11. Possíveis melhorias futuras

Implementar autenticação SSH baseada em chaves.
Configurar monitoramento de logs.
Implementar IDS/IPS para detecção de intrusão.
Aplicar políticas de segurança adicionais no sistema operacional.

