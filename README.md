# Skannio

App web de conferência e expedição para lojistas de e-commerce. O pedido é conferido por bipagem em duas etapas (escritório imprime, galpão confere), agrupado em gaiolas e fechado em um romaneio por coleta.

Cores: branco, preto e lime `#CCFF00`. Fontes: Atkinson Hyperlegible Next e Mono (Google Fonts).

> **Estado:** protótipo funcional que roda inteiro no navegador, em um único arquivo (`expedicao.html`). Tudo fica salvo no aparelho. Não há servidor, e por isso não há segurança real nem integração por API ainda (veja *Limites*).

---

## O que o app faz

| Área | O que tem |
|---|---|
| **Bipagem** | Etapa 1 (escritório): bipa a etiqueta ao imprimir. Etapa 2 (galpão): bipa o pacote fechado e o app diz a gaiola. Resposta com som, cor e mascote. Detecta o canal pelo código (Shopee, Mercado Livre, Amazon, Correios). |
| **Gaiolas** | Progresso por coleta e por canal. |
| **Pedidos** | Importação por planilha CSV/Excel colada ou carregada. Busca e filtro por situação. |
| **Etiquetas** | Três abas, descritas abaixo. |
| **Fechamento** | Resumo do dia, alerta de pedidos não conferidos, romaneio por coleta para copiar e arquivar. |
| **Histórico** | Dias fechados, busca por pedido ou código, romaneio de cada dia. |
| **Devoluções** | Recebimento por bipe, motivo, condição e destino, até 4 fotos por devolução, fila e análise. |
| **Estoque** | Entrada por bipe, produtos, saldo. |
| **Vendas** | Faturamento por dia e por canal, venda de balcão por bipe. |
| **Integrações** | Lista de conexões planejadas, com marcação de interesse. |
| **Usuários** | Perfis Administrador, Escritório e Galpão, cada um vendo só o que precisa. |
| **Painel de TV** | Tela cheia para o galpão com progresso por gaiola e recados. |

### Etiquetas

- **Etiquetas de envio** (oficiais do marketplace):
  1. Importe o PDF (uma etiqueta por página) ou o ZPL/TXT (uma por bloco `^XA…^XZ`) baixado do marketplace ou do ERP.
  2. O app lê o código de cada etiqueta e casa com o pedido.
  3. Imprime em lote, na ordem das gaiolas: PDF pela janela de impressão (10x15 cm, A4 com 4 por folha, A4 com 1 por folha); ZPL como arquivo `.txt`.
  4. Pedido já impresso não entra de novo. **Reimprimir** fica registrado (quem, quando, quantas vezes).
  5. Pedido sem etiqueta não trava o lote. Aparece na lista de pendências com as fontes tentadas.
  6. Código repetido em mais de uma etiqueta: usa a primeira e avisa.
- **Etiqueta interna:** etiqueta 10x15 ou A4 com código de barras Code 128, canal e gaiola. Não substitui a etiqueta de envio.
- **Fontes:** ordem de prioridade das fontes de etiqueta e o que cada uma exige.

**Conferência no galpão:** ao bipar o pacote na etapa 2, o app confirma se a etiqueta de envio daquele pedido foi impressa pelo Skannio. No Fechamento, aparece a lista dos pedidos com etiqueta impressa que ainda não foram conferidos.

---

## Como usar

1. Abra o app. No primeiro acesso, crie o administrador (nome, usuário e senha com no mínimo 6 caracteres).
2. **Pedidos → Importar:** cole a planilha do dia. Colunas esperadas: pedido, código de rastreio, produto e coleta/gaiola. O app guarda o canal detectado pelo código.
3. **Etiquetas → Etiquetas de envio:** importe os arquivos do marketplace e imprima o lote.
4. **Bipagem:** o escritório bipa na impressão; o galpão bipa o pacote fechado.
5. **Fechamento:** confira os alertas, feche o dia e copie o romaneio para o motorista assinar.
6. Para testar sem dados reais, use os dados de exemplo e o botão **Usar etiquetas de exemplo**. Depois, **Apagar e começar do zero**.

### Perfis

| Perfil | Acesso |
|---|---|
| Administrador | Tudo, inclusive usuários, integrações e troca de etapa da bipagem. |
| Escritório | Bipagem (etapa 1), pedidos, etiquetas, vendas, conta. |
| Galpão | Bipagem (etapa 2), gaiolas, devoluções, estoque, conta. |

---

## Limites desta versão

- **Segurança:** o login (senha com PBKDF2, bloqueio após 5 falhas) roda no navegador. Serve para organizar quem vê o quê, **não** protege dados contra quem tem acesso ao aparelho. Segurança real exige servidor.
- **Dados locais:** pedidos, estoque, devoluções e usuários ficam no `localStorage`; fotos e etiquetas importadas ficam no IndexedDB (`bipou-fotos`, nome mantido de uma versão anterior). Limpar os dados do navegador apaga tudo. Não há sincronização entre aparelhos nem backup.
- **API do Bling, Tiny, Mercado Livre e Shopee:** precisam de servidor (a chave OAuth não pode ficar no navegador) e, no caso dos marketplaces, de credenciamento. Estão na tela *Fontes* como futuras. Falta validar se o Bling devolve etiqueta de Mercado Livre e Shopee ou só de transportadoras.
- **ZPL direto na impressora de rede (porta 9100) e impressora USB:** o navegador não abre conexão com a impressora. Exigem app de computador.
- **PDF na térmica:** imprime pelo driver do sistema; falta conferir a nitidez na sua térmica 10x15.
- **Leitura de código:** vem do texto da etiqueta. PDF só de imagem depende do leitor de código de barras do navegador, quando existe (não testado). Os padrões de código por canal são heurísticas e precisam ser validados com etiquetas reais.
- **Cancelar a impressão:** o navegador não avisa. O pedido fica como impresso e o remédio é Reimprimir.
- **LGPD:** etiquetas têm dados do comprador. Ficam só no aparelho e nada é enviado para fora. O leitor de PDF (pdf.js 3.11.174) é carregado do cdnjs na primeira importação de PDF.

---

## Estrutura técnica

Arquivo único `expedicao.html`: HTML, CSS e JavaScript puro (estilo ES5, sem build).

- **Estado:** objeto `S` (pedidos, produtos, vendas, devoluções, movimentos, bipagens, histórico, etapa, som) salvo em `expedicao.web.v1`. Usuários em `expedicao.users.v1`; sessão em `sessionStorage` (`expedicao.sessao.v1`).
- **UI:** objeto `ui`, função `render()` com uma view por aba, ganchos `AFTER`, eventos por delegação (`data-act`, `data-m`).
- **Pedido:** `id, canal, produto, codigo, coleta, etapa (0/1/2), t1, t2`, mais `etq/etqn` (etiqueta interna) e `eo/ec` (etiqueta de envio impressa e conferida).
- **Detecção de canal:** Shopee `^BR\d{12}`, Amazon `^TBA\d+`, Correios `^[A-Z]{2}\d{9}BR$`, Mercado Livre `^\d{11}$`.
- **Fontes de etiqueta:** lista `ESRC`; `eoResolve(pedido)` tenta cada fonte ativa em ordem e isola erros. Somar uma fonte é adicionar um item com `get(pedido)`.
- **Fila de impressão:** estados por pedido `pronta`, `impressa`, `falhou`, `sem etiqueta`.
- **Código de barras:** gerador Code 128 (conjunto B) próprio, validado por leitura com zxing-cpp.
- **Impressão:** `#printArea` + `@media print` + `@page`; fallback de download pelo recurso `downloads` do artefato.
- **Design:** tokens em `:root`, modo escuro automático, metáfora de etiqueta de envio, lime só como marca-texto, mascote astronauta em SVG.

### Testes feitos

Verificação de sintaxe (`node --check`), Playwright em Chromium headless (fluxo completo, login, etiquetas, impressão em PDF, download, celular 390 px), leitura dos códigos de barras gerados e importação de PDF/ZPL de teste. Nenhum teste usou etiqueta real de marketplace, impressora física ou API real.

---

## Próximos passos

1. Servidor com login real, banco de dados e sincronização entre aparelhos.
2. Conta de desenvolvedor no Bling: testar se a API devolve a etiqueta de envio de Mercado Livre e Shopee.
3. Rodar a importação sobre 50 a 100 etiquetas reais de cada canal e ajustar os padrões de código.
4. App de computador para ZPL na impressora de rede, impressora USB e PDF na térmica.
5. Iniciar o credenciamento nas APIs de Mercado Livre e Shopee.
6. Busca de marca no INPI (classes 9, 35 e 42) e registro de domínio antes de investir no nome Skannio. Nomes parecidos já existem (Skanio, Scannio, Scanio).
7. Acordo por escrito sobre propriedade do código, dado o vínculo CLT com a D Coração Home. O app foi escrito do zero, sem usar código nem contas dela.
