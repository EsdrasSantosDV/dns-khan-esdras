# Delegação de zona DNS

> Resumo do artigo [DNS zone delegation](https://www.nslookup.io/learning/zone-delegation/) (NsLookup.io, atualizado em 28/03/2023, CC BY 4.0). Complementa [árvore, zonas e rótulos](03-dns-banco-federado-arvore-zonas-rotulos.md) e [visão geral do DNS](02-o-que-e-dns-visao-geral.md).

## Definição

**Delegação de zona** é a forma como a **zona pai** avisa os resolvedores de que a **autoridade** sobre uma **zona filha** está a cargo de **outro conjunto de servidores** (outra operação, outra infraestrutura).

O DNS é enorme; cada zona tem dono e gestão **independentes**. Exemplo clássico: a zona **`.com`** é operada pela **Verisign**; a zona **`google.com`** pelo **Google**. A Verisign **delega** `google.com` ao Google.

## Delegações são registros NS

- Uma delegação é um **conjunto de registros NS** com os **hostnames** dos servidores **autoritativos** da zona filha.
- Cada NS tem **TTL** (segundos) e, nos dados, o **nome DNS** do servidor de nomes — que precisa **resolver para IP** (A/AAAA) para o resolvedor conseguir contactar o software DNS.

Exemplo (domínio reservado **`example.org`**):

```text
example.org. 86400 NS ns1.example.org.
example.org. 86400 NS ns2.example.org.
example.org. 86400 NS ns3.example.org.
example.org. 86400 NS ns4.example.org.
```

Esse mesmo conjunto de NS aparece em **dois contextos** diferentes (ver secção “dualidade” abaixo): na zona **`example.org`** e na zona **`org`** (cópia na pai para **delegar**).

**Convenções:** nomes `ns1`, `ns2` são comuns, mas não obrigatórios. Os servidores podem estar **dentro** da zona que servem ou **fora** dela.

**Quantidade:** não há número fixo exigido; em muitos casos usa-se **quatro** NS iguais no conjunto de delegação e no autoritativo. Poucos NS → fragilidade; muitos → respostas maiores.

## Tudo começa na raiz

- A árvore começa na **[zona raiz](https://www.nslookup.io/learning/what-is-the-dns-root-zone/)**, notada muitas vezes como **`.`**.
- Resolvedores **recursivos** vêm com a lista de IPs dos **servidores raiz** (hoje **13** “letras”, tipicamente **1× IPv4 + 1× IPv6** cada um → **26 endereços** lógicos).
- Na prática esses IPs são servidos por **muitas** máquinas ( **anycast** ), não um único servidor por IP.
- Operadores incluem ICANN, Verisign, ISC (BIND), NASA, exército dos EUA, universidades, etc. Sem a raiz, a resolução global quebra.

## Zonas filhas e encadeamento

Cada zona (em especial a raiz) pode ter **zonas filhas** de outra entidade. Na **zona pai**, no **nome** que inicia a filha, publicam-se os **NS** da delegação.

O recursivo, ao ver essa delegação, **reenvia** a consulta aos autoritativos da filha. Isso pode repetir **várias vezes** para um único nome.

### Recursão em ação (`www.example.org`)

1. Consulta a um **servidor raiz** → não há dados de `example.org`, mas há **delegação para `org`** (lista de NS de `.org`).
2. Consulta a um NS de **`org`** → delegação para **`example.org`**.
3. Consulta a um NS de **`example.org`** → resposta final (A/AAAA, CNAME, etc.) para `www.example.org`.

Neste exemplo há **duas** delegações “no caminho” (raiz→org, org→example.org); na prática podem existir **mais**. Não há limite rígido “oficial” baixo; em teoria cenários estranhos podem ir longe — na prática **cinco ou seis** saltos já é considerado **alto**.

## Cache

Sem cache, cada nome “frio” exigiria **uma consulta por nível** de delegação (ordem de **dezenas de ms** cada), deixando tudo lento.

- Na **primeira** resolução, o recursivo percorre a cadeia.
- Os **NS** de cada delegação ficam em **cache** pelo **TTL** (muitas vezes horas ou dias).
- Depois, o recursivo costuma ir **direto** ao autoritativo adequado → cache “aquecido”.

### TTL dos NS de delegação

Registradores muitas vezes escolhem o TTL pelos clientes. Recomendações típicas do artigo:

- **TTL longo** nas delegações é desejável; **~1 dia** é comum.
- Evitar TTL **muito baixo** (menos de uma hora) ou **muito alto** (mais de uma semana).
- Pode usar o **mesmo TTL** no conjunto da **delegação (pai)** e no conjunto **autoritativo (filho)**.

## A “dualidade” dos registros NS

No **ápice** de uma zona há sempre **SOA** + conjunto **NS** (servidores autoritativos **daquela** zona). Ver [tipo SOA](https://www.nslookup.io/learning/dns-record-types/soa/).

**Ao mesmo tempo**, na **zona pai**, no nome da filha, existe **outro** conjunto de **NS** — a **delegação**.

Assim, para um nome como `example.org`:

1. **Dentro da zona `example.org`**: NS (e SOA) — “estes são os meus servidores autoritativos”.
2. **Dentro da zona `org`**: NS em `example.org` — “a autoridade de `example.org` está nestes hosts” (normalmente **iguais** aos de 1, quando o registrador copia).

**Recomenda-se** que os dois conjuntos sejam **idênticos**; não é obrigatório. Resolvedores podem usar qualquer um; quando têm **ambos**, tendem a **preferir** o conjunto **autoritativo na zona filho**. Priorização descrita na [RFC 2181, secção 5.4.1](https://www.rfc-editor.org/rfc/rfc2181#section-5.4.1).

## Glue records (registros “cola”)

**Glue** são cópias de **A** / **AAAA** na **zona pai** para os hostnames dos NS **quando esses hostnames caem dentro da zona delegada**.

Problema **ovo e galinha**: a delegação diz “pergunta a `ns1.example.org`”, mas para contactar `ns1.example.org` o resolvedor precisaria seguir a delegação de `example.org` — sem IP não consegue.

**Solução:** a resposta de delegação da **pai** inclui **NS + glue** (A/AAAA dos `nsX.example.org`). Registradores costumam **exigir** glue quando aplicável.

Exemplo (delegação + glue):

```text
example.org. 86400 NS ns1.example.org.
example.org. 86400 NS ns2.example.org.

ns1.example.org. 86400 A 1.2.3.4
ns2.example.org. 86400 A 1.2.3.5
```

## Falhas comuns

### Delegação em falta ou apagada

Se na **pai** não existir delegação para a filha, **por definição** os nomes “abaixo” dessa delegação **não existem** no DNS → clientes podem ver **NXDOMAIN**.

### Delegação “lame” (inválida / inconsistente)

Os nomes nos **NS** da delegação apontam para servidores **errados**, **desligados** ou desatualizados (ex.: mudou-se para `rainier` / `denali` mas o registrador ainda lista `ns1` / `ns2`). Definição e termos: [RFC 8499](https://datatracker.ietf.org/doc/html/rfc8499).

Sintoma típico: recursivo não consegue contactar autoritativos válidos → após tentativas, **SERVFAIL**.

**Boas práticas:** alinhar **sempre** delegação no registrador com a realidade da zona; **sobrepor** mudanças por pelo menos o **TTL** dos NS; alterar NS **um a um** quando possível.

## Privacidade: minimização de nome (QNAME minimization)

Historicamente o nome **completo** da consulta era enviado a cada salto, **revelando** mais do que o necessário (ex.: raiz não precisa de `www.example.org` para delegar `.org`).

**DNS Query Name Minimisation:** [RFC 7816](https://datatracker.ietf.org/doc/html/rfc7816), atualizada pela [RFC 9156](https://datatracker.ietf.org/doc/html/rfc9156). Não exige mudar zonas nem delegações — é comportamento do **resolvedor**.

## Ferramentas

- **`dig +trace`**: segue da raiz pelas delegações (BIND / pacote `dnsutils` em muitas distros).
- **[dnsviz.net](https://dnsviz.net)**: útil, com foco forte em **DNSSEC** (curva de aprendizado maior).

## Referências

- [DNS zone delegation — NsLookup.io](https://www.nslookup.io/learning/zone-delegation/)
- [What is a DNS zone?](https://www.nslookup.io/learning/what-is-a-dns-zone/)
- [RFC 2181 §5.4.1](https://www.rfc-editor.org/rfc/rfc2181#section-5.4.1) (prioridade de dados)
- [RFC 8499](https://datatracker.ietf.org/doc/html/rfc8499) (lame delegation)
- [RFC 7816](https://datatracker.ietf.org/doc/html/rfc7816) / [RFC 9156](https://datatracker.ietf.org/doc/html/rfc9156) (QNAME minimization)
