# Plano de testes de integração

## Objetivo
Validar a integração entre o front-end, o back-end e a API do Google Maps JavaScript para exibir um mapa com pins nas coordenadas dos usuários.

## Escopo
- Front-end: carregamento do mapa e renderização dos pins
- Back-end: endpoint REST responsável por fornecer os dados dos usuários e suas coordenadas
- Google Maps JavaScript API: carregamento do mapa e desenho dos marcadores

## Premissas
- O back-end expõe um endpoint REST para consulta dos usuários.
- O front-end realiza a chamada ao back-end e renderiza os pins no mapa.
- A API do Google Maps deve estar disponível para o carregamento correto do mapa.
- Os cenários a seguir foram escritos para revisão e execução manual ou automatizada.

## Estrutura dos cenários
Cada cenário contém:
- ID
- Objetivo
- Pré-condições
- Passos
- Resultado esperado
- Prioridade

## Cenários de teste

| ID | Objetivo | Pré-condições | Passos | Resultado esperado | Prioridade |
|---|---|---|---|---|---|
| INT-MAP-01 | Validar o fluxo principal de carregamento do mapa com pins em posições válidas | Front-end acessível; back-end disponível; Google Maps JavaScript API disponível; endpoint retorna lista com usuários e coordenadas válidas | 1. Acessar a tela do mapa. 2. Confirmar que o script do Google Maps é carregado. 3. Verificar a chamada ao endpoint do back-end. 4. Observar o mapa renderizado. 5. Confirmar que os pins aparecem nas coordenadas corretas. | O mapa é carregado com sucesso; os dados do back-end são recebidos corretamente; os pins são exibidos nas posições esperadas; a interface permanece estável. | Alta |
| INT-MAP-02 | Validar o comportamento com lista vazia de usuários | Back-end disponível; endpoint retorna uma lista vazia | 1. Acessar a tela do mapa. 2. Executar a consulta ao back-end. 3. Observar o que acontece no mapa e na interface. | O sistema continua funcionando; o mapa não quebra; nenhum pin é renderizado; a IU exibe o estado vazio, se esse comportamento estiver definido. | Alta |
| INT-MAP-03 | Validar rejeição de dados inválidos do back-end | Back-end disponível; endpoint responde com dados inconsistentes ou incompletos | 1. Acessar a tela do mapa. 2. Simular resposta com campos ausentes, coordenadas fora do intervalo ou estrutura inválida. 3. Observar o processamento no front-end. | O front-end n��o quebra; registros inválidos são ignorados ou tratados conforme regra de negócio; não há pin em coordenada inválida; a interface continua estável. | Alta |
| INT-MAP-04 | Validar falha HTTP 400 do back-end | Back-end responde 400 Bad Request para a consulta | 1. Acessar a tela do mapa. 2. Executar a consulta. 3. Verificar o tratamento da falha no front-end. | A aplicação trata a falha sem quebrar; o usuário recebe feedback adequado, caso definido; o mapa não fica em estado inconsistente. | Alta |
| INT-MAP-05 | Validar falha HTTP 401 ou 403 | Endpoint exige autenticação/autorização; cliente sem credenciais válidas ou sem permissão | 1. Acessar a tela do mapa. 2. Realizar a chamada ao endpoint. 3. Verificar a resposta e tratamento do front-end. | O front-end trata a falha de autenticação/autorização sem quebrar; exibe mensagem apropriada ou bloqueia a listagem de marcadores, conforme regra definida. | Alta |
| INT-MAP-06 | Validar falha HTTP 404 para o endpoint | Endpoint inexistente, removido ou respondendo 404 | 1. Acessar a rota da funcionalidade. 2. Verificar a chamada ao endpoint. 3. Observar o comportamento da aplicação. | A aplicação trata a ausência do recurso sem falha crítica; o erro é exibido de forma controlada; não há tela branca ou crash. | Média |
| INT-MAP-07 | Validar falha HTTP 500 do back-end | Back-end indisponível ou com erro interno | 1. Acessar a tela do mapa. 2. Simular falha do endpoint. 3. Observar o comportamento da interface. | O front-end não entra em erro não tratado; o mapa continua estável; a falha é exibida de maneira controlada. | Alta |
| INT-MAP-08 | Validar indisponibilidade do Google Maps JavaScript API | Script da API indisponível, bloqueado ou falhou no carregamento | 1. Acessar a tela do mapa. 2. Simular indisponibilidade da API do Google. 3. Observar o comportamento da interface. | O sistema trata a falha sem quebrar; exibe mensagem de erro ou fallback, se houver; mantém a interface funcional. | Alta |
| INT-MAP-09 | Validar timeout de resposta do back-end | Back-end lento ou respondendo após timeout | 1. Acessar a tela do mapa. 2. Simular latência alta no endpoint. 3. Monitorar o comportamento da aplicação antes do timeout. | O front-end trata o timeout com feedback ao usuário; não fica travado; não renderiza pins sem dados válidos. | Alta |
| INT-MAP-10 | Validar resposta do back-end com dados parcialmente válidos | Endpoint retorna uma mistura de registros válidos e inválidos | 1. Acessar a tela do mapa. 2. Receber payload com alguns usuários válidos e outros sem coordenadas ou com estrutura inconsistente. 3. Observar a renderização. | Só os registros válidos são renderizados; os dados inválidos são ignorados sem quebrar a funcionalidade; o mapa continua operacional. | Média |
| INT-MAP-11 | Validar comportamento com múltiplos pins na mesma coordenada | Back-end retorna vários usuários com mesma latitude/longitude | 1. Acessar a tela do mapa. 2. Verificar os dados retornados. 3. Observar a exibição dos pins. | Os pins são exibidos de forma consistente; não há erro de renderização; sobreposição ou agrupamento, se houver, é tratado conforme a regra definida. | Média |
| INT-MAP-12 | Validar desempenho e estabilidade com volume elevado de usuários | Back-end retorna um conjunto grande de registros | 1. Acessar a tela do mapa com grande volume de dados. 2. Verificar tempo de carregamento e renderização. 3. Observar estabilidade da interface. | O mapa e os pins carregam sem travamentos; a interface responde adequadamente; não há falhas visuais ou lentidão crítica fora do critério esperado. | Média |
| INT-MAP-13 | Validar comportamento da tela sem dados após primeira carga | O mapa foi carregado em um momento anterior; posteriormente o back-end retorna lista vazia ou sem dados válidos | 1. Acessar a tela do mapa. 2. Carregar dados válidos. 3. Atualizar ou refazer a consulta para retorno vazio/inválido. 4. Observar o mapa e a interface. | A tela atualiza corretamente; o mapa não mantém pins obsoletos; o estado de vazio ou erro é refletido na UI conforme o comportamento definido. | Média |
| INT-MAP-14 | Validar continuidade da funcionalidade quando a resposta muda de estado | O endpoint começa com dados válidos e depois retorna falha ou dados inconsistentes | 1. Acessar a interface. 2. Consultar dados válidos. 3. Simular mudança de resposta para falha ou dados inválidos. 4. Observar a UI e o mapa. | O sistema reage de maneira segura: limpa ou mantém estado consistente sem quebra; não exibe pins antigos como se fossem válidos. | Média |

## Critérios de aceite
- O mapa deve ser carregado corretamente quando a API do Google Maps estiver disponível e os dados do back-end forem válidos.
- O front-end não deve quebrar em cenários de falha HTTP, timeout ou dados inconsistentes.
- A interface deve tratar erros e ausência de dados de maneira controlada.
- O mapa deve refletir apenas registros com dados válidos e coerentes para renderização.
- Os pins devem ser exibidos nas coordenadas corretas quando a resposta do back-end estiver correta.

## Pendências e requisitos não especificados
Os itens abaixo precisam ser confirmados antes da execução final ou da automação dos testes, porque não foram definidos no escopo informado:

1. Definição da autenticação do back-end
   - O endpoint exige token, sessão ou API key?
   - Como o front-end deve se comportar quando a autenticação falha?

2. Estrutura exata do payload do back-end
   - Quais campos são obrigatórios?
   - Qual é o formato de latitude e longitude?
   - Existe contrato formal (OpenAPI, Swagger, JSON Schema)?

3. Comportamento para lista vazia
   - A interface deve mostrar mensagem, manter o mapa sem pins ou ocultar o mapa?

4. Tratamento de erro na UI
   - Quais mensagens de erro devem aparecer para 400, 401, 403, 404, 500 e timeout?
   - O produto define fallback visual para estas situações?

5. Timeout e retry
   - Qual é o timeout aceitável para a chamada REST?
   - Há necessidade de retry automático?

6. Indisponibilidade da API do Google Maps
   - Há fallback visual ou mensagem de erro definida pelo produto?

7. Múltiplos pins na mesma localização
   - A aplicação deve mostrar todos os pins, agrupá-los ou usar algum mecanismo de cluster?

8. Dados geográficos
   - Há regras de negócio para coordenadas fora de um país ou região específica?
   - Há intervalos aceitáveis para latitude e longitude já definidos?

9. Critérios de performance
   - Qual tempo máximo aceitável para carregamento do mapa e renderização dos pins?

10. Atualização em tempo real
   - O mapa deve atualizar automaticamente sem recarregar a tela?

## Observações finais
Este plano deve ser revisado junto ao product owner e ao time de backend antes da automatização, para confirmar regras de negócio, mensagens de erro e contratos de dados. O documento foi estruturado para permitir revisão, execução manual e futura automatização de testes de integração.
