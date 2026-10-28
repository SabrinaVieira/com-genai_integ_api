# Plano de testes de integração

## Objetivo
Validar a integração entre front-end, back-end e Google Maps JavaScript API para exibir um mapa com pins nas coordenadas dos usuários, respeitando boas práticas de desenvolvimento web, autenticação JWT, validação de dados geográficos, gestão de erros e otimização de performance.

## Escopo
- Front-end: carregamento do mapa, renderização de marcadores, tratamento de erros, feedback visual e feedback de autenticação.
- Back-end: API REST com autenticação por JWT e contrato de dados em JSON.
- Google Maps JavaScript API: carregamento do mapa, uso de marcadores e clusters.

## Premissas e arquitetura
- O back-end expõe endpoints REST protegidos por token JWT no header HTTP:
  - Authorization: Bearer <token>
- Cada requisição ao endpoint de dados do mapa deve incluir o token válido.
- O payload de retorno do back-end deve seguir um contrato mínimo padronizado em JSON.
- O front-end deve tratar falhas de autenticação, rede, timeout, dados inválidos e indisponibilidade da API do Google Maps.
- O mapa deve manter-se funcional e informativo mesmo em cenários de ausência de dados ou falha de serviço.

## Contrato recomendado do payload do back-end

### Estrutura recomendada
- id: string ou number
- latitude: number
- longitude: number
- title: string (opcional)
- description: string (opcional)
- category: string (opcional)
- iconUrl: string (opcional)

### Regras
- latitude: faixa válida entre -90.0 e 90.0
- longitude: faixa válida entre -180.0 e 180.0
- coordenadas em notação decimal WGS 84
- precisão recomendada: até 6 casas decimais
- dados inválidos ou fora do intervalo devem ser descartados antes da renderização no Google Maps

### Exemplo
```json
{
  "id": "loc_123",
  "latitude": -23.550520,
  "longitude": -46.633308,
  "title": "Ponto de Interesse",
  "description": "Local de referência",
  "category": "estabelecimento",
  "iconUrl": "https://example.com/icon.png"
}
```

## Regras de autenticação

### JWT
- O token deve ser enviado em todas as requisições protegidas.
- Formato: Authorization: Bearer <token>

### Comportamento do front-end em falhas
- HTTP 401 Unauthorized: redirecionar imediatamente para a tela de login e limpar o token armazenado em localStorage/sessionStorage.
- HTTP 403 Forbidden: exibir banner/toast informando ausência de permissão, mantendo a sessão ativa.

## Tratamento de erros e fallback visual

### Mensagens esperadas
- 400 Bad Request: "Parâmetros de busca inválidos."
- 401 Unauthorized: redirecionar para login.
- 403 Forbidden: "Acesso negado a estes dados."
- 404 Not Found: "Recurso ou rota não encontrada."
- 500 / Timeout: "Sistema temporariamente indisponível. Tente novamente em instantes."

### Fallback visual
- Manter o estado anterior da tela com overlay/toast em destaque.
- Exibir uma camada de erro superior ao conteúdo principal.
- Incluir botão de ação manual "Tentar novamente".

## Timeout e retry
- Timeout da chamada REST: 5.000 ms (5s)
- Retry automático: até 2 tentativas
- Política de backoff exponencial:
  - tentativa 1: 1s
  - tentativa 2: 2s
- Retry apenas para erros de rede e HTTP 502, 503 e 504
- Não aplicar retry em erros 4xx

## Comportamento para lista vazia
- A lista pode retornar []
- O mapa deve permanecer visível
- Centralizar o mapa nas coordenadas padrão do sistema ou na localização atual do usuário/cidade padrão
- Exibir card/toast informativo, por exemplo: "Nenhum ponto encontrado para os filtros selecionados."
- O mapa nunca deve ser ocultado como resposta a lista vazia

## Indisponibilidade da Google Maps API
- Tratar falhas no carregamento do script via onerror na tag <script> ou rejeição do Promise do SDK
- Quando o carregamento falhar:
  - substituir o container do mapa por uma caixa de estado vazio (empty state)
  - exibir ícone de alerta e texto: "Não foi possível carregar o mapa interativo."
  - se aplicável, renderizar alternativa em lista/tabela padrão

## Dados geográficos e validação
### Faixas válidas
- Latitude: -90.0 a 90.0
- Longitude: -180.0 a 180.0

### Regras de negócio
- Descartar ou sinalizar pontos fora do Bounding Box do país quando o escopo for nacional, por exemplo Brasil:
  - Latitude: -33.75 a 5.27
  - Longitude: -73.98 a -34.79
- Descartar pontos nulos, indefinidos ou fora das faixas numéricas antes de renderizar no SDK do Google Maps
- Não enviar coordenadas inválidas para os objetos Marker/AdvancedMarker

## Estratégia para múltiplos pins na mesma localização
- Usar a biblioteca @googlemaps/markerclusterer
- Pins próximos ou coincidentes devem ser agrupados em clusters numéricos
- Ao clicar em um cluster, o mapa deve aplicar zoom automático na região para separar os marcadores
- Para coordenadas estritamente idênticas, aplicar variação visual leve de deslocamento (jitter/spiderfy) quando atingir o zoom máximo

## Critérios de performance
- First Contentful Paint do SDK do Google Maps: <= 1,5 segundo em conexões padrão
- Renderização dos pins: <= 300 ms para até 500 marcadores após o recebimento dos dados do back-end
- Otimização:
  - usar Advanced Markers do Google Maps
  - destruir instâncias antigas de marcadores ao atualizar a lista
  - evitar re-renderização completa do mapa quando apenas parte dos dados muda

## Atualização em tempo real
### Padrão
- Atualização sob demanda por ação do usuário
- Aplicação de filtros e paginação com debounce de 300 ms na busca por área
- Atualizar apenas as posições relevantes sem recarregar todo o mapa

### Cenários dinâmicos
- Polling curto: 10–15 segundos
- Ou WebSocket/SSE para dados em tempo real
- Atualizar apenas os marcadores existentes e preservar estado do mapa

## Cenários de teste

| ID | Objetivo | Pré-condições | Passos | Resultado esperado | Prioridade |
|---|---|---|---|---|---|
| INT-MAP-01 | Validar fluxo principal com dados válidos | Front-end acessível; back-end disponível; token JWT válido; Google Maps API disponível; endpoint retorna payload válido | 1. Acessar a tela do mapa. 2. Confirmar que o script do Google Maps é carregado. 3. Verificar a chamada com header Authorization Bearer. 4. Validar renderização dos pins. | O mapa carrega corretamente; os pins aparecem nas coordenadas corretas; a interface permanece estável e sem erros. | Alta |
| INT-MAP-02 | Validar autenticação JWT inválida | Token ausente, expirado ou inválido | 1. Acessar a tela do mapa. 2. Executar requisição sem token ou com token inválido. 3. Observar comportamento da UI. | HTTP 401 resulta em redirecionamento para login e remoção do token armazenado. | Alta |
| INT-MAP-03 | Validar autorização insuficiente | Usuário autenticado, mas sem permissão ao recurso | 1. Acessar a tela do mapa com usuário sem permissão. 2. Requisitar os dados. 3. Verificar comportamento do front-end. | HTTP 403 exibe toast de acesso negado e mantém a sessão ativa. | Alta |
| INT-MAP-04 | Validar lista vazia retornada pelo back-end | Endpoint responde com [] | 1. Acessar a tela do mapa. 2. Executar busca. 3. Verificar estado do mapa. | O mapa continua visível; centro padrão é mantido; exibe mensagem de nenhum ponto encontrado. | Alta |
| INT-MAP-05 | Validar payload com dados inválidos | Endpoint retorna registros com coordenadas fora do intervalo ou em formato incorreto | 1. Acessar a tela do mapa. 2. Receber dados inválidos. 3. Verificar processamento do front-end. | Dados inválidos são descartados; pins inválidos não são renderizados; a aplicação não quebra. | Alta |
| INT-MAP-06 | Validar 400 Bad Request | Endpoint retorna 400 para parâmetros inválidos | 1. Acessar a tela com parâmetros incorretos. 2. Verificar resposta do backend. | Exibir toast "Parâmetros de busca inválidos." e manter estado anterior da interface. | Alta |
| INT-MAP-07 | Validar 404 Not Found | Endpoint inexistente ou rota inválida | 1. Acessar a funcionalidade. 2. Executar chamada ao endpoint. 3. Verificar resposta do front-end. | Exibir toast "Recurso ou rota não encontrada." sem quebrar a interface. | Média |
| INT-MAP-08 | Validar 500 Internal Server Error | Backend com falha interna | 1. Acessar a tela do mapa. 2. Simular falha no endpoint. 3. Observar feedback do front-end. | Exibir toast amigável com mensagem de indisponibilidade; botão "Tentar novamente" habilitado. | Alta |
| INT-MAP-09 | Validar timeout de requisição | Backend lento ou indisponível | 1. Acessar a tela do mapa. 2. Esperar o timeout de 5s. 3. Verificar comprovação da falha. | Exibir mensagem de indisponibilidade; manter estado anterior; permitir nova tentativa. | Alta |
| INT-MAP-10 | Validar retry automático | Erros de rede ou 502/503/504 | 1. Simular falha inicial do endpoint. 2. Executar requisição. 3. Observar quantidade de tentativas e atraso. | A aplicação tenta até 2 vezes com backoff exponencial; sem retry em 4xx. | Média |
| INT-MAP-11 | Validar indisponibilidade do Google Maps API | Script bloqueado ou falho | 1. Acessar a tela do mapa. 2. Simular falha de carregamento do SDK. 3. Observar estado final da tela. | Exibir empty state com mensagem "Não foi possível carregar o mapa interativo."; manter alternativa em lista/tabela quando aplicável. | Alta |
| INT-MAP-12 | Validar clusterização de pins coincidentes | Vários usuários na mesma coordenada | 1. Carregar dados com múltiplos pins idênticos ou próximos. 2. Observar mapa. 3. Clicar no cluster. | Os pins são agrupados em clusters; ao clicar, o mapa aproxima a região; não há falhas de renderização. | Alta |
| INT-MAP-13 | Validar jitter/spiderfy para pins idênticos | Múltiplos pins na mesma lat/long no zoom máximo | 1. Aplicar zoom máximo. 2. Observar marcadores simultâneos. | Os pins com coordenadas idênticas sofrem deslocamento visual leve e continuam visíveis. | Média |
| INT-MAP-14 | Validar geofencing nacional | Dados com coordenadas fora do Bounding Box do país | 1. Enviar localizações fora do Brasil. 2. Verificar processamento e renderização. | Pontos fora do escopo do país são descartados ou sinalizados conforme regra definida; não aparecem no mapa. | Média |
| INT-MAP-15 | Validar performance com 500 marcadores | Back-end retorna 500 pontos válidos | 1. Carregar mapa com volume elevado. 2. Medir tempo de renderização e estabilidade. | Renderização em <= 300 ms; mapa permanece responsivo; sem travamentos. | Alta |
| INT-MAP-16 | Validar atualização sob demanda | Dados alterados por filtros ou busca por área | 1. Acessar mapa com filtros. 2. Aplicar nova busca. 3. Observar atualização. | O mapa atualiza conforme os filtros; há debounce de 300 ms; sem recarga completa da tela. | Média |
| INT-MAP-17 | Validar polling/dados dinâmicos | Dados em tempo real | 1. Simular atualização periódica de posições. 2. Observar comportamento do mapa. | Só os marcadores relevantes são atualizados; o mapa não é recarregado inteiro; desempenho permanece estável. | Média |

## Critérios de aceite finais
- Requisições protegidas ao back-end devem conter header Authorization com JWT válido.
- Erros de autenticação, autorização, rede e timeout devem ser tratados sem quebra da interface.
- Lista vazia não deve ocultar o mapa nem gerar tela quebrada.
- Dados inválidos geográficos devem ser descartados antes da renderização.
- Google Maps API indisponível deve resultar em empty state consistente.
- Múltiplos pins na mesma localização devem ser agrupados ou tratados visualmente sem perda de usabilidade.
- O sistema deve manter performance adequada para até 500 marcadores.

## Observações finais
Este plano foi revisado conforme boas práticas de desenvolvimento web com APIs do Google Maps e arquitetura REST. Ele pode ser usado como referência para execução manual, revisão por times de produto e automação de testes de integração em ambiente de QA.
