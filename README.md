# SI-Prova3

Nesta avaliação, você deverá criar uma máquina virtual Linux (preferencialmente com Debian Server) para configurar um firewall usando iptables e instalar dois serviços essenciais: SSH e o servidor web Nginx. Crie um script chamado firewall-if.sh com regras que bloqueiem todo o tráfego, exceto a porta 22 (SSH) inicialmente. As portas 80 (HTTP), o protocolo ICMP (ping) e o próprio SSH deverão ser testados individualmente, com bloqueios e liberações controladas pelo script. Em seguida, crie um serviço systemd chamado firewall-if.service para que o firewall possa ser ativado ou desativado com os comandos systemctl start e stop.

Demonstração: (1) o funcionamento normal de SSH, web e ping com o firewall desativado; (2) a ativação do firewall com todas as conexões bloqueadas, mostrando que nenhum dos serviços está acessível; (3) o bloqueio do SSH via firewall (mostrar que a conexão remota cai); (4) o bloqueio das portas 80 (HTTP) e ICMP (ping), com novos testes a cada etapa.

# Apresentação

[Link da Apresentação](https://youtu.be/BeRT20ydCmk)

# Passo a Passo

## Atualizações básicas
apt update
apt upgrade

## Instalando o iptables
apt install nginx openssh-server iptables -y

## Criando o script
nano /usr/local/bin/firewall-if.sh

````
#!/bin/bash

start() {
    # 1. Limpa todas as regras antigas
    iptables -F
    iptables -X

    # 2. Política Básica: Bloquear a entrada e repasse silenciosamente (DROP)
    # Isso diminui a eficiência de ataques, pois deixa o atacante sem resposta
    iptables -P INPUT DROP
    iptables -P FORWARD DROP
    iptables -P OUTPUT ACCEPT 

    # Permite o funcionamento interno do próprio Linux
    iptables -A INPUT -i lo -j ACCEPT
    #iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

    # 3. EXCEÇÕES (Regras de liberação)
    # Deixe sem o '#' para liberar, ou coloque o '#' na frente para bloquear.

    # Libera SSH (Porta 22)
    iptables -A INPUT -p tcp --dport 22 -j ACCEPT

    # Libera HTTP/Nginx (Porta 80)
    iptables -A INPUT -p tcp --dport 80 -j ACCEPT

    # Libera Ping (ICMP)
    iptables -A INPUT -p icmp -j ACCEPT

    echo "Firewall LIGADO!"
}

stop() {
    # Função para desativar o firewall, aceitando tudo (ACCEPT)
    iptables -F
    iptables -X
    iptables -P INPUT ACCEPT
    iptables -P FORWARD ACCEPT
    iptables -P OUTPUT ACCEPT
    echo "Firewall DESLIGADO!"
}

case "$1" in
    start) start ;;
    stop) stop ;;
    restart) stop; start ;;
    *) echo "Use: $0 {start|stop|restart}" ;;
esac
````

## Dando permissão para o script
chmod +x /usr/local/bin/firewall-if.sh

## Criando serviço
nano /etc/systemd/system/firewall-if.service

````
[Unit]
Description=Meu Firewall Iptables
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/firewall-if.sh start
ExecStop=/usr/local/bin/firewall-if.sh stop
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
````

## Reiniciando systemctl
systemctl daemon-reload



# Fase de testes

## Fase 1, tudo desprotegido
systemctl stop firewall-if
(em outro terminal) ping 192.168.56.56
(no navegador) http://192.168.56.56

## Fase 2, tudo bloqueado
sudo nano /usr/local/bin/firewall-if.sh
(comentar as liberações)
systemctl restart firewall-if

## Fase 3, SSH cai
ssh alunos@192.168.56.56

## Fase 4, HTTP e ICMP cai
(em outro terminal) ping 192.168.56.56
(no navegador) http://192.168.56.56

