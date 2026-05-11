# `dig +trace` e `+nodnssec`

Referência rápida para lembrar o que o comando faz. Relaciona com a cadeia de delegações em [Delegação de zona DNS](04-delegacao-zona-dns.md).

## Comando

```bash
dig +trace blog.esdrassystems.com.br +nodnssec
```

A ordem dos argumentos costuma ser flexível no `dig`; equivalente comum:

```bash
dig +trace +nodnssec blog.esdrassystems.com.br
```

## O que é `dig`

**`dig`** (Domain Information Groper) é a ferramenta de linha de comando para consultas DNS (pacote **BIND** / `dnsutils` em muitas distribuições). Mostra a **pergunta**, a **resposta** e metadados (TTL, servidor que respondeu, etc.).

## O que faz `+trace`

- **`+trace`** faz o `dig` **imitar um resolvedor recursivo**, mas **passo a passo**, começando nos **servidores raiz**.
- Ele **não** usa só o seu resolver local (127.0.0.53 / ISP) para devolver o resultado final de uma vez; em vez disso:
  1. consulta a **raiz** (`.` → lista de NS da raiz);
  2. segue as **delegações** (registros **NS**) para cada nível (ex.: `.` → `br.` → `esdrassystems.com.br.` → nome pedido);
  3. até obter a resposta **autoritativa** (A, AAAA, CNAME, etc.) ou um erro (NXDOMAIN, etc.).

Assim você **vê a árvore de delegação** na prática — ideal para estudar DNS ou depurar “quem responde por este domínio?”.

## O que faz `+nodnssec`

- **`+nodnssec`** desliga o comportamento **DNSSEC** nesta sequência de consultas (não pede validação em cadeia, não foca em registros `DNSKEY`/`DS`/etc.).
- A saída fica **mais simples** quando você só quer ver **delegações NS** e o registo final (A/AAAA), sem o ruído da cadeia DNSSEC.

Se precisar de depuração **com** DNSSEC, use **`+dnssec`** em vez de `+nodnssec`.

## Como ler a saída (resumo)

- Blocos `;; Received … from IP#53(...)` indicam **de qual servidor** veio cada resposta.
- Linhas **`IN NS`** = delegação para a zona seguinte.
- No fim, registos **`IN A`** / **`IN AAAA`** (ou `CNAME`) = resposta para o nome que pediu.
- Avisos como **`UDP setup ... IPv6 ... network unreachable`** aparecem quando o `dig` tenta contactar servidores só em **IPv6** e a rede local não tem rota IPv6 — é comum e **não invalida** o resto do trace se depois houver resposta por IPv4.

## Exemplo real (executado neste ambiente)

Trecho ilustrativo para `blog.esdrassystems.com.br` (pode variar com o tempo e com a rede):

```text
; <<>> DiG 9.18.x <<>> +trace blog.esdrassystems.com.br +nodnssec

.			523	IN	NS	a.root-servers.net.
...
;; Received ... from 127.0.0.53#53 ...

br.			172800	IN	NS	a.dns.br.
...
;; Received ... from ... (c.root-servers.net) ...

esdrassystems.com.br.	3600	IN	NS	ns1.dns-parking.com.
esdrassystems.com.br.	3600	IN	NS	ns2.dns-parking.com.
;; Received ... from ... (f.dns.br) ...

blog.esdrassystems.com.br. 14400 IN	A	187.77.47.217
;; Received ... from ... (ns2.dns-parking.com) ...
```

Interpretação rápida: raiz → **`.br`** → zona **`esdrassystems.com.br`** nas **Cloudflare registrar parking NS** → **`blog`** resolve para o **A** mostrado.

## Ver também

- [Delegação de zona DNS — ferramentas](04-delegacao-zona-dns.md#ferramentas) (`dig +trace`, dnsviz)
- Manual: `man dig` (secções **+trace**, **+dnssec** / **+nodnssec**)
