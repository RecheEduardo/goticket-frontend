# GoTicket Frontend

Interface web da GoTicket, uma plataforma de venda de ingressos projetada para eventos de alta procura. A aplicação é construída com React 19, TypeScript e Vite 7. Ela cobre a jornada completa do comprador, da descoberta do evento até o pagamento e o ingresso com QR Code. Também oferece painéis dedicados para organizadores e administradores.

Este repositório trabalha em conjunto com a API do projeto, mantida em um repositório separado:

**Backend:** https://github.com/JoaoRamosVS/goticket-backend


## Resumo

- **O que é:** o frontend de uma plataforma de ingressos preparada para picos de demanda, com sala de espera virtual, mapa de setores interativo e pagamento integrado à Stripe.
- **Para quem:** quatro perfis com experiências próprias:
    - **visitantes**, que descobrem e pesquisam eventos;
    - **clientes**, que compram e acompanham seus pedidos;
    - **organizadores**, que criam e configuram seus eventos;
    - **administradores**, que gerenciam toda a plataforma.
- **Principais diferenciais técnicos:**
    - sala de espera virtual integrada ao controle de admissão do backend, com posição e tempo estimado atualizados em tempo real;
    - mapa de setores interativo desenhado em canvas com react-konva, alimentado por um editor visual de mapas no painel administrativo;
    - checkout protegido contra pedidos duplicados por chave de idempotência, com cotação calculada no servidor e validação de CPF e de meia-entrada;
    - pagamento com o Payment Element da Stripe e acompanhamento automático do pedido até a emissão dos ingressos com QR Code;
    - sessão com renovação silenciosa de token e fila para requisições simultâneas;
    - arquitetura organizada por features, TypeScript em modo estrito e identidade visual própria sobre Tailwind CSS 4 e shadcn/ui.
- **Para rodar:** suba o backend, crie um `.env` com `VITE_API_URL=http://localhost:8080` e execute `npm install` e `npm run dev`.


## Índice

1. [Sobre o projeto](#sobre-o-projeto)
    - [Relação com o backend](#relação-com-o-backend)
    - [Perfis e funcionalidades](#perfis-e-funcionalidades)
2. [Destaques técnicos](#destaques-técnicos)
    - [Sala de espera virtual](#sala-de-espera-virtual)
    - [Mapa de setores interativo](#mapa-de-setores-interativo)
    - [Editor visual de mapas](#editor-visual-de-mapas)
    - [Checkout protegido contra duplicidade](#checkout-protegido-contra-duplicidade)
    - [Pagamento e acompanhamento do pedido](#pagamento-e-acompanhamento-do-pedido)
    - [Sessão com renovação silenciosa](#sessão-com-renovação-silenciosa)
    - [Formulários com validação brasileira](#formulários-com-validação-brasileira)
    - [Assistentes em etapas](#assistentes-em-etapas)
    - [Busca com sugestões instantâneas](#busca-com-sugestões-instantâneas)
    - [Experiência visual e animações](#experiência-visual-e-animações)
    - [Requisições canceláveis e tipagem estrita](#requisições-canceláveis-e-tipagem-estrita)
3. [Stack tecnológica](#stack-tecnológica)
4. [Primeiros passos](#primeiros-passos)
    - [Pré-requisitos](#pré-requisitos)
    - [Instalação e execução](#instalação-e-execução)
    - [Variáveis de ambiente](#variáveis-de-ambiente)
    - [Dados de demonstração](#dados-de-demonstração)
5. [Scripts disponíveis](#scripts-disponíveis)
6. [Arquitetura do código](#arquitetura-do-código)
    - [Estrutura de pastas](#estrutura-de-pastas)
    - [Organização por features](#organização-por-features)
    - [Catálogo de features](#catálogo-de-features)
    - [Onde encontrar cada coisa](#onde-encontrar-cada-coisa)
7. [Como os fluxos funcionam](#como-os-fluxos-funcionam)
    - [Autenticação e sessão](#autenticação-e-sessão)
    - [Proteção de rotas por perfil](#proteção-de-rotas-por-perfil)
    - [Jornada de compra](#jornada-de-compra)
    - [Do desenho do mapa à escolha do setor](#do-desenho-do-mapa-à-escolha-do-setor)
    - [Ciclo de vida de um evento](#ciclo-de-vida-de-um-evento)
8. [Mapa de rotas](#mapa-de-rotas)
9. [Integração com a API](#integração-com-a-api)
10. [Padrões de desenvolvimento](#padrões-de-desenvolvimento)
    - [Convenções de código](#convenções-de-código)
    - [Design system](#design-system)
    - [Criando uma nova feature](#criando-uma-nova-feature)


## Sobre o projeto

### Relação com o backend

A GoTicket é dividida em dois repositórios que formam um único sistema:

- **[goticket-backend](https://github.com/JoaoRamosVS/goticket-backend)**: API REST em Java com Spring Boot, PostgreSQL, Redis, Stripe e S3. Em desenvolvimento, o S3 é simulado pelo LocalStack. O backend é responsável pelas garantias críticas do negócio:
    - impedir que o mesmo ingresso seja vendido duas vezes, mesmo com milhares de compras simultâneas;
    - impedir cobranças duplicadas por meio de idempotência;
    - manter o checkout disponível durante picos, limitando quantas pessoas entram na compra ao mesmo tempo por meio de uma fila de admissão.
- **goticket-frontend** (este repositório): transforma essas garantias em uma experiência clara para cada perfil de usuário. Ele coloca o comprador na sala de espera quando o evento está concorrido, envia cada pedido com uma chave de idempotência, exibe o mapa de setores e conduz o pagamento.

Toda informação exibida pelo frontend vem da API. Por isso, para usar a aplicação com dados reais, o backend precisa estar em execução. O README do backend explica como subi-lo com Docker em poucos comandos.

### Perfis e funcionalidades

A aplicação atende quatro perfis. Cada um tem rotas, navegação e moldura visual próprias.

- **Visitante** (sem login)
    - Navega por uma home com banner rotativo de destaques, grade de categorias e carrosséis de eventos.
    - Pesquisa eventos pela barra de busca, que sugere resultados enquanto você digita.
    - Explora eventos por categoria, com listagem paginada.
    - Abre a página do evento, com banner em parallax, descrição, organizador, local com mapa do Google incorporado e as datas disponíveis.
    - Conhece a plataforma pela página institucional "Quem somos".
- **Cliente**
    - Cria a conta e entra com e-mail e senha.
    - Escolhe uma data do evento e passa pela sala de espera virtual quando o evento está em alta demanda.
    - Seleciona o setor no mapa interativo ou na lista e define a quantidade de cada tipo de ingresso: inteira, meia-entrada e solidária.
    - Informa nome e CPF de cada portador e, na meia-entrada, a categoria do benefício.
    - Paga com cartão pela Stripe e acompanha o pedido até a emissão dos ingressos, cada um com seu QR Code.
    - Gerencia perfil, endereço, senha e histórico de pedidos na área "Minha conta".
- **Organizador**
    - Faz o cadastro pela mesma tela de cadastro, no modo organizador.
    - Acompanha seus eventos em um painel agrupado por status, como aguardando aprovação, aprovado, adiado, concluído, recusado e cancelado.
    - Cria eventos em um assistente de cinco etapas e envia para aprovação.
    - Edita eventos e configura setores, datas e lotes de ingressos.
- **Administrador**
    - Gerencia eventos, inclusive aprovando, recusando ou alterando seu status.
    - Cadastra espaços em um assistente de cinco etapas que termina no editor visual do mapa de setores.
    - Administra clientes, organizadores, categorias e pedidos, com telas de listagem e de detalhe. As listagens de eventos, espaços, clientes, organizadores e pedidos são paginadas.


## Destaques técnicos

Esta seção apresenta as soluções que tornam o frontend da GoTicket robusto em cenários de alta concorrência e agradável de usar. Cada item indica onde a implementação está no código.

### Sala de espera virtual

Quando muitas pessoas tentam comprar ao mesmo tempo, o backend limita quantas entram no checkout e coloca o excedente em uma fila ordenada. O frontend torna essa espera transparente e confiável.

- **Entrada e acompanhamento:** ao escolher uma data, o cliente entra na fila do evento. A partir daí, a posição é consultada a cada 4 segundos, intervalo alinhado ao ciclo de admissão de 5 segundos do backend.
- **Sem requisições sobrepostas:** o polling usa `setTimeout` recursivo em vez de `setInterval`. Assim, uma nova consulta só começa depois que a anterior termina, mesmo em redes lentas.
- **Tolerância a falhas de rede:** um erro transitório não tira o usuário da fila. A tela apenas tenta de novo no próximo ciclo.
- **Resistente a recarregamento:** a data escolhida fica salva no `sessionStorage`, então um F5 durante a espera não perde o contexto.
- **Passagem direta fora do pico:** se o evento não está em alta demanda, o backend admite o cliente imediatamente e a tela segue sem espera perceptível.
- **Token de admissão:** ao ser admitido, o cliente recebe um token que acompanha a compra até a criação do pedido. Se ele expirar, o checkout devolve o cliente à fila automaticamente.

Implementação: `src/features/waiting-room/hooks/useWaitingRoom.ts` e `src/pages/public/WaitingRoomPage.tsx`.

### Mapa de setores interativo

A escolha do setor acontece sobre a planta real do local, desenhada em canvas com react-konva.

- **Planta vetorial:** o mapa chega do backend como um SVG que contém a imagem base do local e um polígono para cada setor. O frontend interpreta esse SVG, extrai as dimensões e as coordenadas de cada polígono e redesenha tudo no canvas.
- **Cores por estado:** cada setor indica visualmente se está disponível, sob o cursor, selecionado ou esgotado.
- **Mapa e lista sincronizados:** passar o mouse sobre um setor no mapa destaca o mesmo setor na lista lateral, e vice-versa, porque os dois leem o mesmo estado.
- **Responsivo:** um `ResizeObserver` mede o contêiner e reescala o desenho para qualquer largura de tela, mantendo a proporção original da planta.
- **Estado fora do canvas:** a seleção, o hover e as quantidades vivem no hook `useTicketSelection`. O canvas apenas recebe esses valores prontos, o que evita redesenhar o mapa inteiro a cada clique.
- **Limites de quantidade:** a quantidade de cada tipo de ingresso respeita o estoque disponível no lote e o máximo de 10 por tipo.

Implementação: `src/features/ticket-purchase/components/SectorMapViewer.tsx`, `src/features/ticket-purchase/utils/sector-map.helpers.ts` e `src/features/ticket-purchase/hooks/useTicketSelection.ts`.

### Editor visual de mapas

O mapa que o cliente vê é criado pelo administrador dentro da própria aplicação, sem ferramenta externa.

- **Imagem base:** o administrador envia a planta do local, e o editor ajusta a área de desenho às dimensões reais da imagem.
- **Polígonos editáveis:** cada setor recebe um polígono com cor própria. Os vértices podem ser arrastados, e novos vértices podem ser inseridos no ponto mais próximo de qualquer aresta.
- **Coordenadas seguras:** todo vértice é limitado às bordas do mapa, então nenhum setor fica fora da área visível.
- **Geração do SVG:** ao salvar, o editor sincroniza os setores com a API e gera um SVG em que cada polígono carrega o identificador do setor correspondente. Esse arquivo é enviado ao backend e passa a ser a planta oficial usada na compra.
- **Dois modos:** o modo completo permite criar e remover setores. O modo de layout só altera a imagem e os polígonos.

Implementação: `src/features/admin/admin-venues/components/venue-map-editor/`.

### Checkout protegido contra duplicidade

O checkout foi desenhado para que nenhum clique duplo, reenvio ou instabilidade de rede gere dois pedidos.

- **Chave de idempotência:** cada tela de checkout gera um identificador único com `crypto.randomUUID()` e o envia no cabeçalho `Idempotency-Key`. Se a mesma intenção de compra chegar duas vezes, o backend devolve o mesmo pedido em vez de criar outro. A chave fica guardada em um `useRef`, então sobrevive a novas renderizações e a novas tentativas.
- **Cotação no servidor:** o valor total, com taxas, é calculado pelo backend a partir dos itens escolhidos. O frontend nunca envia preços, apenas os lotes e tipos de ingresso.
- **Um formulário por ingresso:** a seleção é expandida em um bloco por portador, com nome e CPF validados.
- **Regras de meia-entrada:** ingressos de meia exigem a categoria do benefício, e a categoria de estudante exige o número do documento. As regras ficam no esquema zod do formulário.
- **Admissão verificada:** o token da fila segue no cabeçalho `X-Queue-Token`. Se o backend responder 403 porque a admissão expirou, o cliente é avisado e levado de volta à fila.

Implementação: `src/features/checkout/` e `src/pages/public/CheckoutPage.tsx`.

### Pagamento e acompanhamento do pedido

Depois que o pedido é criado, a tela de acompanhamento conduz o pagamento e reflete cada mudança de status sem que o cliente precise recarregar a página.

- **Stripe Payment Element:** o formulário de pagamento é montado com o `clientSecret` e a chave pública que o backend devolve junto com o pedido. Nenhuma chave da Stripe precisa ser configurada no frontend.
- **Instância única da Stripe:** a instância é guardada em cache por chave pública, o que evita carregamentos duplicados no modo estrito do React.
- **Status em tempo real:** o pedido é consultado a cada 3 segundos até chegar a um estado final: pago, cancelado, expirado ou reembolsado. Cada estado tem um banner com título e explicação próprios.
- **Contagem regressiva:** enquanto o pagamento está pendente, um cronômetro mostra quanto tempo resta antes de a reserva expirar.
- **Ingressos com QR Code:** após a confirmação, cada ingresso exibe seu QR Code individual, gerado a partir do token devolvido pelo backend.

Implementação: `src/features/order-tracking/` e `src/pages/public/OrderTrackingPage.tsx`.

### Sessão com renovação silenciosa

O usuário permanece logado sem interrupções, mesmo que o token de acesso expire no meio de uma ação.

- Todas as chamadas passam por uma única instância do Axios, que adiciona o token de acesso automaticamente.
- Quando a API responde 401, o interceptador renova o token em segundo plano e repete a requisição original.
- Se várias requisições recebem 401 ao mesmo tempo, apenas uma faz a renovação. As outras aguardam em uma fila e são repetidas com o novo token.
- Se a renovação falhar, a sessão é encerrada de forma limpa.

O funcionamento completo está em [Autenticação e sessão](#autenticação-e-sessão).

### Formulários com validação brasileira

Os formulários usam react-hook-form com esquemas zod. A validação foi adaptada aos documentos e endereços brasileiros.

- **CPF e CNPJ:** máscaras de digitação e validação dos dígitos verificadores, e não apenas do formato.
- **CEP com preenchimento automático:** nos cadastros administrativos de espaços, organizadores e clientes, o endereço é completado pela API pública do ViaCEP assim que o CEP é digitado.
- **Estados:** lista padronizada das 27 unidades federativas.
- **Reaproveitamento:** máscaras, validadores e esquemas ficam centralizados em `src/utils/validation/` e são usados por diferentes features.

### Assistentes em etapas

Cadastros extensos são divididos em assistentes guiados, com indicador de progresso. O usuário pode navegar livremente entre as etapas já alcançadas. Nos assistentes de novo espaço e de novo evento, cada etapa é validada antes de liberar a próxima.

- **Novo espaço** (administrador), em cinco etapas:
    1. Identificação: nome, documento e descrição.
    2. Endereço: localização completa, com busca de CEP.
    3. Organizador: responsável pelo espaço.
    4. Setores: capacidades e metadados.
    5. Mapa SVG: imagem base e polígonos interativos.
- **Novo evento** (organizador), em cinco etapas:
    1. Detalhes: título, descrição, categoria e classificação.
    2. Espaço: local onde o evento acontece.
    3. Datas: uma ou mais sessões.
    4. Imagens: fotos enviadas como `multipart/form-data`, com ordem definida.
    5. Revisão: conferência final e envio para aprovação.
- **Configuração de evento** (organizador e administrador), em três etapas:
    1. Setores: quais setores do espaço participam do evento.
    2. Datas e setores: em quais datas cada setor é vendido.
    3. Lotes: para cada setor em cada data, o preço, a data de ativação e a cota de cada tipo de ingresso.

### Busca com sugestões instantâneas

A barra de busca da navegação mostra até cinco eventos enquanto o usuário digita.

- A consulta só dispara a partir de dois caracteres e depois de 300 milissegundos sem digitação, o que evita uma requisição por tecla.
- Se o usuário continuar digitando, a requisição anterior é cancelada com `AbortController`. Uma resposta antiga nunca sobrescreve uma mais recente.
- Ao confirmar a busca, o usuário vai para a página de resultados completa e paginada.

Implementação: `src/components/layout/Navbar.tsx` e `src/features/event-search/`.

### Experiência visual e animações

A identidade visual segue uma linguagem inspirada no glassmorphism, com superfícies translúcidas e desfocadas, gradientes de azul, sombras bem definidas e bordas generosamente arredondadas.

- **Rolagem suave global** com Lenis, sincronizada ao `requestAnimationFrame` do navegador.
- **Animações de entrada e transição** com framer-motion nas seções da página institucional, no FAQ e na barra de navegação.
- **Carrosséis próprios:** o banner principal avança sozinho e responde a arraste com mouse ou toque, implementado com eventos de ponteiro.
- **Parallax** no banner da página do evento e **efeito de digitação** no título da página institucional.
- **Notificações próprias:** um sistema de toasts com cinco tipos (sucesso, erro, informação, aviso e atualização), animação de entrada e saída e fechamento automático.
- **Componentes acessíveis:** dropdowns, selects, labels e avatares usam primitivas do Radix UI por meio do shadcn/ui.

### Requisições canceláveis e tipagem estrita

- **Cancelamento:** os hooks que buscam dados criam um `AbortController` e cancelam a requisição quando o componente sai da tela ou quando os parâmetros mudam. Isso evita atualizar o estado de um componente que já foi desmontado e elimina condições de corrida entre respostas.
- **TypeScript estrito:** o projeto compila com `strict`, `noUnusedLocals`, `noUnusedParameters`, `noFallthroughCasesInSwitch` e `verbatimModuleSyntax` ativos. Os formatos de dados trocados com o backend são tipados em cada feature.


## Stack tecnológica

| Categoria | Tecnologia | Versão | Papel no projeto |
|---|---|---|---|
| Base | React | 19 | Biblioteca de interface |
| Base | TypeScript | 5.9 | Tipagem estática em modo estrito |
| Build | Vite | 7 | Servidor de desenvolvimento e build de produção |
| Estilo | Tailwind CSS | 4 | Estilização utilitária, integrada pelo plugin `@tailwindcss/vite` |
| Estilo | shadcn/ui e Radix UI | estilo "new-york" | Componentes base acessíveis |
| Estilo | class-variance-authority, clsx, tailwind-merge | - | Variantes de componentes e composição de classes |
| Rotas | React Router | 7 | Roteamento e rotas protegidas |
| Estado | Zustand | 5 | Estado global de autenticação |
| HTTP | Axios | 1.14 | Cliente HTTP com interceptadores |
| Formulários | react-hook-form, zod, @hookform/resolvers | 7, 4, 5 | Formulários e validação declarativa |
| Canvas | Konva e react-konva | 10, 19 | Mapa de setores e editor de mapas |
| Pagamento | Stripe.js e React Stripe.js | 9, 6 | Payment Element e confirmação de pagamento |
| Animação | framer-motion, Lenis, tw-animate-css | 12, 1.3, 1.4 | Transições, rolagem suave e animações utilitárias |
| Ícones | lucide-react | 0.553 | Biblioteca de ícones |
| Qualidade | ESLint e typescript-eslint | 9, 8 | Análise estática, incluindo regras de hooks do React |


## Primeiros passos

### Pré-requisitos

- **Node.js e npm:** o Vite 7 exige Node 20.19 ou mais recente (ou 22.12 em diante na linha 22). O projeto foi verificado com Node 22.22 e npm 10.9.
- **Backend GoTicket em execução:** por padrão ele responde em `http://localhost:8080`. Siga o README do [goticket-backend](https://github.com/JoaoRamosVS/goticket-backend). A forma mais simples é subir tudo com Docker Compose.
- **LocalStack (opcional, mas recomendado):** as imagens de eventos e as plantas dos locais são servidas por um bucket S3. Em desenvolvimento, esse bucket é simulado pelo LocalStack na porta 4566, que o docker-compose do backend já sobe.
- **Porta 5173 livre:** o backend só aceita requisições vindas de `http://localhost:5173`, por causa da sua configuração de CORS. CORS é a regra do navegador que bloqueia chamadas entre endereços diferentes, a menos que o servidor as autorize. A porta 5173 é a padrão do Vite. Se ela estiver ocupada e o Vite escolher outra, o navegador vai bloquear as chamadas à API.

### Instalação e execução

1. Suba o backend e confirme que `http://localhost:8080` responde.
2. Clone este repositório e instale as dependências:

    ```bash
    git clone https://github.com/RecheEduardo/goticket-frontend.git
    cd goticket-frontend
    npm install
    ```

3. Crie um arquivo `.env` na raiz do projeto:

    ```bash
    VITE_API_URL=http://localhost:8080
    VITE_S3_PUBLIC_URL=http://localhost:4566/goticket-dev
    ```

4. Inicie o servidor de desenvolvimento:

    ```bash
    npm run dev
    ```

5. Abra `http://localhost:5173`. A home da GoTicket deve aparecer com os eventos de demonstração criados pelo backend.

### Variáveis de ambiente

O Vite só entrega ao código do navegador as variáveis cujo nome começa com `VITE_`. Elas são incorporadas aos arquivos JavaScript no momento do build e ficam visíveis para qualquer pessoa que abrir o site. Por isso, nunca coloque senhas ou chaves secretas nelas.

| Variável | Obrigatória | Valor de exemplo | Para que serve |
|---|---|---|---|
| `VITE_API_URL` | Sim | `http://localhost:8080` | Endereço base da API. Usado pela instância do Axios em `src/services/api.ts`, pela renovação de token e pelo carregamento das plantas dos locais. |
| `VITE_S3_PUBLIC_URL` | Não | `http://localhost:4566/goticket-dev` | Endereço público do bucket de imagens, usado em `src/utils/events.ts`. Se ausente, o próprio valor de exemplo, do LocalStack, é usado como padrão. |

A chave pública da Stripe não é uma variável do frontend. O backend a devolve junto com cada pedido, e ela é configurada no `.env` do backend, na variável `STRIPE_PUBLISHABLE_KEY`.

### Dados de demonstração

Ao subir em modo de desenvolvimento, o backend popula o banco com dados de exemplo:

- **Administrador:** use as credenciais da seção "Credenciais iniciais" do README do backend.
- **Organizador de demonstração:** o backend cria o organizador `organizer@events.com`, com espaços, eventos e lotes já configurados.
- **Cliente:** o backend não cria clientes. Cadastre um pela tela `/cadastro` para testar a compra.
- **Sala de espera em ação:** o evento "Allianz Live Experience" vem configurado como de alta demanda. A seção "Simular alta demanda" do README do backend mostra como encher a fila com usuários fictícios para acompanhar a sua posição descendo na tela.


## Scripts disponíveis

| Comando | O que faz | Quando usar |
|---|---|---|
| `npm run dev` | Inicia o servidor de desenvolvimento em `http://localhost:5173`, com recarregamento instantâneo a cada alteração salva. | No dia a dia do desenvolvimento. |
| `npm run build` | Verifica os tipos de todo o projeto com `tsc -b` e gera os arquivos otimizados de produção em `dist/`. | Antes de publicar, ou para validar que o projeto compila. |
| `npm run preview` | Serve a pasta `dist/` gerada pelo build. | Para conferir localmente o comportamento da versão de produção. |
| `npm run lint` | Executa o ESLint em todo o projeto, incluindo as regras de hooks do React. | Antes de abrir um pull request. |


## Arquitetura do código

### Estrutura de pastas

```text
src/
├── main.tsx              ponto de entrada: monta o React na página
├── App.tsx               todas as rotas e os portões de acesso por perfil
├── index.css             Tailwind, tokens de tema, cores, raios e sombras
├── pages/                uma tela por rota, agrupadas por área
│   ├── public/           páginas abertas e jornada de compra
│   ├── auth/             login e cadastro
│   ├── client/           área "Minha conta"
│   ├── organizer/        área do organizador
│   └── admin/            área do administrador
├── layouts/              moldura de cada área: navegação, barra lateral e rodapé
├── features/             lógica de cada funcionalidade, uma pasta por domínio
├── components/
│   ├── ui/               componentes base do shadcn/ui, toasts e cartões de evento
│   ├── layout/           barra de navegação, rodapé e barras laterais
│   └── shared/           peças reutilizadas, como a rolagem suave
├── services/
│   ├── api.ts            cliente HTTP único que conversa com o backend
│   └── cep.service.ts    consulta de CEP no ViaCEP
├── stores/               estado global de autenticação, com Zustand
├── types/                tipos compartilhados entre features
├── lib/                  utilitários puros, como cn() e leitura do usuário no JWT
└── utils/                formatação de eventos, máscaras e validações
```

### Organização por features

O código separa a **tela** da **lógica**.

- **`pages/`** contém componentes finos. Eles leem os parâmetros da URL, chamam hooks de uma feature e organizam os componentes na tela.
- **`features/<nome>/`** concentra a lógica de um domínio de negócio, sempre com a mesma estrutura interna:
    - `services/`: funções que chamam a API por meio de `goTicketApi`;
    - `hooks/`: hooks React que usam os serviços e entregam à tela dados prontos, estado de carregamento e erro;
    - `components/`: as partes visuais daquela funcionalidade;
    - `types/`: os formatos dos dados trocados com o backend;
    - `utils/` (quando necessário): funções auxiliares e esquemas de validação.

Essa divisão permite entender uma funcionalidade inteira olhando uma única pasta. Ela também permite reaproveitar uma feature em mais de uma tela. Por exemplo, a configuração de eventos é a mesma para organizadores e administradores, e a sala de espera reutiliza os dados da página do evento.

### Catálogo de features

- **Experiência pública**
    - `public-landing`: home (banner, destaques, categorias, carrosséis e FAQ) e a página institucional "Quem somos".
    - `event-search`: busca de eventos por texto, usada pela navegação e pela página de resultados.
    - `category-search`: página de categoria e listagem dos eventos de cada categoria.
    - `event-details`: página do evento com banner, informações, local, organizador e seletor de datas.
- **Jornada de compra**
    - `waiting-room`: sala de espera virtual.
    - `ticket-purchase`: escolha de setor e quantidades, incluindo o mapa de setores.
    - `checkout`: portadores, cotação e criação do pedido com idempotência.
    - `order-tracking`: acompanhamento do pedido, pagamento pela Stripe e ingressos com QR Code.
- **Conta e autenticação**
    - `auth`: login, cadastro de cliente e de organizador, e logout.
    - `client`: perfil, endereço, senha e histórico de pedidos.
- **Organizador**
    - `organizer/organizer-dashboard`: painel de eventos agrupados por status.
    - `organizer/organizer-events`: listagem e edição dos eventos do organizador.
    - `organizer/organizer-new-event`: assistente de criação de eventos.
    - `event-configure`: setores, datas e lotes, compartilhado com o administrador.
- **Administração**
    - `admin/admin-events`: eventos e mudança de status.
    - `admin/admin-venues`: espaços, assistente de novo espaço e editor de mapas.
    - `admin/admin-clients`, `admin/admin-organizers`, `admin/admin-categories`: cadastros de clientes, organizadores e categorias.
    - `admin/admin-pedidos`: pedidos e resumo detalhado de cada um.

### Onde encontrar cada coisa

- **Adicionar uma rota ou mudar quem pode acessá-la:** `src/App.tsx`.
- **Envio e renovação do token:** `src/services/api.ts`.
- **Dados do usuário logado:** `src/stores/authStore.ts` e `src/lib/jwt.ts`.
- **Tela de seleção de ingressos:** `src/pages/public/EventDateTicketsPage.tsx`.
- **Desenho do mapa de setores na compra:** `src/features/ticket-purchase/components/SectorMapViewer.tsx`.
- **Editor de mapas do administrador:** `src/features/admin/admin-venues/components/venue-map-editor/`.
- **Regras do formulário de checkout:** `src/features/checkout/utils/checkout-form.schema.ts`.
- **Máscaras e validações de CPF, CNPJ e CEP:** `src/utils/validation/`.
- **Formatação de datas, preços e imagens de eventos:** `src/utils/events.ts`.
- **Notificações (toasts):** `src/components/ui/toast.tsx`, usado por meio do hook `useToast`.
- **Cores, raios e variáveis de tema:** `src/index.css`.


## Como os fluxos funcionam

### Autenticação e sessão

1. **Login.** O hook `src/features/auth/hooks/useLogin.ts` envia e-mail e senha e recebe dois tokens:
    - o **access token**, um JWT de curta duração. Um JWT é uma credencial assinada pelo servidor que carrega o nome e o perfil do usuário;
    - o **refresh token**, que serve apenas para obter um novo access token sem pedir a senha de novo.
2. **Armazenamento.** Os tokens e a expiração ficam no `localStorage` com os nomes `accessToken`, `refreshToken` e `tokenExpiration`. O store `src/stores/authStore.ts`, feito com Zustand, lê o perfil de dentro do JWT e o expõe como `userScope`, com o valor `ADMIN`, `ORGANIZER` ou cliente. Por ler o próprio token, o store reconstrói a sessão sozinho quando a página é recarregada.
3. **Requisições autenticadas.** O interceptador de requisição em `src/services/api.ts` adiciona `Authorization: Bearer <token>` a toda chamada. Um interceptador é uma função que o Axios executa automaticamente antes de cada requisição ou depois de cada resposta.
4. **Renovação automática.** Quando uma resposta volta com 401:
    - a primeira requisição afetada chama `/auth/refresh` com o refresh token;
    - as demais que falharem nesse intervalo entram em uma fila de espera;
    - com o novo token salvo, a requisição original e as da fila são repetidas;
    - se a renovação falhar, todas as requisições pendentes são rejeitadas e o usuário é deslogado.
5. **Logout.** O hook `useLogout` avisa o backend em `/auth/logout`, limpa os tokens e o estado local mesmo que a rede falhe e leva o usuário para a home.

### Proteção de rotas por perfil

Em `src/App.tsx`, três componentes funcionam como portões para grupos de rotas:

- **`ProtectedClientRoute`**: libera a jornada de compra e a área "Minha conta" apenas para clientes.
- **`OrganizerRoute`**: libera `/organizer/*` apenas para organizadores e aplica o layout da área.
- **`AdminRoute`**: libera `/admin/*` apenas para administradores e aplica o layout da área.

Em todos eles:

- quem não está logado é levado para `/login`;
- quem está logado com outro perfil é levado para a própria área e recebe um aviso explicando o motivo. Administradores e organizadores vão para o painel, e clientes vão para a home;
- quem já está logado e tenta abrir `/login` ou `/cadastro` é redirecionado para a sua área.

Essa camada cuida da navegação e da experiência. A autorização efetiva é feita pelo backend, que valida o token e o perfil em cada chamada.

### Jornada de compra

A compra percorre cinco telas, e cada uma entrega à próxima apenas o que ela precisa.

1. **Página do evento** (`/evento/:eventId`)
    - O cliente vê os detalhes e escolhe uma data.
    - O seletor de datas leva à sala de espera, carregando a data escolhida.
2. **Sala de espera** (`/evento/:eventId/fila`)
    - O frontend chama `POST /events/{id}/queue` para entrar na fila.
    - Enquanto o status é de espera, consulta `GET /events/{id}/queue/position` a cada 4 segundos e mostra a posição, o total na fila e o tempo estimado.
    - Ao ser admitido, recebe o token de admissão e segue automaticamente para a seleção de ingressos.
3. **Seleção de ingressos** (`/evento/:eventId/data/:eventDateId/ingressos`)
    - O cliente escolhe o setor pelo mapa ou pela lista.
    - Define a quantidade de cada tipo de ingresso disponível no lote atual do setor.
    - Um resumo lateral mostra os itens escolhidos, a quantidade total e o valor. Ao confirmar, a seleção e o token de admissão seguem para o checkout.
4. **Checkout** (`/checkout`)
    - O frontend pede a cotação em `POST /orders/quote` e exibe o total calculado pelo servidor.
    - O cliente preenche os portadores. Para meia-entrada, as categorias de benefício são carregadas de `GET /eligibility-types`.
    - O pedido é criado em `POST /orders`, com os cabeçalhos `Idempotency-Key` e `X-Queue-Token`.
5. **Acompanhamento do pedido** (`/pedidos/:orderId`)
    - Quem chega do checkout vê a tela imediatamente, montada com os dados devolvidos na criação do pedido. Quem abre a página diretamente, como pelo histórico ou no retorno da Stripe, recebe os dados de `GET /orders/{id}`.
    - O cliente paga pelo Payment Element da Stripe. Ao confirmar, a Stripe retorna para a mesma página.
    - `GET /orders/{id}/status` é consultado a cada 3 segundos até o pedido chegar a um estado final.
    - Com o pagamento confirmado, `GET /orders/{id}/summary` traz os tokens que geram o QR Code de cada ingresso.

### Do desenho do mapa à escolha do setor

O mapa de setores atravessa a aplicação de ponta a ponta:

1. **Criação:** no editor visual, o administrador envia a planta do local e desenha um polígono por setor.
2. **Persistência:** ao salvar, os setores são enviados para `PUT /venues/{id}/sectors`, e o SVG gerado vai para `PUT /venues/{id}/sector-map`. Cada polígono do SVG leva no atributo `id` o identificador do setor (`mapElementId`).
3. **Configuração do evento:** organizador ou administrador escolhem quais setores do espaço participam do evento, em quais datas e com quais lotes de preço.
4. **Compra:** a tela de ingressos baixa o SVG de `GET /venues/{id}/sector-map`, interpreta os polígonos e cruza cada `mapElementId` com os setores à venda naquela data. O resultado é desenhado no canvas com disponibilidade e preço de cada área.

### Ciclo de vida de um evento

1. O administrador cadastra o **espaço** e desenha o mapa de setores.
2. O organizador cria o **evento** no assistente de cinco etapas. Ao final, o evento fica com o status **aguardando aprovação**.
3. O administrador revisa o evento e altera seu **status**: aprovado, recusado, adiado, concluído ou cancelado.
4. Organizador ou administrador **configuram a venda**: setores, datas de cada setor e lotes. Cada lote pertence a um setor em uma data e define preço, data de ativação e cotas por tipo de ingresso.
5. O organizador acompanha seus eventos no **painel por status**, em que cada status tem cor, ícone e descrição próprios.


## Mapa de rotas

| Rota | Acesso | Tela |
|---|---|---|
| `/` e `/home` | Público | Home |
| `/quem-somos` | Público | Página institucional |
| `/login` | Público | Login |
| `/cadastro` | Público | Cadastro de cliente ou organizador |
| `/busca/:searchTerm` | Público | Resultados da busca |
| `/categorias` | Público | Todas as categorias |
| `/categoria/:slug` | Público | Eventos de uma categoria |
| `/evento/:eventId` | Público | Página do evento |
| `/evento/:eventId/fila` | Cliente | Sala de espera |
| `/evento/:eventId/data/:eventDateId/ingressos` | Cliente | Seleção de setor e ingressos |
| `/checkout` | Cliente | Checkout |
| `/pedidos/:orderId` | Cliente | Acompanhamento e pagamento do pedido |
| `/minha-conta/perfil` | Cliente | Dados pessoais |
| `/minha-conta/endereco` | Cliente | Endereço |
| `/minha-conta/seguranca` | Cliente | Troca de senha |
| `/minha-conta/pedidos` | Cliente | Histórico de pedidos |
| `/organizer/dashboard` | Organizador | Painel por status |
| `/organizer/eventos` | Organizador | Meus eventos |
| `/organizer/eventos/:eventId` | Organizador | Edição de evento |
| `/organizer/eventos/:eventId/configurar` | Organizador | Setores, datas e lotes |
| `/organizer/novo-evento` | Organizador | Assistente de novo evento |
| `/admin/dashboard` | Administrador | Painel inicial |
| `/admin/eventos` e `/admin/eventos/:eventId` | Administrador | Eventos e edição com mudança de status |
| `/admin/eventos/:eventId/configurar` | Administrador | Setores, datas e lotes |
| `/admin/espacos` e `/admin/espacos/:venueId` | Administrador | Espaços e edição com editor de mapa |
| `/admin/new-venue` | Administrador | Assistente de novo espaço |
| `/admin/clientes` e `/admin/clientes/:clientId` | Administrador | Clientes |
| `/admin/organizadores` e `/admin/organizadores/:organizerId` | Administrador | Organizadores |
| `/admin/categorias` e `/admin/categorias/:categoryId` | Administrador | Categorias |
| `/admin/pedidos` e `/admin/pedidos/:orderId` | Administrador | Pedidos e resumo do pedido |
| `/admin/configuracoes` | Administrador | Configurações da plataforma |


## Integração com a API

Todas as chamadas usam a URL definida em `VITE_API_URL`. As principais integrações da jornada de compra e da autenticação são:

| Momento | Método e rota | Detalhes |
|---|---|---|
| Login | `POST /login` | Devolve access token, refresh token e expiração |
| Cadastro | `POST /clients` e `POST /organizers` | Cadastro de cliente ou de organizador |
| Renovação de sessão | `POST /auth/refresh` | Chamado automaticamente pelo interceptador ao receber 401 |
| Logout | `POST /auth/logout` | Encerra a sessão no servidor |
| Página do evento | `GET /events/{id}` | Detalhes, datas, setores e lotes |
| Entrar na fila | `POST /events/{id}/queue` | Devolve espera com posição, ou admissão imediata |
| Posição na fila | `GET /events/{id}/queue/position` | Consultado a cada 4 segundos |
| Planta do local | `GET /venues/{id}/sector-map` | SVG com imagem base e polígonos dos setores |
| Cotação | `POST /orders/quote` | Total e taxas calculados no servidor |
| Categorias de meia-entrada | `GET /eligibility-types` | Carregado apenas quando há ingresso de meia |
| Criar pedido | `POST /orders` | Cabeçalhos `Idempotency-Key` e `X-Queue-Token` |
| Pedido | `GET /orders/{id}` | Itens, valores, prazo e dados de pagamento |
| Status do pedido | `GET /orders/{id}/status` | Consultado a cada 3 segundos até um estado final |
| Resumo com ingressos | `GET /orders/{id}/summary` | Tokens usados para gerar o QR Code de cada ingresso |

Com o backend em execução, a documentação interativa completa da API fica disponível em `http://localhost:8080/scalar`.


## Padrões de desenvolvimento

### Convenções de código

As convenções completas estão em `CLAUDE.md`. As principais:

- **Imports:** sempre pelo alias `@/`, que aponta para `src/`. Exemplo: `import goTicketApi from '@/services/api'`.
- **HTTP:**
    - toda chamada passa por `goTicketApi`;
    - o `axios` não é importado diretamente nas features;
    - o cabeçalho `Authorization` nunca é montado à mão, porque o interceptador cuida do token e da renovação.
- **Formulários:** react-hook-form com esquemas zod, mantidos perto do formulário que os usa ou em `src/utils/validation/` quando são compartilhados.
- **Estado:**
    - estado de uma funcionalidade fica em hooks da própria feature;
    - o único store global é o de autenticação, e um novo store deve ser alinhado com a equipe antes de ser criado;
    - o estado nunca é mutado diretamente: use spread, `map` e `filter`.
- **Estabilidade da árvore de componentes:** componentes ao lado de um controle interativo, como um resumo lateral, não devem desmontar quando o usuário interage com o controle. Evite `key` dinâmica e condições que recriem o componente pai.
- **Formatação:** Prettier com indentação por tabulação de largura 4, conforme `.prettierrc`.

### Design system

- **Tailwind CSS 4** é a única forma de estilização. Os tokens de cor, raio e sombra ficam em `src/index.css`.
- **shadcn/ui primeiro:** antes de criar um componente do zero, verifique `src/components/ui/` ou adicione o componente pela CLI do shadcn/ui, configurada em `components.json`.
- **Classes com `cn()`:** combine classes com a função `cn()` de `src/lib/utils.ts`, que une `clsx` e `tailwind-merge` e resolve conflitos entre classes do Tailwind.
- **Variantes com `cva`:** use class-variance-authority para variações de um componente, como em `button.tsx`, `badge.tsx` e `alert.tsx`.
- **Linguagem visual:** superfícies translúcidas com `backdrop-blur`, gradientes de azul, sombras bem definidas e bordas bem arredondadas. Os componentes dentro de `src/features/` são a melhor referência.
- **Ícones:** lucide-react.

### Criando uma nova feature

1. Crie a pasta `src/features/<nome-da-feature>/` com as subpastas `services/`, `hooks/`, `components/` e `types/`.
2. Em `types/`, descreva os formatos de requisição e resposta da API.
3. Em `services/`, escreva as funções que chamam o backend com `goTicketApi`, aceitando um `AbortSignal` quando a chamada for de leitura.
4. Em `hooks/`, crie o hook que chama o serviço, cancela a requisição ao desmontar e expõe dados, carregamento e erro.
5. Em `components/`, monte as partes visuais seguindo o design system.
6. Crie a página em `src/pages/<área>/`, usando o hook e os componentes.
7. Registre a rota em `src/App.tsx`, dentro do portão de acesso adequado.
8. Rode `npm run lint` e `npx tsc -b` para confirmar que o código segue as regras do projeto.
