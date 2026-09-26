<div align="center">

# Açaí da Patrícia — Comanda Digital

**Pedido pelo QR code da mesa, cozinha em tempo real, fechamento no caixa**

![Next.js](https://img.shields.io/badge/Next.js_14-000000?logo=nextdotjs&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-Postgres_+_Realtime-3ECF8E?logo=supabase&logoColor=white)
![Tailwind](https://img.shields.io/badge/Tailwind-06B6D4?logo=tailwindcss&logoColor=white)
![Vercel](https://img.shields.io/badge/Vercel-000000?logo=vercel&logoColor=white)

[**Abrir demo**](https://a-a-da-patr-cia.vercel.app)

</div>

---

Sistema de comanda presencial feito para uma açaiteria real. Cada mesa tem um cartão com QR code:
o cliente escaneia, monta o açaí no celular e envia o pedido, que **aparece na hora no balcão** com aviso sonoro.
A conta da comanda é atualizada em tempo real e o pagamento é feito no caixa (Pix ou maquininha).

## Três telas, três perfis

| Rota | Quem usa | O que faz |
|---|---|---|
| `/c/<token>` | Cliente (sem login) | Cardápio, montador de açaí guiado por etapas, envio do pedido, conta ao vivo |
| `/balcao` | Balcão / cozinha | Pedidos chegando em tempo real com som, mudança de status, fechamento da comanda |
| `/admin` | Dona da loja | Liga/desliga produtos, marca esgotado, altera preços, gera e imprime os QR das comandas |

## Arquitetura

```
Celular do cliente ──(QR: /c/<token>)──► Next.js (Vercel)
                                              │
                                              ▼
                              Supabase ── Postgres + Row Level Security
                                              │   (Realtime / websockets)
                                              ▼
                                   Tela do balcão atualiza sozinha
```

- **Sem backend próprio para manter**: regras de acesso ficam no banco via **Row Level Security** —
  cardápio público para leitura, escrita só para a equipe autenticada, cliente anônimo só cria pedido na própria comanda
- **Token por comanda** no QR, em vez de número de mesa adivinhável
- **Modelo de opções flexível** (`option_groups` / `options`) para montar o açaí com tamanho, complementos e adicionais
- Custo de infraestrutura zero no plano gratuito de Vercel + Supabase

## Passo a passo pra colocar no ar

### 1) Banco (Supabase)
1. Crie um projeto grátis em https://supabase.com
2. Vá em **SQL Editor** → rode `supabase/schema.sql` (tabelas + tempo real + cardápio de exemplo + 15 comandas).
3. Em seguida rode `supabase/security.sql` (ativa o RLS: leitura do cardápio pública, edição só para a equipe logada).
4. Em **Authentication → Users**, crie um usuário para a Patrícia e um para o balcão (e-mail + senha).
5. Em **Project Settings → API**, copie a `Project URL` e a `anon public key`.

### 2) Variáveis de ambiente
Copie `.env.local.example` para `.env.local` e preencha:
```
NEXT_PUBLIC_SUPABASE_URL=...
NEXT_PUBLIC_SUPABASE_ANON_KEY=...
```

### 3) Rodar local
```
npm install
npm run dev
```
Abra http://localhost:3000 . Para testar o cliente, pegue um token de comanda:
no Supabase, tabela `comandas`, copie um `token` e acesse `/c/<token>`.
Abra `/balcao` noutra aba e veja o pedido cair em tempo real.

### 4) Publicar (Vercel)
1. Suba o projeto num repositório (GitHub).
2. Em https://vercel.com importe o repo, adicione as duas variáveis de ambiente e faça deploy.
3. Pronto: o app fica num endereço tipo `acaidapatricia.vercel.app` (dá pra ligar um domínio próprio depois).

### 5) Imprimir as comandas
Abra `/admin` → aba **Comandas & QR** → imprima a página. Cada QR já aponta pra `/c/<token>` da comanda certa.
Plastifique os cartões (igual ao modelo do Gelato) e distribua nas mesas.

## Segurança (já incluída)
- **Login da equipe**: `/balcao` e `/admin` exigem login (Supabase Auth). Crie os usuários em Authentication → Users.
- **RLS** (`supabase/security.sql`): cardápio com leitura pública; edição do cardápio, mudança de status e fechamento de comanda só para equipe logada; cliente anônimo só consegue criar o próprio pedido via QR.
- Endurecimento opcional (blindar leitura de pedidos por token via RPC) está anotado no fim do `security.sql`.

## Próximos passos (fase 2)
- Impressão automática na térmica (ESC/POS via ponte local).
- Relatórios (faturamento do dia, mais vendidos, horário de pico).
- Cashback/fidelidade.
- Fotos reais dos produtos (campo `image_url` já existe — é só preencher).
