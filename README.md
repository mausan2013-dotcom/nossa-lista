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
- `supabase/functions/nl-nota/index.ts` — função que lê a nota fiscal pelo link do QR Code (v1.3)
- `divulgacao/` — PDF de convite para teste com amigos (e os arquivos para refazê-lo)
- `PLANO-VERSAO-COMERCIAL.md` — plano técnico para vender o app (várias famílias, cobrança)
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

- **v1.2** — Arrastar o item: para a **direita** marca (ou desmarca, se já está no carrinho); para a **esquerda** apaga, com "Desfazer". A rolagem vertical segue normal; começar colado na borda esquerda é o gesto de voltar do iPhone e é ignorado. Uma dica aparece uma vez.
- **v1.3** — **Nota fiscal (NFC-e de SP e RS):** no Histórico, botão 🧾 → lê o QR Code da nota (ou colar o link). A compra entra com mercado, data/hora, itens, quantidades, preços e desconto. Os nomes da nota ("ERVA MATE SEIVA PURA 1kg") são casados com os itens da lista ("Erva mate", ✓); dá para renomear antes de guardar e o app aprende para a próxima nota. Opção de tirar da lista o que veio na nota e, ao preencher uma compra feita, de tirar dela o que não veio na nota (v1.3.1). Também dá para **preencher uma compra já finalizada** pela nota (botão no aviso "Compra guardada · Ler a nota" ou na edição da compra). Nota repetida não duplica: abre a compra que já existe. O nome do mercado fica guardado pelo CNPJ (`config` → `mercados`).
- **v1.2** — Editar compra feita (Histórico → toque na compra): preencher/corrigir preço e quantidade de cada item (total ao vivo), tirar ou incluir item, mudar a data. O histórico mostra "N sem preço" nas compras com preço faltando. O preço salvo atualiza o catálogo (itens novos já vêm com ele), a não ser que exista compra mais recente do produto.

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

## Configurar a v1.3 (nota fiscal) — feito em 24/10/2026

- Edge Function `nl-nota` (código em `supabase/functions/nl-nota/index.ts`), **Verify JWT desligado**, sem segredos.
- O app chama com `x-region: sa-east-1` (roda em São Paulo). A função só abre endereços oficiais (`fazenda.sp.gov.br`, `sefaz.rs.gov.br`, `svrs.rs.gov.br`), confere login e membro, e devolve só os dados da compra: **nunca** CPF, nome ou endereço do consumidor que aparecem na página.
- Página lida: layout nacional do DANFE NFC-e (classes `txtTit`, `RCod`, `Rqtd`, `RUN`, `RvlUnit`, `valor`, `totalNumb`, `txtTopo`, `chave`). Se um estado mudar a página, é aqui que ajusta.
- Outro estado: incluir o domínio da Secretaria em `HOSTS` e reimplantar.

## Testes

- 23/10/2026 (v1.0): Playwright, perfil iPhone 14 Pro, claro e escuro, dois celulares com nuvem simulada.
- 24/10/2026 (v1.1): dois celulares com nuvem simulada + câmera falsa mostrando um EAN-13: orçamento, aviso de itens agrupado e rápido com o outro no mercado, modo mercado, faixa, leitura de código (novo e conhecido), foto enviada e vista no outro celular, aviso de compra, abrir pelo aviso. Função `nl-avisar` rodada com Deno contra Supabase e "servidor de push" simulados: recusa sem login, respeita preferências, não avisa quem enviou, remove celular inexistente (410), envio criptografado (aes128gcm) e assinado (VAPID). O aviso de verdade no iPhone só dá para testar nos aparelhos.

## Versões

- **1.3** (24/10/2026) — ler a nota fiscal (NFC-e) pelo QR Code: nova compra ou preencher uma compra feita. Testado com nota real do RS (Zaffari, 23 itens, R$ 421,55): função com Deno contra a página real; app com dois celulares, nuvem simulada e câmera falsa mostrando o QR. SP ainda sem teste com nota real.
- **1.3.1** (24/10/2026) — ao preencher uma compra pela nota, opção (marcada) de tirar da compra os itens que não vieram na nota (marcados no mercado, mas não comprados), com os nomes na tela; assim o total e as quantidades ficam iguais aos da nota.
- **1.3.2** (28/10/2026) — notas de SP: o site da Secretaria de SP recusa leitura automática (firewall devolve "Request Rejected" para a função, mesmo saindo do Brasil). Para SP o app não chama a função: abre a folha "Copiar a nota" (botão Abrir a nota → Selecionar Tudo → Copiar → Colar no app) e lê o texto copiado (`lerTextoNota`, mesmo modelo nacional; testado com nota real de SP, 4 itens, R$ 42,52, e com a do RS). Se a função falhar em outra nota, o aviso oferece "Colar o texto". Valores de item e totais arredondados para centavos, como no caixa.

- **1.2.2** (24/10/2026) — ícone do atalho embutido na página (o iPhone do Mauricio ficou com a letra N: não conseguiu baixar o arquivo do ícone ao salvar).
- **1.2.1** (24/10/2026) — X no canto de todas as telas que abrem de baixo (fixo ao rolar); na edição da compra, tirar item agora é uma lixeira para não confundir com o X de fechar.
- **1.2** (24/10/2026) — arrastar para marcar/apagar; editar compras realizadas (preços em casa).
- **1.1.1** (24/10/2026) — o botão "Enviar teste" diz se o celular não está cadastrado ou se o envio falhou.
  - Função `nl-avisar` corrigida: removido o cabeçalho `topic` (a Apple devolvia 400 BadWebPushTopic). Já implantada no Supabase em 24/10/2026.
- **1.1** (24/10/2026) — avisos no celular, "Estou no mercado" + modo mercado, código de barras, foto do item, orçamento do mês.
- **1.0** (23/10/2026) — primeira versão, tema rosa (pedido da esposa; destaque #d63a72, no escuro #e8558a) e ícone rosa.
