# Exemplo ACBrNFe + Reforma Tributária (IBS/CBS/IS) — NFC-e SP (Restaurante / Simples Nacional)

Este repositório demonstra **como emitir NFC-e (modelo 65) no Estado de São Paulo** para um **restaurante optante pelo Simples Nacional**, usando **Delphi 13 (RAD Studio 12)** + **ACBrNFe (última versão)**.

Além do fluxo tradicional de NFC-e, o exemplo mostra **como preencher os novos grupos da Reforma Tributária do Consumo (RTC)** — **IBS/CBS/IS** — **com valores fictícios** para **validar o leiaute** em homologação e orientar sua implementação nos sistemas.

> **Atenção**
>
> * Os campos RTC (IBS/CBS/IS) **podem estar em fase de transição** e **não devem ser usados em produção** sem obedecer às NTs vigentes, legislação (EC 132/2023, LC 214/2025) e parametrizações contábeis do contribuinte.
> * Este projeto é **didático**: valores e CSTs RTC são **exemplos para validação de schema**.

---

## Sumário

* [Pré-requisitos](#pré-requisitos)
* [Como executar](#como-executar)
* [Estrutura do projeto](#estrutura-do-projeto)
* [Configuração necessária](#configuração-necessária)
* [Fluxos disponíveis no app demo](#fluxos-disponíveis-no-app-demo)
* [RTC no ACBr — mapeamento e boas práticas](#rtc-no-acbr--mapeamento-e-boas-práticas)

  * [Cabeçalho (IDE)](#cabeçalho-ide)
  * [Item — IS (Imposto Seletivo)](#item--is-imposto-seletivo)
  * [Item — IBS/CBS (tributação padrão)](#item--ibscbs-tributação-padrão)
  * [Item — IBS/CBS monofásico](#item--ibscbs-monofásico)
  * [Totais (RTC)](#totais-rtc)
* [Como parametrizar os dados](#como-parametrizar-os-dados)
* [Roteiro de testes (homologação) e rejeições comuns](#roteiro-de-testes-homologação-e-rejeições-comuns)
* [Governança, trilha de auditoria e conformidade](#governança-trilha-de-auditoria-e-conformidade)
* [FAQ rápido](#faq-rápido)
* [Referências úteis](#referências-úteis)

---

## Pré-requisitos

* **Delphi 13 / RAD Studio 12.x**
* **ACBrNFe** atualizado (com suporte aos grupos RTC em `pcnNFeRT`)
* **Certificado digital** do emitente (A1 recomendado para testes)
* **CSC e IdCSC** de **homologação** (NFC-e SP)
* Credenciamento ativo para **NFC-e** na **SEFAZ-SP** (ambiente de homologação)

> Dica: mantenha o ACBr sempre atualizado (componentes, schemas e tabelas). Mudanças de NT podem alterar nomes de tags, cardinalidades ou regras de validação.

---

## Como executar

1. **Clone** este repositório e abra o projeto no **Delphi**.
2. Abra o form principal e ajuste a **configuração** (ver [Configuração necessária](#configuração-necessária)).
3. Compile e execute (**F9**).
4. Use os botões do formulário para **emitir NFC-e** nos cenários de exemplo.
5. Acompanhe o **retorno da SEFAZ** no log e verifique os **XMLs** salvos nos diretórios configurados.

---

## Estrutura do projeto

```
refactoring-acbrnfe-tax-reform/
 ├─ README.md                      ← este arquivo
 ├─ source/
 │   ├─ Frm_ACBrNFe.pas            ← form principal, botões e métodos de emissão
 │   └─ ...                        ← demais units/recursos do exemplo ACBr
 └─ ...
```

* **Frm\_ACBrNFe.pas** contém:

  * Método **`EmitirNFCe_SP_Restaurante`** (1 item: refeição — cenário padrão)
  * Método **`EmitirNFCe_SP_Restaurante_DoisItens`** (2 itens: refeição + bebida monofásica)
  * Rotinas de **configuração do ACBr** (UF/Ambiente, CSC/IdCSC, certificado, paths)

---

## Configuração necessária

No método de configuração do ACBr (ex.: `ConfigurarACBr` no form):

```delphi
// Ambiente
ACBrNFe1.Configuracoes.WebServices.UF := 'SP';
ACBrNFe1.Configuracoes.WebServices.Ambiente := taHomologacao; // produção: taProducao

// CSC / IdCSC (HOMOLOGAÇÃO - NFC-e/SP)
ACBrNFe1.Configuracoes.Geral.IdCSC := '000001';              // SUBSTITUA pelo Id do Token
ACBrNFe1.Configuracoes.Geral.CSC   := 'SEU_CSC_HOMOLOGACAO'; // SUBSTITUA pelo CSC

// Certificado (A1)
ACBrNFe1.SSL.TipoCertificado := tcArquivo;
ACBrNFe1.SSL.Caminho := 'C:\\certs\\empresa.pfx';
ACBrNFe1.SSL.Senha   := 'senha-do-certificado';

// Diretórios
ACBrNFe1.Configuracoes.Arquivos.PathSalvar := 'C:\\ACBr\\NFCe\\XML';
ACBrNFe1.Configuracoes.Arquivos.PathSchemas := 'C:\\ACBr\\Schemas';
```

**Importante**

* **CSC/IdCSC** de **homologação** e **produção** são **diferentes**. Não misture.
* Certifique-se de que o **certificado digital** (A1/A3) **está válido** e acessível.
* NFC-e é **síncrona**: o exemplo envia o **lote de 1** e guarda o retorno.

---

## Fluxos disponíveis no app demo

### 1) `EmitirNFCe_SP_Restaurante`

* **Cenário**: restaurante (venda balcão), **Simples Nacional**.
* **Item**: 1× "REFEIÇÃO SELF-SERVICE" (`NCM 21069090`, `CFOP 5102`).
* **ICMS (SN)**: `CSOSN 102` (sem crédito de ICMS na saída da NFC-e).
* **RTC**: preenche **IBS/CBS** no item (CST `000`, `cClassTrib` fictício `000001`) **apenas para validar leiaute**; **IS** com `0,00`.
* **Pagamento**: dinheiro (pode alternar para PIX, cartão crédito/débito etc.).

### 2) `EmitirNFCe_SP_Restaurante_DoisItens`

* **Cenário**: 2 itens — **refeição** + **bebida**.
* **Item 1 (refeição)**: RTC **padrão** (mesmos parâmetros do fluxo 1).
* **Item 2 (bebida)**: RTC **monofásico** (`CST 620`) usando grupo `gIBSCBSMono`.
* **Totais**: soma dos itens + grupos RTC.

> Use estes dois cenários para aprender **qual grupo RTC** preencher em cada caso e como os **totais** devem ser **consistentes** com a soma dos itens.

---

## RTC no ACBr — mapeamento e boas práticas

### Cabeçalho (IDE)

Campos usados no exemplo:

```delphi
Ide.cMunFGIBS := 3550308; // Município (SP capital) para efeitos de IBS/CBS
Ide.tpNFDebito  := tdNenhum;  // Nota de Débito (ajustes) — não usado na venda padrão
Ide.tpNFCredito := tcNenhum;  // Nota de Crédito (ajustes) — não usado na venda padrão

// Exemplo didático de compra governamental (não aplicável ao restaurante comum)
Ide.gCompraGov.tpEnteGov := tcgEstados;
Ide.gCompraGov.pRedutor  := 0;
Ide.gCompraGov.tpOperGov := togFornecimento;
```

> **Quando preencher `cMunFGIBS`?** Em regra, use conforme a NT e as orientações do Portal NF-e. No exemplo, é usado para **validar o leiaute RTC** e dar visibilidade do campo. Em produção, avalie as situações (operações presenciais fora do estabelecimento, sem endereço de entrega/destinatário etc.).

---

### Item — IS (Imposto Seletivo)

```delphi
with Imposto.ISel do
begin
  CSTIS := cstis000;  // situação do IS (consulte tabela/NT)
  // cClassTribIS: preencha quando a tabela exigir
  vBCIS := 25.00;     // base (exemplo)
  pIS   := 0.00;      // alíquota (restaurantes tipicamente N/A)
  vIS   := 0.00;      // valor do IS
end;
```

* **IS é “por fora”**: quando devido, influencia o **total** da NF-e (veja [Totais (RTC)](#totais-rtc)).
* No restaurante comum, **geralmente não se aplica**; mantivemos **zerado** para teste de leiaute.

---

### Item — IBS/CBS (tributação padrão)

```delphi
with Imposto.IBSCBS do
begin
  CST := cst000;               // exemplo: tributação integral
  cClassTrib := '000001';      // classificação tributária (tabela oficial)

  // Grupo padrão (operações não monofásicas)
  gIBSCBS.vBC  := 25.00;       // base IBS/CBS (exemplo)
  gIBSCBS.vIBS := 1.25;        // valor IBS (exemplo)

  // CBS (subgrupo dentro de gIBSCBS)
  gIBSCBS.gCBS.pCBS := 2.00;   // % (exemplo)
  gIBSCBS.gCBS.vCBS := 0.50;   // valor CBS (exemplo)
end;
```

Outros subgrupos podem ser exigidos dependendo do **CST/cClassTrib** (ex.: `gIBSUF`, `gIBSMun`, `gDif`, `gDevTrib`, `gRed`, `gTribRegular`). **Somente preencha quando a tabela exigir**.

---

### Item — IBS/CBS monofásico

Para **CST 620 (monofásico)**, utilize o **grupo monofásico** no lugar do padrão:

```delphi
with Imposto.IBSCBS do
begin
  CST := cst620;          // monofásico
  cClassTrib := '000002'; // classificação tributária (exemplo)

  with gIBSCBSMono.gMonoPadrao do
  begin
    qBCMono   := 1;     // quantidade base (ad rem)
    adRemIBS  := 0.20;  // valor ad rem IBS por unidade (exemplo)
    adRemCBS  := 0.10;  // valor ad rem CBS por unidade (exemplo)
    vIBSMono  := 0.20;  // valor IBS monofásico (exemplo)
    vCBSMono  := 0.10;  // valor CBS monofásico (exemplo)
  end;
end;
```

> **Regra de ouro**: o **CST que você escolhe direciona o grupo** que deve ser preenchido. **Não** use `gIBSCBS` (padrão) quando o **CST** exigir `gIBSCBSMono` (monofásico), e vice-versa.

---

### Totais (RTC)

No totalizador, consolide os valores **por grupo**:

```delphi
// Totais clássicos
Total.ICMSTot.vProd := 30.00;
Total.ICMSTot.vNF   := 30.00; // atenção: RTC/IS “por fora” podem alterar o total efetivo

// Totais RTC
Total.IBSCBSTot.vBCIBSCBS := 30.00;
Total.IBSCBSTot.gIBS.vIBS := 1.45;
Total.IBSCBSTot.gCBS.vCBS := 0.60;

// IS total (se houver)
Total.ISTot.vIS := 0.00;

// Algumas versões de leiaute RTC incluem totalizador agregado da nota (ex.: vNFTot)
// conforme a NT vigente. Ajuste quando aplicável.
```

**Boas práticas em totais**

* **Some por item** com exatidão (IBS, CBS e IS) e leve ao totalizador correspondente.
* Mantenha **consistência** entre **itens** e **totais** (rejeições comuns são diferenças de centavos/rounding).

---

## Como parametrizar os dados

**Por produto/serviço** (tabelas de cadastro):

* **CST IBS/CBS** (ex.: `000`, `620` etc.)
* **`cClassTrib`** (código da Classificação Tributária compatível com o CST)
* **Regras do grupo** a preencher: `gIBSCBS` (padrão) **ou** `gIBSCBSMono` (monofásico)
* **NCM**, **CFOP**, **unidade**, **origem**

**Por operação/UF**:

* **`cMunFGIBS`** (município para efeitos do IBS/CBS quando a NT exigir)
* **Compra governamental** (somente quando aplicável)

**Por regime**:

* **CRT** (Simples Nacional, Normal)
* **ICMS** (no SN, NFC-e costuma ir com **CSOSN 102**)

> **Parametrização é chave**: crie uma **estrutura de regras** (tabelas paramétricas) que mapeie **CST/cClassTrib → grupos** e **campos obrigatórios**, evitando preencher campos indevidos.

---

## Roteiro de testes (homologação) e rejeições comuns

**Testes básicos**

1. **1 item (refeição)**: `CSOSN 102`, RTC padrão. Esperado: **Autorizada (cStat 100)**.
2. **2 itens (refeição + bebida)**: item 1 padrão, item 2 monofásico (`CST 620`). Esperado: **Autorizada**, com grupo **Mono** no item 2.
3. **Formas de pagamento**: mude `tPag` para `fpPix`, `fpCartaoCredito`, `fpCartaoDebito`.

**Rejeições comuns e correções**

* **QR-Code/CSC**: `cStat 464` (hash do QR-Code difere) → revisar **IdCSC/CSC** e **Ambiente** (homologação x produção).
* **Grupo RTC incorreto**: `CST 620` com `gIBSCBS` em vez de `gIBSCBSMono` → ajustar o grupo conforme o **CST**.
* **Schemas/tabelas desatualizados**: erro de schema → **atualize ACBr + schemas + tabelas**.
* **Totais inconsistentes**: soma de itens ≠ totalizadores → revisar **rounding** e agregações.

> Dica: habilite logs detalhados e salve **XML de envio** e **procNFe** (autorização) para análise.

---

## Governança, trilha de auditoria e conformidade

* **Diretórios por ambiente/UF** para XMLs (organização e backups).
* **Logs de envio/retorno** com data/hora, `cStat`, `xMotivo`, `chNFe`, `nProt`.
* **Versionamento** de regras (CST/cClassTrib) e **auditoria** de alterações.
* **Plano de contingência** (consulta, reenvio, contingência quando aplicável pela UF).
* **Monitoramento** de **NTs** e **releases ACBr** (mudanças frequentes durante a transição RTC).

---

## FAQ rápido

**Posso deixar os campos RTC sempre preenchidos?**

> Não. Preencha **somente** quando aplicável e conforme a **tabela CST/cClassTrib**. Em transição, use para **homologar/validar leiaute**.

**Simples Nacional destaca IBS/CBS na NFC-e?**

> Siga a legislação/NTs vigentes. O exemplo usa RTC **didaticamente** (validação). A regra efetiva pode variar por fase.

**Como sei quais subgrupos RTC preencher?**

> A **tabela oficial** (CST ↔ subgrupos) e as **NTs** indicam exatamente o que informar (ex.: padrão vs monofásico, diferimento, devolução).

---

## Referências úteis

* **EC 132/2023 (Reforma Tributária)** — texto oficial: `https://www.planalto.gov.br/ccivil_03/constituicao/emendas/emc/emc132.htm`
* **LC 214/2025 (IBS, CBS e IS)** — texto oficial: `https://www.planalto.gov.br/ccivil_03/leis/lcp/lcp214.htm`
* **Portal Nacional da NF-e (RTC / Notas Técnicas / Tabelas)**: `https://www.nfe.fazenda.gov.br/portal/principal.aspx`
* **Fórum ACBr — Como preencher RTC no ACBrNFe**: `https://www.projetoacbr.com.br/forum/topic/84094-como-preencher-as-informações-relacionadas-a-reforma-tributária-no-componente-acbrnfe/`
* **Portal NFC-e SP (CSC)** — área do contribuinte para gerenciar **IdCSC/CSC** (acesso com certificado)

---

### Nota final

Este exemplo **não substitui** a análise contábil/fiscal. Antes de produzir, alinhe **CNAE**, **regras setoriais**, **CST/cClassTrib**, **UF/município** e **cronograma RTC** com o **contador** e as **publicações oficiais**. Parametrizações incorretas podem gerar **rejeições**, **glosas de crédito** e **riscos de conformidade**.
# refactoring-acbrnfe-tax-reform