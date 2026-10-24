# Nossa Lista — lista de compras do casal

App de lista de compras (PWA para iPhone) do Mauricio e da esposa. Funciona sem internet no mercado e sincroniza os dois celulares pela nuvem Supabase.

Endereço: https://mausan2013-dotcom.github.io/nossa-lista/ · Supabase: projeto `ddhhfmucyiehpybkzblc` (conta pessoal, plano gratuito).
Publicar mudanças: `~/.local/bin/publica-nossa-lista "mensagem"`.

## Arquivos

- `index.html` — o app inteiro (telas, dados, sincronização, avisos, fotos, leitor)
- `sw.js` — deixa o app abrir sem internet e mostra os avisos
- `zxing.min.js` — biblioteca de leitura de código de barras (@zxing/library 0.21.3, carregada só ao ler)
- `manifest.webmanifest`, `icon-*.png`, `apple-touch-icon.png` — instalação na tela inicial
- `supabase/01-tabelas.sql` — tabelas e regras de acesso
- `supabase/02-liberar-logins.sql` — libera os logins criados (rodar de novo se criar/recriar usuário)
- `supabase/03-avisos-e-fotos.sql` — tabela dos celulares que recebem avisos + pasta das fotos (v1.1)
- `supabase/functions/nl-avisar/index.ts` — função que envia os avisos (v1.1)
- `supabase/SEGREDOS-nao-compartilhar.txt` — chaves dos avisos (NÃO publicar)

## Funções

- Várias listas (Mercado e Farmácia já vêm criadas; dá para criar outras com ícone)
- Itens por categoria, na ordem dos corredores (editável em Ajustes); "2 kg arroz" já preenche quantidade e unidade
- Quantidade, unidade, observação e preço por item; total do carrinho e da lista
- "Finalizar compra" guarda no histórico (total por mês) e tira os itens marcados da lista
- Sugestões ao digitar e botões "+ item" com o que vocês mais compram; repetir compra; enviar a lista por WhatsApp
- Mostra quem adicionou e quem marcou cada item; "Pedir o preço ao marcar"; cópia de segurança
- **v1.1** — Avisos no celular: itens adicionados (juntos num aviso só), compra finalizada, chegou ao mercado; cada um escolhe quais recebe
- **v1.1** — "Estou no mercado" (botão do carrinho na lista): modo mercado (letras grandes, tela acesa, tocar no nome marca, lápis edita) + faixa no outro celular + aviso
- **v1.1** — Código de barras: lê pela câmera (ou digita o número); produto conhecido entra direto; produto novo busca o nome no Open Food Facts
- **v1.1** — Foto do item (fica no aparelho e na pasta privada `lc-fotos`; o produto lembra a foto para as próximas vezes)
- **v1.1** — Orçamento do mês por lista: barra na tela inicial, "resta/passa" no rodapé (contando o carrinho) e no histórico

## Como a sincronização funciona

- Tudo fica gravado no celular primeiro; com internet, sobe para a nuvem em menos de 1 s.
- Com o app aberto, ele confere a nuvem a cada 4 s (uma consulta leve na tabela `lc_dados`).
- Cada registro tem a hora da última mudança (`atualizado`); vale sempre a mais recente, inclusive na nuvem (gatilho `lc_antes_gravar`).
- Nada é apagado de verdade: o registro fica marcado como `apagado`, para o outro celular saber.
- Só os logins que estão em `lc_membros` leem e gravam (RLS).
- "Estou no mercado" é um registro em `config` (`nomercado:<email>`), válido por até 3 h.
- Avisos: o app chama a função `nl-avisar` com o login de quem usa; a função lê `lc_push` pelas regras do banco (sem chave de serviço) e manda para os celulares do outro. Itens adicionados esperam 45 s sem novidade (4 s se o outro está no mercado) ou saem quando o app é fechado.

## Configurar a nuvem (feito em 24/10/2026)

1. Projeto gratuito `nossa-lista` (São Paulo) na conta pessoal.
2. SQL 01 → usuários com **Create new user** + Auto Confirm (o "Invite user" manda link para localhost e não funciona) → desligar "Allow new users to sign up" → SQL 02.

## Configurar a v1.1 (avisos e fotos)

1. SQL Editor → rodar `supabase/03-avisos-e-fotos.sql` (no fim deve listar `lc_push` e `lc-fotos`).
2. Edge Functions → **Deploy a new function** → **Via Editor** → nome `nl-avisar` → colar `supabase/functions/nl-avisar/index.ts` → Deploy.
3. Na função → **Details/Settings** → desligar **Verify JWT with legacy secret** (o login é conferido dentro da função) → Save.
4. Edge Functions → **Secrets** → cadastrar `VAPID_PUBLICA` e `VAPID_PRIVADA` (valores em `SEGREDOS-nao-compartilhar.txt`).
5. Em cada iPhone: app instalado na Tela de Início → Ajustes → **Ativar avisos** → **Enviar teste**.

Projeto gratuito: o Supabase pausa se ficar 7 dias sem nenhum uso. Abrindo o app toda semana, não pausa; se pausar, é só reativar no painel.

## Testes

- 23/10/2026 (v1.0): Playwright, perfil iPhone 14 Pro, claro e escuro, dois celulares com nuvem simulada.
- 24/10/2026 (v1.1): dois celulares com nuvem simulada + câmera falsa mostrando um EAN-13: orçamento, aviso de itens agrupado e rápido com o outro no mercado, modo mercado, faixa, leitura de código (novo e conhecido), foto enviada e vista no outro celular, aviso de compra, abrir pelo aviso. Função `nl-avisar` rodada com Deno contra Supabase e "servidor de push" simulados: recusa sem login, respeita preferências, não avisa quem enviou, remove celular inexistente (410), envio criptografado (aes128gcm) e assinado (VAPID). O aviso de verdade no iPhone só dá para testar nos aparelhos.

## Versões

- **1.1** (24/10/2026) — avisos no celular, "Estou no mercado" + modo mercado, código de barras, foto do item, orçamento do mês.
- **1.0** (23/10/2026) — primeira versão, tema rosa (pedido da esposa; destaque #d63a72, no escuro #e8558a) e ícone rosa.
