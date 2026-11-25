# Laboratorio no Cisco Packet Tracer

## Equipamentos sugeridos

- 2 switches multicamada Cisco 3560-24PS: `CORE-A` e `CORE-B`.
- 5 switches Cisco 2960: um por predio e um para o centro de dados.
- 1 ASA 5505, 1 roteador ISP e 1 Cloud.
- Servidores para DHCP, DNS e intranet; PCs, impressoras, telefones IP e APs.

Se uma versao do Packet Tracer nao oferecer um dos modelos ou comandos,
selecione um modelo equivalente que suporte a funcao e adapte as interfaces.

## Montagem

1. Posicione os dois switches L3 no nucleo e ligue-os pelas portas
   `GigabitEthernet0/1` e `GigabitEthernet0/2`, que formarao o Port-channel 1.
2. Ligue cada switch de acesso ao CORE-A e ao CORE-B por trunks independentes.
   Nao configure EtherChannel nesses uplinks duplos.
3. Conecte servidores ao switch do centro de dados em portas de acesso VLAN 50.
   Configure DHCP `10.20.50.10/24`, DNS `10.20.50.11/24` e intranet
   `10.20.50.20/24`; gateway `10.20.50.1`.
4. Conecte o ASA ao CORE-A: rede interna `10.20.250.0/30`. Conecte a interface
   externa do ASA ao roteador ISP na rede `203.0.113.0/30`. Conecte tambem um
   servidor de teste a uma porta LAN do ISP; use `198.51.100.10/24` com gateway
   `198.51.100.1`.
5. Aplique os arquivos de `configs/`. Ajuste nomes e identificadores de portas
   de acordo com o cabeamento real no arquivo.
6. No servidor DHCP, crie um pool para cada VLAN dinamica da tabela de
   enderecamento. Configure gateway `.1`, DNS `10.20.50.11`, mascara
   `255.255.255.0` e inicio de faixa `.50`. Exclua enderecos reservados.
7. No servidor DNS, crie registros A como `intranet.campus.local` apontando
   para `10.20.50.20`. Configure o servidor intranet com endereco estatico.
8. Salve o arquivo como `campus-universitario.pkt` na raiz deste projeto.

## Verificacao

1. Confirme VLANs e trunks com `show vlan brief` e `show interfaces trunk`.
2. Confirme Port-channel e STP com `show etherchannel summary` e
   `show spanning-tree vlan 10`.
3. Confirme os gateways ativos com `show standby brief`. VLANs impares devem
   preferir CORE-A; VLANs pares, CORE-B.
4. Em um PC de cada VLAN, selecione DHCP e verifique IP, mascara, gateway e DNS.
5. Teste ping ao gateway virtual e a `10.20.50.20`; teste resolucao de
   `intranet.campus.local` no navegador ou com `nslookup`, se suportado.
6. Teste acesso externo por um servidor de teste na rede do ISP. Verifique NAT
   e politica do firewall.
7. Desconecte um uplink de acesso e confirme que o caminho alternativo assume.
   Depois desligue um nucleo e confirme que o HSRP e os gateways continuam
   acessiveis pelo outro.
8. Registre resultados e eventuais diferencas de suporte do Packet Tracer.

Um ping de primeira tentativa pode expirar enquanto ARP e protocolos de
redundancia convergem. Repita-o e confirme o estado dos enlaces antes de
classificar como falha.
