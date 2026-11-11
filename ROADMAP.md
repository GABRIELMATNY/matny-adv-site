# MATNY ADV — Estado e Roteiro de continuidade

> Handoff para retomar o desenvolvimento em qualquer sessão. Leia este arquivo primeiro.

Última atualização: 2026-11-11.

## Como retomar
1. Abrir o repositório `GABRIELMATNY/matny-adv-site` (add_repo + clone) e ler este arquivo.
2. Editar `index.html` e dar `git push origin main` — o GitHub Pages publica sozinho em 1–2 min.
3. Validar sintaxe e testar com navegador headless (ver "Como testar") ANTES de cada push.

## Arquitetura
- **Frontend:** arquivo único `index.html` (vanilla JS). Estado global `S`; UI por `render()` + objeto `VIEWS` (uma função por tela) + objeto `ACTS` (ações). Eventos por delegação: `data-go="rota"` navega, `data-act="nome"` chama `ACTS.nome`.
- **Publicação:** GitHub Pages → https://gabrielmatny.github.io/matny-adv-site/
- **Backend:** Supabase projeto `tidpsljigqudrivcefla`. Edge Function `api` (supabase-js **2.112.3**). Login e-mail/senha; token em `localStorage.matny_token`.

## Notas técnicas (não redescobrir)
- A service key é formato novo `sb_secret_` → exige supabase-js ≥ 2.11x (está em 2.112.3). Versão antiga (2.45) quebrava login.
- CORS da função `api` precisa aceitar os headers `authorization, content-type, apikey, x-client-info`.
- GRANTs: `service_role` tem SELECT/INSERT/UPDATE/DELETE em todas as tabelas de `public` (+ default privileges). `anon`/`authenticated` sem acesso direto (RLS ligado, sem policies).
- Site URL e Redirect URLs do Supabase apontam para o GitHub Pages.
- Campo de senha gov.br é guardado SEM criptografia — não usar com dados reais até criptografar.

## Como testar (headless, antes de publicar)
- Sintaxe: extrair os `<script>` sem src e rodar `new Function(code)` no Node.
- Render: Playwright (chromium pré-instalado) abre `file://.../index.html`, esconde `#login`, injeta dados em `S`, seta `ROTA` e chama `render()`, capturando `pageerror`. Processos de teste precisam de `ultimaMov` (senão `procParado` quebra).

## FEITO (no ar)
- Login funcional; reset de senha por e-mail.
- Topo: menu do usuário (Meus dados / Configurações / Sair), sininho com notificações (publicações não lidas), nome/avatar do usuário logado.
- Ficha do cliente (dados, processos, agenda, financeiro); nome do cliente clicável em qualquer tela.
- Botões "Copiar" em nome, CPF, telefone, e-mail, senha gov.br e número do processo.
- Máscaras CPF/CNPJ/telefone; CEP preenche endereço (ViaCEP).
- Atalhos: Esc fecha modal, Enter salva.
- Painel de gestão com gráficos: donut de processos por setor e clientes por status; barras de funil (processos por etapa) e produtividade da equipe; resumo financeiro.

## PENDENTE (priorizado)

### Seguro para sessão autônoma (só frontend em index.html; testar + push)
- [ ] Menu "⋮" por item nas listas: editar, duplicar, excluir/arquivar (com confirmação).
- [ ] Ações em massa (selecionar vários → etiqueta, mudar etapa, exportar).
- [ ] Checklists por tipo de benefício (BPC/LOAS, aposentadoria por idade, por incapacidade, pensão) na ficha do processo — guia de documentos/passos.
- [ ] Mais atalhos de navegação (ex.: G depois P = Processos) e dica visual deles.
- [ ] Estados vazios instrutivos em todas as telas; confirmação antes de excluir.
- [ ] Exportar listas (CSV) de clientes/processos.

### Requer o usuário presente (mexe em banco/estrutura/integrações — NÃO fazer autônomo)
- [ ] **Multi-tenant** (escolha do cliente: primeiro): tabela `escritorios` + `escritorio_id` nas tabelas; função filtra por escritório do usuário.
- [ ] **Persistência na nuvem**: hoje clientes/processos/tarefas/agenda são locais (em memória) — salvar no Supabase. CRÍTICO para uso real.
- [ ] Importar dados do ADVBOX (planilha).
- [ ] Captura automática do DJEN por OAB (tarefa agendada no servidor).
- [ ] Motor de prazos (dias úteis, feriados forenses).
- [ ] Modelos de peças com preenchimento automático (merge).
- [ ] Financeiro: honorários (contratual/êxito), parcelamento, cobrança PIX/boleto (Asaas), relatórios.
- [ ] WhatsApp real, portal do cliente, assinatura eletrônica, gamificação de produtividade.

### Segurança / comercialização
- [ ] Tornar o repositório privado (hoje é público).
- [ ] Criptografar a senha gov.br.
- [ ] LGPD: consentimento, controle de acesso ao CID, política.

## Decisões do cliente
- Opção 1: multi-tenant (isolamento por escritório) antes de novas features.
- Identidade MATNY própria; **não** copiar o ADVBOX; **não** usar fotos de pessoas reais.
- Todo botão precisa ter ação; clicar em processo/cliente abre o registro.
