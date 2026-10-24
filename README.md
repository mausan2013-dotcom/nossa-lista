# Nossa Lista — lista de compras do casal

App de lista de compras (PWA para iPhone) do Mauricio e da esposa. Funciona sem internet no mercado e sincroniza os dois celulares pela nuvem Supabase.

## Arquivos

- `index.html` — o app inteiro (telas, dados e sincronização)
- `sw.js` — deixa o app abrir sem internet
- `manifest.webmanifest`, `icon-*.png`, `apple-touch-icon.png` — instalação na tela inicial
- `supabase/01-tabelas.sql` — cria as tabelas e as regras de acesso (rodar uma vez)
- `supabase/02-liberar-logins.sql` — libera os logins criados (rodar depois de criar os usuários)

## Funções (v1.0)

- Várias listas (Mercado e Farmácia já vêm criadas; dá para criar outras com ícone)
- Itens agrupados por categoria, na ordem dos corredores (ordem editável em Ajustes)
- Digitar "2 kg arroz" já preenche quantidade e unidade; a categoria é adivinhada pelo nome
- Quantidade, unidade, observação e preço por item; total do carrinho e da lista
- "Finalizar compra" guarda no histórico (total por mês) e tira os itens marcados da lista
- Sugestões ao digitar e botões "+ item" com o que vocês mais compram em cada lista
- Repetir uma compra do histórico; enviar a lista por WhatsApp/Mensagens
- Mostra quem adicionou e quem marcou cada item
- Opção "Pedir o preço ao marcar"; cópia de segurança em arquivo

## Como a sincronização funciona

- Tudo fica gravado no celular primeiro; com internet, sobe para a nuvem em menos de 1 s.
- Com o app aberto, ele confere a nuvem a cada 4 s (uma consulta leve na tabela `lc_dados`).
- Cada registro tem a hora da última mudança (`atualizado`); vale sempre a mais recente, inclusive na nuvem (gatilho `lc_antes_gravar`).
- Nada é apagado de verdade: o registro fica marcado como `apagado`, para o outro celular saber.
- Só os logins que estão em `lc_membros` leem e gravam (RLS).

## Configurar a nuvem (uma vez)

1. Em supabase.com, na conta do e-mail pessoal, criar um projeto **gratuito** chamado `nossa-lista` (região São Paulo).
2. SQL Editor → colar e rodar `supabase/01-tabelas.sql`.
3. Authentication → Users → **Add user** (com "Auto Confirm") para você e para sua esposa.
4. Authentication → Sign In / Providers → desligar **Allow new users to sign up**.
5. SQL Editor → rodar `supabase/02-liberar-logins.sql` (deve listar os dois e-mails).
6. Project Settings → API: copiar a **Project URL** e a **Publishable key** e colocar em `SUPA_URL` e `SUPA_KEY` no `index.html`.

Projeto gratuito: o Supabase pausa se ficar 7 dias sem nenhum uso. Abrindo o app toda semana, não pausa; se pausar, é só reativar no painel.

## Testes

Testado em 23/10/2026 com Playwright (perfil iPhone 14 Pro, claro e escuro): dois celulares com uma nuvem simulada, item marcado em um aparecendo no outro em menos de 2 s, totais, finalizar, histórico, repetir compra, nova lista.

## Versões

- **1.0** (23/10/2026) — primeira versão, tema rosa (pedido da esposa; destaque #d63a72, no escuro #e8558a) e ícone rosa.
