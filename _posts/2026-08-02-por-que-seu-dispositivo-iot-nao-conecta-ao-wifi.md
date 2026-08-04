---
layout: post
title: "Por que seu dispositivo IoT não consegue conectar ao Wi-Fi (mesmo com a senha certa)"
date: 2026-08-02 00:00:00 +0000
---

Recentemente tentei conectar um brinquedo eletrônico (um [Yoto Player](https://yotoplay.com/), para quem tiver curiosidade) ao Wi-Fi de casa e recebi um genérico e nada útil "falha na conexão". Sem detalhes, sem código de erro, nada.

O que começou como "deve ser a senha errada" virou uma investigação de várias horas que passou por DHCP, DNS, IPv6 e terminou com a descoberta de que meu provedor de internet estava distribuindo um servidor DNS inválido para qualquer dispositivo que dependesse exclusivamente de IPv4.

Esse tipo de problema não é exclusivo do meu roteador nem do meu dispositivo. Ele é comum o suficiente em dispositivos IoT baratos (assistentes de voz, tomadas inteligentes, brinquedos conectados, campainhas, etc.) que achei que valia a pena documentar o processo de debug, porque a causa raiz é quase sempre a mesma: **o dispositivo consegue entrar na rede Wi-Fi, mas não consegue resolver nomes de domínio.**

## O sintoma

A maioria dos dispositivos IoT segue o mesmo fluxo de configuração: escolhemos a rede Wi-Fi, digitamos a senha, e o aparelho tenta se conectar aos servidores do fabricante para finalizar o cadastro. Quando esse processo falha, normalmente aparece uma de duas categorias de erro:

1. **Falha ao entrar na rede Wi-Fi** — senha errada, rede de 5GHz (a maioria dos dispositivos IoT baratos só suporta 2,4GHz), modo de segurança incompatível (WPA3-only em vez de WPA2), etc.
2. **Falha depois de entrar na rede** — o dispositivo consegue se associar ao Wi-Fi e obter um IP, mas não consegue "telefonar para casa" e finalizar o cadastro.

O segundo caso é o mais traiçoeiro, porque tudo *parece* estar funcionando: o dispositivo aparece na lista de clientes conectados do roteador, tem um endereço IP válido, e mesmo assim a configuração falha com uma mensagem genérica.

## Por que "outros dispositivos funcionam" não é uma prova de nada

O meu instinto foi testar se o problema era da rede: "meu notebook e meu celular conectam sem problema, então a rede deve estar OK". Essa suposição é exatamente o que torna esse bug tão difícil de encontrar.

Sistemas operacionais modernos (macOS, iOS, Android, Windows) têm múltiplos mecanismos de fallback para descobrir um servidor DNS válido:

- **DHCPv4**, o mecanismo clássico, onde o roteador informa um servidor DNS via a opção 6 do DHCP.
- **IPv6 Router Advertisements (SLAAC/RDNSS)**, um mecanismo completamente separado, no qual o roteador anuncia servidores DNS via IPv6, independente do que está configurado no DHCPv4.
- **UPnP / TR-064**, protocolos que permitem que aplicativos consultem o gateway diretamente sobre a configuração da conexão WAN, incluindo os servidores DNS que o próprio roteador usa.

Um dispositivo IoT barato, por outro lado, normalmente implementa **apenas o DHCPv4 clássico**. Sem suporte a IPv6, sem UPnP, sem plano B. Se a única informação que ele recebe (a opção 6 do DHCP) estiver quebrada, ele fica sem DNS — mesmo que o resto da sua rede pareça funcionar perfeitamente.

## Debugando: descubra o que o DHCP está realmente entregando

Percebi que o primeiro passo real de debug não era "meu notebook consegue navegar", e sim: **o que exatamente o servidor DHCP estava entregando como servidor DNS?**

No macOS, rodei:

```bash
$ ipconfig getpacket en0
```

Isso mostra o pacote DHCP bruto recebido pela interface, incluindo todas as opções. Reparei na linha `domain_name_server`:

```text
options:
Options count is 7
dhcp_message_type (uint8): ACK 0x5
server_identifier (ip): 192.168.0.1
lease_time (uint32): 0x15180
subnet_mask (ip): 255.255.255.0
router (ip_mult): {192.168.0.1}
domain_name_server (ip_mult): {0.0.0.0}
end (none):
```

Ali estava o problema: `domain_name_server` igual a `0.0.0.0`. Um endereço IP inválido, que nenhum dispositivo consegue usar para resolver nomes de domínio.

No Ubuntu/Debian, o equivalente (usando o NetworkManager) seria:

```bash
$ nmcli device show wlan0 | grep IP4.DNS
```

Em seguida, comparei com o que o sistema operacional estava *realmente usando* para resolver nomes — que pode ser diferente do que veio pelo DHCP, exatamente por causa dos mecanismos de fallback mencionados acima. No macOS:

```bash
$ scutil --dns | grep -A5 "resolver #1"
```

O resultado revelou o resto da história:

```text
resolver #1
  nameserver[0] : 2804:14d:1:0:181:213:132:2
  nameserver[1] : 2804:14d:1:0:181:213:132:3
  if_index : 15 (en0)
  flags    : Request A records, Request AAAA records
```

Servidores DNS válidos — só que via IPv6, completamente diferentes (e desconectados) do que veio pelo DHCPv4. Meu notebook nunca "sentiu" o problema porque, na prática, ele ignorou o DHCPv4 quebrado e usou os servidores anunciados via IPv6.

## Investigando o roteador

Com a hipótese confirmada (DHCPv4 não entrega DNS válido), o próximo passo foi entender *por que*. Entrei no painel administrativo do roteador — um ONT Huawei HG8245W5-6T-V2 fornecido pela Claro Brasil — e fui até a configuração do servidor DHCP.

Duas coisas chamaram atenção:

1. O campo de DNS estava **em branco**.
2. O campo estava **desabilitado** — não dava para digitar nada ali.

Um pouco mais abaixo, na mesma tela, havia duas opções marcadas como ativas: **"Ativar Retransmissão DHCP"** e **"Ativar Opção 125"**.

Pelo que entendi, essa combinação é uma configuração típica de operadoras para cenários de "triple play" (internet + TV + telefone), onde o roteador não atua como um servidor DHCP completo — ele apenas *retransmite* as requisições para um servidor central da operadora, que deveria responder com todas as opções, incluindo o DNS.

O problema: esse servidor central, por algum motivo (provavelmente uma falha de provisionamento do lado da operadora), não está respondendo com um DNS válido. O roteador sabe qual servidor DNS usar — ele mesmo usa esse servidor para sua própria conexão WAN, e é exatamente isso que aparece anunciado via IPv6 — mas essa informação nunca chega até os dispositivos conectados via DHCPv4, provavelmente, porque o roteador está em modo "retransmissão" esperando por uma resposta que nunca chega corretamente.

Tentei também remover o atributo `disabled` do campo de DNS pelas ferramentas de desenvolvedor do navegador, preencher um servidor manualmente e salvar. O resultado: o painel administrativo simplesmente me deslogou e ignorou a alteração. Isso é provavelmente alguma proteção no firmware do roteador.

## A solução: assumir o controle do DHCP

Se o roteador da operadora não ia entregar um DNS válido — e o campo estava literalmente bloqueado na interface —, a saída que encontrei foi parar de depender do DHCP dele.

A ideia é usar algum equipamento com capacidade de **NAT/roteamento real** (não um simples repetidor/extensor de Wi-Fi, que apenas faz *bridge* na mesma rede e portanto herda o mesmo problema) entre o roteador da operadora e a rede doméstica. Na prática, foi isso que fiz:

1. Conectei um extensor de Wi-Fi que eu já tinha em casa (um TP-Link RE305) ao roteador da operadora.
2. Desativei por completo o servidor DHCP do roteador da operadora, para evitar dois servidores DHCP respondendo na mesma rede (uma fonte clássica de instabilidade — [DHCP starvation](https://en.wikipedia.org/wiki/DHCP_starvation_attack) à parte, ter dois servidores DHCP ativos na mesma rede sem coordenação gera respostas conflitantes e endereços IP duplicados).
3. Configurei manualmente o servidor DHCP do extensor com um servidor DNS válido — no meu caso, `8.8.8.8` (Google) e `1.1.1.1` (Cloudflare) — e o gateway apontando para o roteador da operadora, em vez de depender do modo automático (que, por padrão, se desativa sozinho quando detecta outro servidor DHCP na rede).
4. Conectei o dispositivo IoT à rede desse extensor.

Um detalhe que me pegou de surpresa: como o extensor estava conectado via Wi-Fi ao roteador principal (em vez de cabo de rede), ele também precisava conseguir um IP para si mesmo nessa rede para funcionar como ponte. Acabei desativando o DHCP do roteador principal antes de configurar um IP fixo para o extensor, e ele ficou sem conseguir se conectar — um problema de "ovo e galinha" que derrubou minha rede inteira por alguns minutos até eu perceber o que tinha acontecido. Por isso vale um cuidado extra: a faixa de IPs do novo servidor DHCP não pode se sobrepor a nenhum IP estático já em uso na rede — incluindo esse IP estático que o próprio extensor passa a usar para se conectar ao roteador principal.

Como exemplo, a configuração que usei no extensor foi mais ou menos essa:

- **IP estático do extensor:** `192.168.0.70` (fora da faixa do DHCP, para não haver conflito)
- **Faixa de IPs do DHCP:** `192.168.0.100` – `192.168.0.200`
- **Máscara de sub-rede:** `255.255.255.0`
- **Gateway:** `192.168.0.1` (o roteador da operadora)
- **DNS primário/secundário:** `8.8.8.8` / `1.1.1.1`

## Resumindo o processo que segui

Se alguém estiver enfrentando um dispositivo IoT que "conecta ao Wi-Fi mas não termina a configuração", esse foi mais ou menos o roteiro que segui:

1. **Confirmar a rede e o modo de segurança**: 2,4GHz (não 5GHz), WPA2 (não WPA3-only). A maioria dos dispositivos baratos não suporta WPA3 nem 5GHz.
2. **Verificar o que o DHCP estava realmente entregando** (`ipconfig getpacket` no macOS, `nmcli` no Ubuntu/Debian), prestando atenção no servidor DNS.
3. **Não confiar no fato de outros dispositivos estarem funcionando** — vale checar explicitamente se o servidor DNS ativo no notebook ou celular veio do DHCPv4 ou de outra fonte (IPv6, DNS configurado manualmente, etc.).
4. **Olhar a configuração do servidor DHCP no painel do roteador**, procurando por menções a "DHCP Relay", "L2 Relay" ou "Option 125" — sinais de que o roteador foi configurado pela operadora para um cenário de rede corporativa/triple-play, e não para uma rede doméstica simples.
5. **Se o campo de DNS estiver vazio ou bloqueado**, em vez de insistir pelo lado do roteador da operadora, faz sentido introduzir um equipamento próprio com um servidor DHCP funcional entre o roteador e o resto da rede.
6. **Em último caso, vale tentar ligar para o suporte técnico da operadora** — ainda não fiz isso, mas pretendo.

## Sobre qual DNS usar

Depois de assumir o controle do DHCP, fiquei me perguntando: devo usar o DNS da própria operadora ou um público como Google (`8.8.8.8`) ou Cloudflare (`1.1.1.1`)?

Prós de usar DNS público:
- Alta disponibilidade e desempenho consolidados globalmente.
- Políticas de privacidade públicas e auditáveis (especialmente no caso da Cloudflare).
- Evita práticas comuns de operadoras, como injeção de anúncios em páginas de erro ou redirecionamento de domínios não encontrados.
- Te desacopla de qualquer instabilidade futura na infraestrutura da operadora — como a que motivou este post.

Prós de usar o DNS da operadora:
- Latência potencialmente um pouco menor, por estar na mesma rede.
- Possivelmente um roteamento um pouco mais otimizado para CDNs baseadas na localização do seu resolver.

Depois de passar por todo esse processo e ver na prática como essa dependência da infraestrutura de DHCP/DNS da minha operadora pode falhar, minha escolha foi clara: desacoplar dessa dependência e usar DNS público.
