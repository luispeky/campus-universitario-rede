# Arquitetura da rede

## Objetivos e premissas

- Interligar quatro predios universitarios e um centro de dados.
- Separar departamentos, usuarios, voz, servidores, impressoras e gestao em
  VLANs distintas.
- Fazer roteamento entre VLANs no nucleo L3, com gateway virtual HSRP.
- Usar DHCP centralizado, DNS interno e firewall/NAT para acesso externo.
- Manter conectividade quando um nucleo ou um enlace de uplink falhar.
- Isolar visitantes e limitar o acesso dos usuarios a redes de infraestrutura.

O projeto e um laboratorio didatico, nao um projeto executivo. Para implantacao
real, dimensione capacidade, cobertura Wi-Fi, energia, fibra, licencas,
monitoramento e politicas de seguranca conforme o campus.

## Camadas e componentes

| Camada | Componentes | Funcao |
|---|---|---|
| Borda | ASA 5505, roteador ISP e nuvem | Inspecao, NAT e ligacao externa |
| Nucleo | CORE-A e CORE-B (switches L3) | SVIs, HSRP, roteamento e uplinks |
| Acesso | Um ou mais switches L2 por predio | Portas de usuario, APs e impressoras |
| Servicos | Servidores DHCP/DNS e intranet | Servicos centrais na VLAN 50 |

Cada switch de acesso tem um trunk para cada nucleo. Use Rapid-PVST e
prioridades por VLAN para controlar o caminho ativo. Os nucleos compartilham
VLANs num Port-channel LACP, evitando loops nesse enlace. Nao agregue em um
mesmo EtherChannel enlaces que terminem em dois switches de nucleo distintos.

## VLANs e sub-redes

Mascara padrao de usuario: `/24` (`255.255.255.0`). O gateway virtual HSRP e
sempre o primeiro endereco utilizavel. CORE-A usa `.2` e CORE-B usa `.3` em
cada VLAN.

| VLAN | Nome | Rede | Gateway HSRP | Uso |
|---:|---|---|---|---|
| 10 | ADMIN | 10.20.10.0/24 | 10.20.10.1 | Administracao e financeiro |
| 20 | ENGENHARIA | 10.20.20.0/24 | 10.20.20.1 | Laboratorios e docentes |
| 30 | SAUDE | 10.20.30.0/24 | 10.20.30.1 | Faculdade de Saude |
| 40 | BIBLIOTECA | 10.20.40.0/24 | 10.20.40.1 | Biblioteca |
| 50 | SERVIDORES | 10.20.50.0/24 | 10.20.50.1 | Servicos centrais |
| 60 | WIFI-ALUNOS | 10.20.60.0/24 | 10.20.60.1 | Alunos e dispositivos pessoais |
| 70 | WIFI-VISITANTES | 10.20.70.0/24 | 10.20.70.1 | Visitantes, somente Internet |
| 80 | VOZ | 10.20.80.0/24 | 10.20.80.1 | Telefones IP |
| 90 | GERENCIA | 10.20.90.0/24 | 10.20.90.1 | Gestao de equipamentos |
| 100 | IMPRESSORAS-IOT | 10.20.100.0/24 | 10.20.100.1 | Impressoras e dispositivos IoT |
| 999 | NATIVA-NAO-USAR | Sem SVI | — | VLAN nativa sem portas de usuario |

As VLANs 50 e 90 devem usar enderecos estaticos ou reservas. Servidores
propostos: DHCP `10.20.50.10`, DNS `10.20.50.11` e intranet
`10.20.50.20`. Reserve `.1` a `.49` para gateways e enderecos estaticos; o
escopo DHCP pode iniciar em `.50`.

## Roteamento, DHCP e DNS

As SVIs dos dois nucleos implementam HSRP. CORE-A e ativo preferencial nas
VLANs impares; CORE-B nas pares. Cada SVI tem `ip helper-address
10.20.50.10` para encaminhar DHCP ao servidor central. Crie no servidor DHCP
um escopo por VLAN, apontando o gateway para o endereco virtual `.1` e o DNS
para `10.20.50.11`.

Ative `ip routing` nos nucleos. O enlace roteado para o firewall usa
`10.20.250.0/30`: firewall `10.20.250.1`, CORE-A `10.20.250.2`. O caminho de
retorno do firewall para `10.20.0.0/16` aponta para o nucleo. No Packet
Tracer, mantenha apenas um enlace roteado ativo entre o firewall e CORE-A, ou
configure roteamento redundante explicitamente antes de adicionar um segundo.

## Politica de seguranca

- Firewall: permitir conexoes originadas de dentro para fora, aplicar NAT e
  negar trafego externo nao solicitado.
- Visitantes (VLAN 70): permitir DHCP/DNS e acesso a Internet; negar redes
  privadas do campus.
- Usuarios: permitir servicos necessarios, como DHCP, DNS, intranet e
  impressao; restringir acesso a VLANs de gerencia e servidores.
- Gerencia (VLAN 90): permitir acesso administrativo somente a equipe de TI.
- Portas nao usadas: desativar e colocar na VLAN 999; nao usar a VLAN nativa
  para dispositivos finais.

Os arquivos de nucleo incluem uma ACL para visitantes que permite DHCP/DNS,
nega acesso ao bloco interno `10.20.0.0/16` e libera outros destinos. As
demais ACLs departamentais sao requisitos para extensao: adicione e teste-as
incrementalmente, evitando bloquear DHCP, DNS, HSRP ou administracao.

## Alta disponibilidade e limites

- HSRP protege o gateway padrao contra falha de um switch de nucleo.
- Dois uplinks por switch de acesso protegem contra falha de um enlace ou nucleo.
- Rapid-PVST evita loops e seleciona o caminho alternativo apos uma falha.
- Neste laboratorio, o ASA esta ligado somente ao CORE-A; se o CORE-A falhar,
  os gateways internos podem continuar pelo CORE-B, mas o acesso a Internet
  sera interrompido. Redundancia de borda exigiria outro enlace e roteamento
  de retorno devidamente configurados.
- O firewall, o DHCP e os servidores aqui sao pontos unicos de falha. Redunde-os
  com um par de firewalls e servidores/escopos redundantes em uma implantacao
  real.
- Packet Tracer oferece uma simulacao simplificada de protocolos e equipamentos;
  confira os comandos suportados pelo modelo escolhido.
