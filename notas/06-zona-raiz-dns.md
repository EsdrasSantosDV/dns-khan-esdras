# Zona raiz do DNS

Resumo do material do curso sobre **quem opera a raiz**, **por que existem 13 “servidores raiz”**, **anycast** e **ordens de grandeza de tráfego**. Complementa [Delegação de zona DNS](04-delegacao-zona-dns.md).

## Quem “é dono” da zona raiz?

Se a zona raiz **deixasse de delegar** um TLD importante (por exemplo **`.com`**), grande parte da internet **deixaria de resolver** nomes sob esse TLD — o poder concentrado na raiz é enorme.

Na prática:

- A zona raiz é administrada no âmbito da **ICANN** (*Internet Corporation for Assigned Names and Numbers* — em português costuma aparecer como **Corporação da Internet para Atribuição de Nomes e Números**).
- A **IANA** (função dentro da estrutura ICANN) **publica** a lista oficial de operadores e dados da raiz.

Ou seja: há **governança** e **publicação** centralizada dos dados da raiz; a **infraestrutura** que responde às consultas é **distribuída** entre várias organizações.

## Os “13 servidores raiz” e as 12 organizações

Existem **13 identificadores** de servidor raiz na literatura clássica (**`a.root-servers.net`** … **`m.root-servers.net`** — letras **A a M**).

São **12 organizações** que **operam** esses 13 pontos (uma delas opera **dois** dos treze — no material do curso cita-se **Verisign** como exemplo de operadora com mais de um servidor raiz). O mix inclui entidades com fins lucrativos e sem fins lucrativos, educacionais e governamentais.

### Por que o número 13?

Uma explicação **prática** usada em cursos: há um limite útil de quantos registros **NS** “cabem” confortavelmente numa resposta DNS clássica em **UDP** sem fragmentação problemática; por isso **adicionar um 14.º** identificador raiz “no papel” não era trivial como mudança histórica (hoje há também **DNS sobre TCP**, **EDNS**, etc. — o argumento é **histórico/engineering**, não uma regra eterna).

Trate como **modelo mental**, não como especificação rígida atual.

## Lista “bem conhecida” (*well-known*)

Os resolvedores **não descobrem** a raiz por consulta aleatória: a lista de **IPs dos servidores raiz** é **pré-configurada** em software DNS e sistemas operativos (equivalente ao ficheiro **`named.root`** / *root hints*).

Do ponto de vista técnico, a zona raiz **não é mágica**: é **um conjunto de registros**, sobretudo **NS** (e dados relacionados), como qualquer zona — só tem um papel **único** na hierarquia.

## Ficheiro de zona da raiz (*root zone file*)

A ICANN publica o **ficheiro de zona** da raiz com **sintaxe standard** de zona DNS (BIND e outros). Esse ficheiro:

- lista os **NS** da raiz (`a.root-servers.net`, `b.root-servers.net`, …);
- para cada nome de servidor raiz, inclui registos **A** (IPv4) e **AAAA** (IPv6) com os endereços publicados.

Formato legível por humanos e por servidores autoritativos da raiz.

## Resiliência: falhas e anycast

O DNS foi desenhado para que, se um servidor referido **não responder**, o cliente pode **tentar outro** da lista.

Em **teoria**, se **vários** dos 13 pontos falhassem **ao mesmo tempo**, ainda poderia haver caminho para a raiz — desde que restem operadores acessíveis (na prática depende de rede, routing e política).

### Por que “só 13 IPv4” não é necessariamente fragilidade extrema?

Cada endereço IP “de um servidor raiz” na lista costuma ser servido por **anycast**: **muitas máquinas** em locais diferentes **partilham o mesmo IP**; o routing envia o tráfego para o nó **mais próximo**. Um nó pode ser **retirado** da anycast sem mudar o número “13” na lista publicada.

O site **[root-servers.org](https://root-servers.org)** mostra **instâncias físicas** ao redor do mundo (ordem de **milhares** no total — números mudam; o curso menciona **>1700** servidores servindo a zona raiz).

Em uma cidade (ex.: Amsterdã), pode haver **várias instâncias** correspondentes a **letras raiz diferentes**.

## Escala de tráfego

A ICANN publica **estatísticas** da infraestrutura raiz. Em ordem de grandeza do curso:

- da ordem de **~100 000 consultas por segundo** **num** servidor DNS raiz (referência do exemplo da aula);
- extrapolando: **>1 milhão de pedidos/s** para o conjunto da zona raiz em condições típicas.

Valores exatos variam ao longo do tempo — use os dashboards oficiais para números atuais.

## Recursos e referências

- [Lista de servidores raiz (IANA)](https://www.iana.org/domains/root/servers)
- [Ficheiro de hints da raiz (`named.root`)](https://www.internic.net/domain/named.root)
- [root-servers.org — localização das instâncias](https://www.root-servers.org/)
- [Estatísticas públicas ICANN (Grafana)](https://stats.dns.icann.org/stats/d/wom-ext-5minagg-qtype/query-types-qtype?orgId=1)
- Artigo NsLookup.io: [What is the DNS root zone?](https://www.nslookup.io/learning/what-is-the-dns-root-zone/)
