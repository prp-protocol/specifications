# Registro DNS PRP — revisão 10 para leitura do autor

Pacote local atualizado para a referência explícita completa. Não enviado à
IANA; sem RRTYPE atribuído e sem alegação de implantação/interoperabilidade.

## Formato integrado

```
cabeçalho: 1 octeto | suite: 2 octetos big-endian | identificador: L octetos
```

O cabeçalho contém dois bits de classe e seis reservados zerados. O mapa é
IDENTITY=00, ALIAS=01, MULTICAST=10, GROUPCAST=11, resultando nos octetos
0x00..0x03. Cada RDATA publica exatamente uma dessas referências. Um RRset pode
conter várias referências distintas, sem ordem, preferência ou fallback
implícitos. Nenhuma classe pode ser reinterpretada como outra. A suite é de
referência, não a suite da sessão. Não há campo extra de versão ou comprimento.

Nas suites1/2/3/5, L=32 e RDATA tem35 bytes; na suite4, L=48 e RDATA tem51.
A forma textual proposta tem três termos: classe decimal0..3, suite decimal e
identificador hexadecimal. Exemplo GROUPCAST sintético, sem prova de criação ou
administração:

```
group.example. 300 IN PRP 3 5 000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f
```

O formato binário é o mesmo envelope da referência independente. Isso não amplia
automaticamente o perfil URI/QR atual, que ainda é IDENTITY-only. O cabeçalho
não altera a formação criptográfica do Strong ID.

## O que mudou e o que permanece

- A revisão6, com34/50 bytes e dois termos textuais, está superada e preservada
  no histórico Git. Seu checker foi mantido em
  `tests/historical/check-dns-identity-rr-rev6.py`. Não há fallback antigo/novo.
- A associação passa a ser nome DNS -> conjunto não ordenado de referências,
  não máquina, endereço IP, localização, rota ou consulta reversa. DNS é opcional.
- Cada RDATA contém uma referência. Duplicatas idênticas não acrescentam
  referências; ordem DNS não seleciona nem prioriza nenhuma delas.
- Representação, associação a ALIAS, controle de MULTICAST, genesis,
  administração e participação em GROUPCAST nunca são concedidos pelo DNS.
- Vínculo DNS-autoritativo exige validação confiável, inclusive redirecionamentos;
  HS2 sozinho não prova o vínculo com o nome. Pins independentes não são ampliados.
- Alterações do RRset não provam rotação, migração, delegação ou equivalência.
  TTL não revoga sessões nem garante a publicação mais recente; cache, replay e
  serve-stale continuam com as ressalvas da especificação.
- O suporte DNS das quatro classes não significa admissão final de seus
  protocolos, codecs de autoridade ou wire operacional.
- Para o DNS, os identificadores são octetos opacos com tamanho determinado
  pela suite. Formação, autoridade e ciclo de vida pertencem às especificações
  PRP de cada classe e podem amadurecer sem mudar este RRTYPE enquanto o envelope
  comum permanecer estável.

## Ordem de leitura

1. Este resumo.
2. [Especificação completa, revisão10](../specifications/prp-dns-reference-rr-v1.md).
3. [PRP Self-Certifying Identifier Reference (SCIR)](../specifications/prp-self-certifying-identity-reference-v1.md).
4. [Formulário preparatório A-J](PRP-RRTYPE-APPLICATION.txt).
5. [Referência canônica C1.2](../docs/reference-canonical-layout-proposal.md).
6. [Direitos](../DOCUMENT-RIGHTS.md): CC BY 4.0 para o material DNS de autoria própria.

A formação de identidade/suites referencia wire-v4 seção9 e o registro em
`926bd194b19a690f07a149bb83b085fa7011288d`; o cabeçalho é da referência
sucessora, não uma reinterpretação de wire-v4. Antes do envio, o pacote deve
citar também o OID exato do formato canônico coordenado e revisado.

## Evidência delimitada

O checker atual `tests/check-dns-reference-rr-v1.py` cobre as vinte combinações
de quatro classes e cinco suites em `vectors/dns-reference-v1.json`: codec,
apresentação canônica, bits reservados, tamanhos, RRsets múltiplos sem ordem,
rejeição de fallback e igualdade da referência completa. Os modelos de confiança,
cache e autoridade por classe continuam simbólicos: não verificam DNSSEC,
aquisição de tempo ou protocolos PRP reais, nem usam resolvedor/BIND.

Resultados realmente executados e OIDs de entrega serão informados no retorno
técnico desta revisão. Não herdar o antigo “21 checkers PASS” como evidência
da revisão9. O checksum dos novos vetores identifica framing, não assinaturas.

## Pendências antes do envio

1. Revisão coordenada da referência/registro e compatibilidade pelo contracts,
   seguida de revisão independente spec; esse encaminhamento já existe.
2. Sua revisão final desta apresentação, exemplos, formulário e envio.
3. Referência estável do envelope comum, direitos de materiais externos e
   revisão DNS/segurança. A licença DNS não licencia automaticamente o restante
   do repo; especificações semânticas das classes não são parte do codec DNS.
4. Preencher data de envio apenas quando autorizado e usar a atribuição IANA
   real; nenhum número privado ou placeholder de TYPE é uma atribuição.
5. Fixar as dependências de representação reivindicadas, sem afirmar admissão
   de HS2 pela mera publicação DNS.

As referências DNS/IANA existentes foram verificadas historicamente em
08/09/2026 e não reconsultadas nesta atualização de bytes. Regras de processo
não foram alteradas; este pacote não promete aprovação ou envio.
