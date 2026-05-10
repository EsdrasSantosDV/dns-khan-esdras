# O que é DNS? — Visão geral (resumo)

> Conteúdo baseado no artigo [“What is DNS?”](https://www.nslookup.io/learning/what-is-dns/) do NsLookup.io (CC BY 4.0). Texto do curso/material em português consolidado aqui como estudo.

Complementa a [introdução do curso](01-introducao-o-que-e-dns.md).

## Função principal

O **Sistema de Nomes de Domínio (DNS)** é peça central da internet. Ele:

1. **Traduz** nomes legíveis para humanos em endereços **legíveis para máquinas** (principalmente IPs).
2. Atua como **diretório global** para outros serviços (e-mail, delegações, aliases, etc.).

**Ideia:** todo dispositivo na internet tem um endereço **IP**. A comunicação (ex.: celular ↔ servidor web) usa IPs. IPs (sobretudo IPv6) são difíceis de memorizar; o DNS permite usar nomes como `www.google.com` em vez de algo como `2607:f8b0:400a:080a::2004`.

## Como funciona (modelo)

- Analogia: **lista telefônica** de nomes → IPs.
- O DNS é um **banco de dados global** formado por **milhões de zonas DNS**.
- Cada zona cobre um **pedaço da árvore** de nomes; as zonas se **encadeiam por delegações** e formam um único espaço de nomes.

### Hierarquia (árvore)

- No **topo**: **zona raiz** (root).
- Abaixo: **TLDs** (domínios de nível superior), ex.: `.org`, `.com`, `.net`, `.uk`.
- Uma zona como `wikipedia.org` contém nomes **dessa** árvore, ex.: `en.wikipedia.org`, `fr.wikipedia.org`.

## Resolução e recursão (fluxo típico)

1. O usuário digita um nome (ex.: `www.google.com`) no navegador.
2. Um software no dispositivo envia uma **consulta DNS**, em geral ao **servidor DNS recursivo** do **ISP** (ou outro resolver configurado).
3. O **recursivo** pode precisar enviar **outras consultas**, seguindo a **cadeia de delegações** da raiz até zonas autoritativas, até obter o IP (ou outros dados) para o nome pedido.
4. A **resposta** volta ao computador; o navegador **conecta-se ao IP**. Esse processo de “seguir delegações até a resposta” é a **recursão**.

### Passos ilustrativos (`www.example.org`)

Ordem lógica (simplificada):

1. Laptop → consulta ao **resolver recursivo**.
2. Recursivo → **servidor raiz** → delegação para `.org`.
3. Recursivo → **autoritativo de `.org`** → delegação para `example.org`.
4. Recursivo → **autoritativo de `example.org`** → registros de endereço para `www.example.org`.
5. Recursivo → devolve a resposta ao laptop.

**Na prática:** nem toda consulta percorre tudo isso — muitas respostas já estão em **cache** no recursivo.

## Propriedade de nomes e governança

- **Registro de domínio:** compra-se o nome junto a um **registrador**.
- É preciso **criar/hospedar uma zona DNS** (muitos registradores oferecem hospedagem de zona; dá para operar a zona por conta própria).
- O registrador insere o domínio no **registro** do **TLD** adequado.
- **ICANN** supervisiona registradores e TLDs (segurança, boas práticas, mitigação de abuso).

## Autoritativo vs. recursivo

| Tipo | Papel |
|------|--------|
| **Autoritativo** | Responde com dados das **zonas** das quais é **autoridade** (delegação + registros daquela fatia da árvore). |
| **Recursivo** | Pode tratar **qualquer** nome; usa **recursão** (e cache) para obter respostas junto aos autoritativos. |

Se o recursivo **já tem** a resposta em cache (e o **TTL** ainda vale), responde **sem** refazer a cadeia inteira.

## Cache de DNS

**Objetivo:** menos trabalho e menos tempo por consulta.

- Ao receber uma resposta, o recursivo (e outros níveis) podem **guardar uma cópia** até o **TTL** do registro expirar.
- **Efeitos:** menor **latência**, menos consultas repetidas, menos carga em recursivos e **autoritativos**.

### Onde costuma haver cache

- **Servidores DNS recursivos** (vários clientes compartilham o mesmo cache).
- **Sistema operacional** do dispositivo.
- **Aplicações** (ex.: **cache do navegador**).

### Invalidação / limpeza

- Registros saem do cache quando o **TTL expira**.
- Alguns resolvedores **ajustam** TTLs muito baixos ou muito altos (política local, memória, proteção contra rajadas).
- Para “ver mudanças mais cedo”, às vezes é preciso **limpar cache** no SO ou no navegador (propagação percebida).

## Além do “nome → IP”

O DNS também transporta/realiza:

| Registro / uso | Função (resumo) |
|----------------|------------------|
| **NS** | **Delegações** — sustenta a hierarquia global. |
| **MX** | Entrega de **e-mail** por domínio. |
| **SPF, DKIM, DMARC** | **Autenticação** e política anti-abuso de e-mail. |
| **CNAME** | **Aliases** — reutilizar IPs/lógica sem duplicar endereços. |
| **SRV** | Alguns apps usam para **descobrir serviço/porta** e balanceamento. |
| **PTR** | **IP → nome** (resolução reversa). |
| Uso interno | Ex.: **Kubernetes** e muitos outros sistemas usam DNS internamente. |

## Referência

- [What is DNS? — NsLookup.io](https://www.nslookup.io/learning/what-is-dns/) (atualizado em 22/05/2023 no original; licença CC BY 4.0).
