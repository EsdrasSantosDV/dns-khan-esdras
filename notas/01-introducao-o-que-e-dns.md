# DNS para desenvolvedores — Introdução: o que é DNS?

## Objetivo do curso

- Entender **como o DNS funciona na prática**.
- Configurar **registros DNS** com confiança.
- **Depurar** problemas comuns de DNS.

## O que é DNS (modelo mental)

O DNS pode ser pensado como um **enorme banco de dados** que associa **nomes de domínio** às **informações** ligadas a esses nomes (por exemplo, endereços IP).

**Fato importante:** navegadores não “conectam” ao nome do domínio diretamente — eles precisam de um **endereço IP**. Por isso, ao inspecionar a aba **Rede** das DevTools (por exemplo, ao abrir a Wikipédia), você vê conexões para **IPs**, não só para `wikipedia.org`.

### O que acontece nos bastidores

1. O cliente (ex.: Chrome) faz uma **consulta DNS** pelo nome (ex.: `wikipedia.org`).
2. A resposta traz **um ou mais endereços IP** (IPv4 e/ou IPv6).
3. O navegador escolhe um destino e abre a conexão para exibir a página.

### Replicar na linha de comando

Com a ferramenta **Dig**, dá para pedir registros específicos. Por exemplo, registros **AAAA** (IPv6) para um domínio:

```bash
dig wikipedia.org AAAA
```

Domínios diferentes podem ter **vários IPs** (balanceamento, CDN, redundância, etc.) — o exemplo clássico é um site grande com muitos endereços.

## Por que “uma tabela gigante” não é um único arquivo?

Uma única tabela centralizada não escalaria nem sobreviveria a falhas. Por isso o DNS foi pensado (desde cerca de **1987**) com objetivos claros:

| Objetivo | Motivo |
|----------|--------|
| **Multi-inquilino** | Muitas entidades administram seus próprios domínios **sem interferir** umas nas outras. |
| **Tolerância a falhas** | A internet depende do DNS; falhas devem **afetar só parte** do sistema, não “derrubar tudo”. |
| **Escalabilidade** | A internet cresce; o desenho precisa **absorver mais nomes e mais consultas**. |
| **Bom desempenho** | DNS lento vira **aplicação lenta**; latência importa. |

## Padrões de desenho que viabilizam isso

- **Federação** → suporta o **multi-inquilino** (autoridade dividida por zonas / delegações).
- **Replicação** → melhora **tolerância a falhas** e disponibilidade dos dados.
- **Distribuição** → permite **escalar** consultas e responsabilidades.
- **Cache** → reduz carga e melhora **desempenho** (com o trade-off de dados eventualmente “atrasados” em alguns pontos).

### Consequência para quem depura

Não existe mais **um único lugar** com a “verdade” instantânea para todo mundo: é um **sistema distribuído**. Pontos diferentes podem ver **versões diferentes** do mesmo dado por um tempo (TTL, cache intermediário, etc.) — isso explica muitos sintomas confusos na prática.

## Protocolo vs. “banco de dados”

De perto, o DNS é um **protocolo de rede** (formato de mensagens, tipos de registro, encadeamento de respostas).

De longe, continua útil enxergar como um sistema **chave → valor**: chave ≈ **nome do domínio**, valor ≈ **dados associados** (IPs, aliases, mail servers, texto, etc.). Esse modelo mental ajuda no começo; os detalhes vêm com registros, resolução recursiva e delegação.

## Recursos e referências

- [O que é DNS?](https://www.nslookup.io/learning/what-is-dns/)
- [Registro A (IPv4)](https://www.nslookup.io/learning/dns-record-types/a/)
