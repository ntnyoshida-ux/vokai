# CLAUDE.md — Vokai

> Este arquivo é lido automaticamente pelo Claude Code a cada sessão.
> Contém todo o contexto estratégico, técnico e de produto da Vokai.

---

## 🏢 Sobre a Vokai

**Empresa:** Vokai — plataforma SaaS multi-tenant de gestão para pequenos negócios de serviço
**Domínio:** vokaisolutions.com.br
**Estágio:** early-stage, em desenvolvimento ativo

**Produto:** plataforma que centraliza agendamentos, controle financeiro e relacionamento com clientes para pequenos negócios de beleza e estética de bairro — sem necessidade de conhecimento técnico.

---

## 👥 ICP (Ideal Customer Profile)

**Primário:** profissional autônomo ou micro-empreendedor do setor de beleza/estética de bairro
- Barbearias (1–3 cadeiras), esmalterias, designers de sobrancelha, manicures
- Fatura R$5k–R$20k/mês
- Agenda pelo WhatsApp ou caderninho
- Zero ferramenta financeira
- Toma decisão sozinho, por confiança e indicação
- Não se vê como "empreendedor digital"

**Secundário (futuro):** salões de beleza com 2–5 profissionais

### Dores principais
1. **Agendamento caótico** — WhatsApp mistura pessoal e profissional, cliente some, horário vazio sem aviso
2. **Inadimplência invisível** — não sabe quem deve, quanto e há quanto tempo
3. **Sem visão financeira** — só sabe se o mês foi bom quando sobra (ou falta) dinheiro no bolso

---

## 🛠 Stack técnica

| Camada | Tecnologia |
|--------|-----------|
| Frontend | React |
| Backend / DB | Supabase (PostgreSQL + Auth + RLS) |
| Dev platform | Lovable |
| Pagamentos | Stripe |
| E-mail transacional | Resend (domínio: vokaisolutions.com.br) |

---

## 🏗 Arquitetura — Multi-tenant

- **Identificador único global:** e-mail (`auth.users.email`)
- **Isolamento:** por `company_id` com RLS ativo em TODAS as tabelas
- **Papéis por empresa** (tabela `company_users`):

| Papel | Acesso |
|-------|--------|
| `owner` | Administrativo pleno |
| `admin` | Operacional amplo |
| `staff` | Operacional restrito (sem dashboard financeiro) |
| `provider` | Apenas próprios agendamentos e horários |
| `client` | Apenas próprios compromissos e assinaturas |

---

## 💳 Monetização

### Receitas
1. **Assinatura recorrente** — plano mensal com período de trial
   - Trial longo = mais dados acumulados = maior switching cost = menor churn
2. **Add-on Agente IA** — upsell sobre o plano base
3. **Take rate por transação** — margem Vokai sobre pagamentos processados via Stripe

### Cálculo de take rate por transação
```
Custo Stripe Brasil: 3,99% + R$0,39 por transação
Vokai cobra: margem própria (% configurável) em cima do custo Stripe
Líquido Vokai = margem cobrada − custo Stripe
```
⚠️ Em tickets baixos (R$30–60), o custo fixo do Stripe pode consumir a margem. Considerar margem maior para tickets menores.

---

## 🔑 Fluxos críticos

### 1. Cadastro de Owner
- Cadastro → criação da empresa → vínculo automático owner/empresa na mesma operação
- E-mail único na plataforma (sem duplicidade entre empresas)

### 2. Fluxo de convite (Provider / Admin / Staff)
- Owner gera link com token único da empresa
- Token/`company_id` capturado da URL ANTES do cadastro
- Após `auth.signUp`, vínculo usuário-empresa inserido com papel correto
- Fluxo de confirmação de e-mail deve preservar o token do convite

### 3. Cadastro de cliente
- **BUG CONHECIDO:** usuário cria login mas NÃO fica vinculado à empresa
- Fix: capturar `company_id` do link antes do signUp; após signUp, inserir relacionamento cliente-empresa sem race condition

### 4. Agendamento sem login
- `company_id` sempre mantido no contexto durante o fluxo
- Dados isolados por empresa mesmo sem autenticação

---

## 📧 Infraestrutura de e-mail

- **Provedor:** Resend (migrado de Zoho/Zeptomail)
- **DNS:** DKIM Resend configurado (`resend._domainkey`)
- **Inbox principal:** Zoho Mail (MX records mantidos)
- **Registros removíveis:** Zeptomail/SES (bounce-zem CNAME, send subdomain MX/SPF)

---

## 📐 Convenções de desenvolvimento

- Sempre usar RLS — nenhuma tabela sem policy por `company_id`
- Não duplicar e-mail entre empresas — e-mail é identificador global
- Ao criar vínculo usuário-empresa: fazer após `auth.signUp` confirmar, sem race condition
- Stripe: considerar custo fixo por transação no cálculo de margens
- Testes com Vitest; hooks de commit via Husky

---

## 🗺 Contexto de produto (PMM)

**Posicionamento:** "a plataforma que organiza o seu negócio para você crescer sem depender de caderninho ou WhatsApp"

**Voz da marca:** direta, próxima, sem jargão técnico — fala com quem nunca usou software de gestão

**Proof points (a validar com beta):**
- Redução de no-shows
- Tempo recuperado em agendamento
- Inadimplência identificada no primeiro mês
- NPS beta

**Concorrentes diretos do ICP:** WhatsApp puro, caderninho

---

## 👩‍💼 Time

- **Karina** — PMM, GTM, produto
- **Nathan** — co-fundador, dev
- Domain expert: dono de barbearia (contribuiu com contexto do ICP)

---

*Atualizado via claude.ai — Mai/2026*
