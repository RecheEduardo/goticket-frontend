# GoTicket Frontend

GoTicket é uma plataforma de venda de ingressos para eventos. Este repositório contém a interface web, ou seja, as telas que o público, os clientes, os organizadores e os administradores usam no navegador. Ela é escrita em React com TypeScript.

Este frontend não funciona sozinho. Todos os dados (eventos, usuários, pedidos, pagamentos, fila de espera) vêm do backend do projeto, que é um repositório separado:

https://github.com/JoaoRamosVS/goticket-backend

O backend é uma API REST em Java com Spring Boot, PostgreSQL, Redis, Stripe e S3 (simulado localmente com LocalStack). Ele resolve o problema central do projeto, que é vender ingressos sob pico de demanda sem vender o mesmo lugar duas vezes e sem cobrar o cliente em dobro. O frontend é a camada que apresenta isso ao usuário: mostra os eventos, coloca o comprador na sala de espera virtual quando o evento está concorrido, monta o carrinho, envia o pedido e exibe o pagamento. Para rodar o frontend com dados reais, você precisa do backend rodando antes. O README do backend explica como subi-lo com Docker.


## O que cada pessoa consegue fazer

Visitantes, sem login, navegam pela página inicial, buscam eventos por texto, filtram por categoria e abrem a página de detalhes de cada evento.

Clientes, depois de criar uma conta e entrar, escolhem uma data do evento, passam pela sala de espera quando ela está ativa, selecionam setor e quantidade de ingressos num mapa interativo do local, preenchem os dados dos portadores dos ingressos, pagam com cartão pela Stripe e acompanham o status do pedido. Na área "Minha conta" eles editam perfil, endereço e senha e veem o histórico de pedidos.

Organizadores se cadastram pela mesma tela de cadastro, escolhendo o modo de organizador. Depois de entrar, eles têm um painel com o status dos próprios eventos, criam eventos num assistente de várias etapas, editam eventos existentes e configuram setores, datas e lotes de ingressos.

Administradores gerenciam tudo o que existe na plataforma: eventos, espaços (locais de evento, incluindo um editor visual do mapa de setores), clientes, organizadores, categorias e pedidos.


## Do que você precisa

Você precisa do Node.js e do npm. O Vite 7, que é a ferramenta de build do projeto, exige Node 20.19 ou mais recente (ou 22.12 em diante na linha 22). Este README foi verificado com Node 22.22 e npm 10.9.

Você precisa do backend GoTicket rodando, normalmente em http://localhost:8080. Sem ele as telas abrem, mas toda listagem, login ou compra falha, porque não há de onde buscar os dados.

Para ver as imagens dos eventos, o frontend busca os arquivos num bucket S3. Em desenvolvimento esse bucket é simulado pelo LocalStack, que o docker-compose do backend já sobe na porta 4566. Se o LocalStack não estiver no ar, as imagens aparecem quebradas, mas o restante funciona.

Há um detalhe importante sobre a porta. O backend só aceita requisições vindas de http://localhost:5173, por causa da configuração de CORS dele (CORS é a regra do navegador que bloqueia chamadas entre endereços diferentes, a menos que o servidor autorize). A porta 5173 é a padrão do Vite. Se ela estiver ocupada e o Vite escolher outra, o navegador vai bloquear as chamadas à API. Libere a porta 5173 antes de rodar.


## Como rodar

1. Suba o backend seguindo o README dele e confirme que http://localhost:8080 responde.

2. Clone este repositório e instale as dependências:

```bash
git clone https://github.com/RecheEduardo/goticket-frontend.git
cd goticket-frontend
npm install
```

3. Crie um arquivo `.env` na raiz do projeto com o conteúdo abaixo. O repositório ainda não traz um `.env.example`, então use este bloco como modelo:

```bash
VITE_API_URL=http://localhost:8080
VITE_S3_PUBLIC_URL=http://localhost:4566/goticket-dev
```

4. Inicie o servidor de desenvolvimento:

```bash
npm run dev
```

5. Abra http://localhost:5173 no navegador. Você deve ver a página inicial do GoTicket com os eventos de demonstração que o backend cria ao subir.

Para entrar como administrador, use as credenciais iniciais documentadas no README do backend, na seção "Credenciais iniciais". O backend também cria um organizador de demonstração. Ele não cria nenhum cliente, então cadastre um pela tela "/cadastro" para testar a compra.

### Variáveis de ambiente

O Vite só entrega ao código do navegador as variáveis cujo nome começa com `VITE_`. O valor delas é copiado para dentro dos arquivos JavaScript no momento do build, então qualquer pessoa que abrir o site consegue lê-lo. Por isso nunca coloque senhas ou chaves secretas nessas variáveis.

| Variável | Obrigatória | Exemplo | Para que serve |
|---|---|---|---|
| `VITE_API_URL` | Sim | `http://localhost:8080` | Endereço base da API do backend. Usado em `src/services/api.ts` para todas as requisições e para a renovação de token. Sem ela, as chamadas vão para o próprio servidor do Vite e falham. |
| `VITE_S3_PUBLIC_URL` | Não | `http://localhost:4566/goticket-dev` | Endereço público do bucket de imagens. Usado em `src/utils/events.ts`. Se ausente, o código já usa esse mesmo valor do LocalStack como padrão. |

A chave pública da Stripe não fica no frontend. O backend a devolve junto com os dados do pedido, e o frontend a usa apenas na tela de pagamento. Ela é configurada no `.env` do backend, na variável `STRIPE_PUBLISHABLE_KEY`.


## Scripts disponíveis

Os comandos abaixo estão definidos no `package.json`.

`npm run dev` inicia o servidor de desenvolvimento em http://localhost:5173, com recarregamento automático a cada alteração salva. É o comando do dia a dia.

`npm run build` primeiro verifica os tipos de todo o projeto com o compilador do TypeScript (`tsc -b`) e, se não houver erros, gera os arquivos de produção na pasta `dist/`. Veja a seção "Estado atual" sobre o resultado desse comando hoje.

`npm run preview` serve a pasta `dist/` gerada pelo build, para você conferir localmente como o site se comporta em produção.

`npm run lint` roda o ESLint em todo o projeto e aponta problemas de estilo e possíveis erros, como regras de hooks do React violadas.

O projeto não tem script de testes porque ainda não há testes automatizados.


## Como o código está organizado

A pasta `src/` é dividida assim:

```text
src/
├── main.tsx          ponto de entrada: monta o React na página
├── App.tsx           todas as rotas e as regras de acesso por perfil
├── index.css         Tailwind e variáveis de tema (cores, raios, sombras)
├── pages/            uma tela por rota, agrupadas por área
│   ├── public/       páginas abertas e o fluxo de compra
│   ├── auth/         login e cadastro
│   ├── client/       área "Minha conta"
│   ├── organizer/    área do organizador
│   └── admin/        área do administrador
├── layouts/          moldura de cada área (menu, barra lateral, rodapé)
├── features/         a lógica de cada funcionalidade, uma pasta por domínio
├── components/
│   ├── ui/           componentes base do shadcn/ui e cartões de evento
│   ├── layout/       barra de navegação, rodapé e barras laterais
│   └── shared/       peças reutilizadas, como o scroll suave
├── services/api.ts   cliente HTTP único que conversa com o backend
├── stores/           estado global (hoje, só a autenticação)
├── types/            tipos usados por várias features
├── lib/              funções utilitárias puras, como cn() e leitura do JWT
└── utils/            formatação de eventos, máscaras e validações de formulário
```

A ideia central é separar a tela da lógica. Um arquivo em `pages/` deveria ser fino: ele lê os parâmetros da URL, chama hooks de uma feature e organiza os componentes na tela. Todo o resto mora em `features/<nome>/`, que segue sempre o mesmo formato. Em `services/` ficam as funções que chamam a API. Em `hooks/` ficam os hooks (funções do React que guardam estado e efeitos) que usam esses serviços e expõem dados prontos para a tela. Em `components/` ficam os pedaços visuais daquela funcionalidade. Em `types/` ficam os formatos dos dados trocados com o backend. Algumas features também têm `utils/` para funções auxiliares e esquemas de validação.

As features existentes são:

`public-landing`, a página inicial e a landing institucional. `event-search` e `category-search` cuidam da busca por texto e por categoria. `event-details` monta a página de um evento. `waiting-room` cuida da sala de espera virtual. `ticket-purchase` faz a escolha de setor e quantidade, incluindo o mapa de setores. `checkout` reúne os dados dos portadores, a cotação e a criação do pedido. `order-tracking` acompanha o pedido e mostra o pagamento com a Stripe. `auth` faz login, cadastro de cliente e de organizador, e logout. `client` é a área "Minha conta". `organizer/` tem o painel, a lista e o assistente de criação de eventos do organizador. `event-configure` é a configuração de setores, datas e lotes, compartilhada entre admin e organizador. `admin/` tem uma subpasta para cada cadastro administrativo: eventos, espaços, clientes, organizadores, categorias e pedidos.

Alguns atalhos para tarefas comuns:

Para adicionar ou mudar uma rota, ou mudar quem pode acessá-la, edite `src/App.tsx`. Para entender ou alterar como o token é enviado e renovado, veja `src/services/api.ts`. Para ver o que fica guardado sobre o usuário logado, veja `src/stores/authStore.ts`. A tela de seleção de ingressos começa em `src/pages/public/EventDateTicketsPage.tsx`, e o mapa de setores desenhado no canvas está em `src/features/ticket-purchase/components/SectorMapViewer.tsx`. O editor de mapa usado pelo admin está em `src/features/admin/admin-venues/components/venue-map-editor/`. Os esquemas de validação reaproveitados em vários formulários, como CPF, telefone e CEP, estão em `src/utils/validation/`.


## Como as partes principais funcionam

### Autenticação

Quando você faz login, o backend devolve dois tokens. O access token é um JWT, uma credencial assinada que o servidor confere a cada requisição e que carrega seu nome e seu perfil. Ele vale por pouco tempo. O refresh token serve apenas para pedir um novo access token quando o atual expira, sem que você precise digitar a senha de novo.

Os dois tokens e a data de expiração ficam no `localStorage` do navegador, com os nomes `accessToken`, `refreshToken` e `tokenExpiration`. O hook `src/features/auth/hooks/useLogin.ts` salva esses valores pelo `login` do store de autenticação, `src/stores/authStore.ts`. Esse store é feito com Zustand (uma biblioteca pequena de estado global). Ele lê o perfil do usuário (`ADMIN`, `ORGANIZER` ou cliente) de dentro do próprio JWT e o expõe como `userScope` para o resto da aplicação.

Todas as chamadas à API passam pela instância do Axios em `src/services/api.ts`. Ela tem dois interceptadores, que são funções executadas automaticamente antes de cada requisição e depois de cada resposta. O de requisição coloca o access token no cabeçalho `Authorization`. O de resposta trata o erro 401, que é a resposta do backend quando o token expirou. Nesse caso ele chama `/auth/refresh` com o refresh token, guarda os novos tokens e repete a requisição original de forma transparente. Se várias requisições recebem 401 ao mesmo tempo, apenas a primeira faz a renovação e as outras esperam numa fila até o novo token chegar. Se a renovação falhar, o usuário é deslogado.

### Controle de acesso

Em `src/App.tsx` existem três componentes que funcionam como portões de rota: `ProtectedClientRoute`, `OrganizerRoute` e `AdminRoute`. Cada um consulta o store de autenticação. Quem não está logado vai para `/login`. Quem está logado com o perfil errado é mandado para a área do próprio perfil e recebe um aviso na tela. As rotas `/admin/*` e `/organizer/*` também recebem aqui a moldura das respectivas áreas. As rotas de compra e de "Minha conta" exigem um cliente logado. Administradores e organizadores não compram ingressos.

Esse controle é apenas de navegação e de experiência do usuário. Quem garante de fato que alguém não acesse o que não pode é o backend, que valida o token em cada chamada.

### Fluxo de compra

A compra percorre cinco telas, nesta ordem.

Na página do evento (`/evento/:eventId`), o cliente escolhe uma data. O componente `TicketSelector` leva para a sala de espera, `/evento/:eventId/fila`.

Na sala de espera, o hook `src/features/waiting-room/hooks/useWaitingRoom.ts` entra na fila do evento pelo backend e consulta a posição a cada 4 segundos. Quando o evento não está em alta demanda, o backend responde na hora que o cliente foi admitido, e a tela passa direto. Quando está, o cliente vê a posição e o tempo estimado até ser admitido. Na admissão, o backend entrega um token de admissão, que o frontend carrega para as próximas telas. A data escolhida fica guardada no `sessionStorage` para que um F5 nessa tela não perca o contexto.

Na seleção de ingressos (`/evento/:eventId/data/:eventDateId/ingressos`), o cliente escolhe setor e quantidade de cada tipo de ingresso, por exemplo inteira e meia. Quando o local tem mapa, os setores aparecem num desenho interativo feito com react-konva, uma biblioteca de desenho em canvas. O estado da seleção fica fora do desenho, no hook `useTicketSelection`, para que um clique não redesenhe o mapa inteiro.

No checkout (`/checkout`), o frontend pede ao backend uma cotação com o valor total e as taxas, coleta os dados de cada portador (e o tipo de elegibilidade, quando há meia-entrada) e envia o pedido. Cada envio carrega o cabeçalho `Idempotency-Key`, um identificador único gerado no navegador (veja `src/features/checkout/utils/idempotency-key.ts`). Com ele, se o cliente clicar duas vezes ou a rede repetir a requisição, o backend devolve o mesmo pedido em vez de criar outro. O token de admissão vai no cabeçalho `X-Queue-Token`. Se o backend responder 403 porque a admissão expirou, o cliente volta para a fila.

No acompanhamento do pedido (`/pedidos/:orderId`), a tela mostra os itens, o prazo para pagar e o formulário de cartão da Stripe, montado com os dados que o backend devolve. O hook `useOrderPolling` consulta o status a cada 3 segundos até ele chegar a um estado final: pago, cancelado, expirado ou reembolsado.


## Como contribuir mantendo o padrão

As convenções completas estão em `CLAUDE.md`. As que mais afetam quem escreve código são estas.

Importe sempre pelo alias `@/`, que aponta para `src/`. Por exemplo, `import goTicketApi from '@/services/api'`.

Faça toda chamada HTTP por `goTicketApi`. Não importe o `axios` diretamente nas features e não monte o cabeçalho `Authorization` à mão, porque o interceptador já cuida do token e da renovação.

Crie formulários com react-hook-form e valide com zod, deixando o esquema perto do formulário que o usa.

Antes de criar um componente visual do zero, verifique se ele já existe em `src/components/ui/` ou se pode ser adicionado pela CLI do shadcn/ui. Combine classes do Tailwind com a função `cn()` de `src/lib/utils.ts`. A identidade visual do projeto usa vidro fosco (blur), sombras definidas, gradientes de azul e bordas bem arredondadas. Os componentes dentro de `src/features/` servem de referência.

Guarde estado local em hooks da própria feature. O único store global é o de autenticação. Combine com a equipe antes de criar outro.

Para criar uma funcionalidade nova:

1. Crie a pasta `src/features/<nome-da-feature>/` com as subpastas `services/`, `hooks/`, `components/` e `types/`.
2. Em `services/`, escreva as funções que chamam o backend usando `goTicketApi`.
3. Em `hooks/`, escreva o hook que chama o serviço e expõe dados, carregamento e erro para a tela.
4. Em `components/`, monte as partes visuais.
5. Crie a página em `src/pages/<área>/` usando o hook e os componentes.
6. Registre a rota em `src/App.tsx`, dentro do portão de acesso correto.
7. Rode `npm run lint` e `npx tsc -b` para conferir que você não adicionou novos erros.


## Estado atual e limitações

O comando `npm run build` falha hoje na etapa de verificação de tipos. O `tsc -b` aponta 10 erros, a maioria variáveis importadas e não usadas, além de duas incompatibilidades de tipo, em `src/features/client/components/ProfileForm.tsx` e em `src/pages/public/OrderTrackingPage.tsx`. A etapa do Vite sozinha (`npx vite build`) gera o `dist/` normalmente, e `npm run dev` funciona. O `npm run lint` também acusa 19 erros e 8 avisos.

O projeto não tem testes automatizados, integração contínua (CI), arquivo de licença nem `.env.example`.

O dashboard e a página de configurações do administrador ainda são placeholders com a mensagem "Em breve". O seletor de cidade da barra de busca da página inicial usa uma lista fixa de exemplo, não dados do backend.

O componente `src/components/shared/SessionWatcher.tsx`, que deslogaria o usuário quando o token expirasse, existe mas está desativado em `App.tsx`. Hoje a sessão depende só da renovação automática do interceptador. Um detalhe relacionado: o login grava `tokenExpiration` como um instante absoluto em milissegundos, enquanto a renovação em `api.ts` grava o valor `expiresIn` recebido do backend. Os dois formatos precisam ser unificados antes de reativar o `SessionWatcher`.

Por fim, o arquivo `package-lock.json` está listado no `.gitignore`, mas já é versionado no repositório. Na prática ele continua sendo usado pelo `npm install`.
