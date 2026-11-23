# Liderança Lounge Bar

Starter de aplicação web para cardápio por QR Code e operação de pedidos, com Next.js, TypeScript, Tailwind CSS e Supabase.

## O que já está incluído
- Dashboard administrativo responsivo com identidade premium preto/dourado.
- Visão geral de pedidos, indicadores e mesas.
- Navegação inicial para pedidos, cardápio e mesas.
- Cliente Supabase preparado para uso no servidor e no navegador.
- Migration SQL com tabelas centrais, índices, políticas RLS e atualização `updated_at`.
- Seed de 15 mesas com tokens de demonstração.
- Rota pública de cardápio `/menu/[tableToken]` para validar o QR Code e a mesa.

## Requisitos
- Node.js 20.9 ou superior
- npm
- Projeto Supabase (gratuito serve para desenvolvimento)

## Executar localmente
1. Descompacte o projeto e entre na pasta:
   ```bash
   cd lideranca-lounge
   ```
2. Instale as dependências:
   ```bash
   npm install
   ```
3. Copie `.env.example` para `.env.local` e preencha:
   - `NEXT_PUBLIC_SUPABASE_URL`
   - `NEXT_PUBLIC_SUPABASE_ANON_KEY`
   - `NEXT_PUBLIC_APP_URL`
4. No Supabase, abra **SQL Editor** e execute, nesta ordem:
   - `supabase/migrations/202611200001_initial_schema.sql`
   - `supabase/seed.sql`
5. Inicie:
   ```bash
   npm run dev
   ```
6. Abra http://localhost:3000

## Rotas
- `/` — painel administrativo inicial.
- `/admin/orders` — fila operacional demonstrativa.
- `/admin/products` — gestão de cardápio (estrutura inicial).
- `/admin/tables` — lista das 15 mesas.
- `/menu/M01-demo-token` — página pública de demonstração da Mesa 01.

## Conectar Supabase
O arquivo `src/lib/supabase/server.ts` cria o cliente server-side. As variáveis públicas devem ser configuradas no `.env.local`.
O dashboard atual usa dados demonstrativos para que a interface rode imediatamente. A próxima etapa é trocar os dados mockados por consultas Supabase e ativar autenticação administrativa.

## Segurança e pagamento
- Nunca coloque `SUPABASE_SERVICE_ROLE_KEY` ou token do Mercado Pago em variáveis `NEXT_PUBLIC_*`.
- Não aceite valor, preço ou estado de pagamento enviados pelo navegador como verdade.
- O PIX ainda **não está integrado**. A integração real exige credenciais do gateway, criação da cobrança em rota server-side, persistência do identificador externo e webhook assinado/idempotente.
- O pedido só deve entrar em preparo após confirmação validada no servidor.
- Antes de produção: habilitar autenticação, revisar RLS, limitar acesso ao painel, validar webhooks, configurar domínio/HTTPS, backups e política de privacidade.

## Criar QR Codes reais
A seed cria tokens de demonstração simples (`M01-demo-token` etc.). Antes de imprimir para uso real, substitua por tokens aleatórios longos e não previsíveis. O QR Code deve apontar para:
`https://SEU-DOMINIO/menu/TOKEN-DA-MESA`

## Estrutura
```text
src/app                 rotas Next.js
src/components          componentes compartilhados
src/lib/supabase        clientes Supabase
src/lib                 dados demonstrativos e utilitários
supabase/migrations     schema SQL
supabase/seed.sql       15 mesas iniciais
```

## Fluxo completo (versão atual)
1. Cliente escaneia o QR da mesa e abre `/menu/TOKEN`, confirma +18, monta o pedido e envia (`POST /api/orders`).
2. O servidor recalcula os preços a partir de `src/lib/catalog.ts` e recusa itens indisponíveis.
3. A equipe entra em `/login`, usando `STAFF_PASSWORD`, e abre `/admin/orders`.
4. O painel atualiza a cada 4 s. Com "Ativar som e avisos", toca um alerta e mostra notificação do navegador.
5. Fluxo do pedido: aguardando pagamento → confirmar pagamento (caixa) → em preparo → pronto → entregue. Cancelar é possível antes de entregar.

## Variáveis de acesso da equipe
- `STAFF_PASSWORD`: senha única da equipe (interina).
- `STAFF_SECRET`: chave longa e aleatória usada para assinar o cookie de sessão.

## Cliente: acompanhamento e chamado
- Após enviar, o cliente vê o status do pedido (recebido → pagamento → preparo → pronto → entregue). A página atualiza a cada 5 s.
- O botão de sino chama o garçom ou pede a conta. A equipe vê o chamado no topo do painel e marca como atendido.

## Limitações atuais
- Os pedidos ficam na memória do servidor. Em desenvolvimento funcionam, mas na Netlify não são persistidos. Antes de usar de verdade, ligar ao Supabase (tabelas `orders` e `order_items`).
- Login com senha única. Próxima etapa: Supabase Auth, com contas por pessoa e cargos (dono, caixa, cozinha).
- Pagamento PIX ainda não integrado; a confirmação é manual pelo caixa.

## Deploy na Netlify
- Use **importar do Git** (conectar o repositório), não o "Drop" de arrastar a pasta. O Drop é para sites estáticos e não instala as dependências nem roda `npm run build`.
- Configure as variáveis de ambiente no painel do Netlify: `NEXT_PUBLIC_APP_URL`, `STAFF_PASSWORD` e `STAFF_SECRET`.
- Se o build falhar, abra o log do deploy e copie as últimas linhas em vermelho.

## Mesas
- A administração adiciona mesas em **Mesas e QR Codes** (até 100 por vez, limite total de 300).
- Cada mesa nova recebe um token aleatório, e o link `/menu/TOKEN` vai no QR Code.
- Mesas desativadas mostram "QR Code não encontrado" para o cliente.
- Execute também `supabase/migrations/202611200002_tables_scale.sql` no Supabase, para remover o limite de 15 mesas do banco.
