# Dados e contrato da API para mapa com pins de usuários

## Objetivo
Fornecer uma massa de dados JSON e um conjunto de verificações objetivas para validar o endpoint REST que fornece usuários e coordenadas para renderização em um mapa.

Este documento não presume um endpoint ou contrato final já definido. Ele apresenta um conjunto de exemplos ilustrativos para revisão, validação e alinhamento com o time de backend e produto.

## 1) Estrutura mínima recomendada do payload

### Contrato sugerido
O payload deve conter uma lista de registros com as seguintes propriedades, no mínimo:

- id: string ou number
- latitude: number
- longitude: number
- title: string (opcional)
- description: string (opcional)
- category: string (opcional)
- iconUrl: string (opcional)

### Regras de validação recomendadas
- latitude deve estar entre -90.0 e 90.0
- longitude deve estar entre -180.0 e 180.0
- coordenadas devem seguir WGS 84 em notação decimal
- precisão recomendada: até 6 casas decimais
- itens inválidos devem ser descartados antes da renderização do mapa

### Exemplo de schema em formato conceitual
```json
{
  "type": "array",
  "items": {
    "type": "object",
    "required": ["id", "latitude", "longitude"],
    "properties": {
      "id": { "type": ["string", "number"] },
      "latitude": { "type": "number", "minimum": -90, "maximum": 90 },
      "longitude": { "type": "number", "minimum": -180, "maximum": 180 },
      "title": { "type": "string" },
      "description": { "type": "string" },
      "category": { "type": "string" },
      "iconUrl": { "type": "string", "format": "uri" }
    },
    "additionalProperties": true
  }
}
```

## 2) Massa de dados JSON

### 2.1 Caso válido: lista com usuários em posições corretas
```json
[
  {
    "id": "user_001",
    "latitude": -23.550520,
    "longitude": -46.633308,
    "title": "Usuário 1",
    "description": "Ponto principal",
    "category": "cliente",
    "iconUrl": "https://example.com/icons/client.png"
  },
  {
    "id": "user_002",
    "latitude": -23.548100,
    "longitude": -46.631200,
    "title": "Usuário 2",
    "description": "Ponto secundário",
    "category": "fornecedor",
    "iconUrl": "https://example.com/icons/provider.png"
  },
  {
    "id": "user_003",
    "latitude": -23.560800,
    "longitude": -46.640100,
    "title": "Usuário 3",
    "description": "Ponto de suporte",
    "category": "suporte",
    "iconUrl": "https://example.com/icons/support.png"
  }
]
```

### 2.2 Caso válido: lista com um único ponto
```json
[
  {
    "id": "loc_123",
    "latitude": -23.550520,
    "longitude": -46.633308,
    "title": "Ponto de Interesse",
    "description": "Local de referência",
    "category": "estabelecimento"
  }
]
```

### 2.3 Caso válido: lista vazia
```json
[]
```

### 2.4 Caso inválido: campo ausente
```json
[
  {
    "id": "user_004",
    "latitude": -23.545000,
    "title": "Falta longitude"
  }
]
```

### 2.5 Caso inválido: campo nulo
```json
[
  {
    "id": "user_005",
    "latitude": null,
    "longitude": -46.630000,
    "title": "Latitude nula"
  }
]
```

### 2.6 Caso inválido: tipo incorreto
```json
[
  {
    "id": "user_006",
    "latitude": "-23.550520",
    "longitude": -46.633308,
    "title": "Latitude em string"
  }
]
```

### 2.7 Caso inválido: coordenada fora do limite geográfico
```json
[
  {
    "id": "user_007",
    "latitude": 91.0,
    "longitude": -46.633308,
    "title": "Latitude fora do intervalo"
  }
]
```

### 2.8 Caso inválido: longitude fora do limite geográfico
```json
[
  {
    "id": "user_008",
    "latitude": -23.550520,
    "longitude": 181.0,
    "title": "Longitude fora do intervalo"
  }
]
```

### 2.9 Caso inválido: fora do Bounding Box do país (exemplo Brasil)
```json
[
  {
    "id": "user_009",
    "latitude": -10.000000,
    "longitude": -40.000000,
    "title": "Ponto fora do Brasil"
  }
]
```

### 2.10 Caso inválido: objeto com estrutura incompatível
```json
[
  {
    "id": "user_010",
    "position": {
      "lat": -23.550520,
      "lng": -46.633308
    },
    "title": "Estrutura incompatível"
  }
]
```

### 2.11 Caso inválido: array com campos extras não esperados
```json
[
  {
    "id": "user_011",
    "latitude": -23.550520,
    "longitude": -46.633308,
    "title": "OK",
    "description": "Nome do usuário",
    "extra": { "nivel": "alto" }
  }
]
```

### 2.12 Caso válido com múltiplos pins coincidentes
```json
[
  {
    "id": "user_012",
    "latitude": -23.550520,
    "longitude": -46.633308,
    "title": "Usuário 12"
  },
  {
    "id": "user_013",
    "latitude": -23.550520,
    "longitude": -46.633308,
    "title": "Usuário 13"
  },
  {
    "id": "user_014",
    "latitude": -23.550520,
    "longitude": -46.633308,
    "title": "Usuário 14"
  }
]
```

## 3) Exemplos de requisição e resposta

### 3.1 Requisição HTTP ilustrativa
Abaixo está um exemplo de requisição com autenticação JWT e uso de filtros. O endpoint real deve ser confirmado pelo time de backend.

```http
GET /api/<recurso-de-mapa>?city=Sao%20Paulo&limit=50 HTTP/1.1
Host: api.exemplo.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Accept: application/json
Content-Type: application/json
```

### 3.2 Resposta HTTP válida
```http
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-cache

[
  {
    "id": "user_001",
    "latitude": -23.550520,
    "longitude": -46.633308,
    "title": "Usuário 1",
    "description": "Ponto principal",
    "category": "cliente",
    "iconUrl": "https://example.com/icons/client.png"
  },
  {
    "id": "user_002",
    "latitude": -23.548100,
    "longitude": -46.631200,
    "title": "Usuário 2",
    "description": "Ponto secundário",
    "category": "fornecedor",
    "iconUrl": "https://example.com/icons/provider.png"
  }
]
```

### 3.3 Resposta HTTP vazia
```http
HTTP/1.1 200 OK
Content-Type: application/json

[]
```

### 3.4 Resposta HTTP com erro de autenticação
```http
HTTP/1.1 401 Unauthorized
Content-Type: application/json

{
  "error": "UNAUTHORIZED",
  "message": "Token de acesso ausente ou inválido."
}
```

### 3.5 Resposta HTTP com erro de autorização
```http
HTTP/1.1 403 Forbidden
Content-Type: application/json

{
  "error": "FORBIDDEN",
  "message": "Acesso negado a estes dados."
}
```

### 3.6 Resposta HTTP com erro de validação
```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "error": "INVALID_REQUEST",
  "message": "Parâmetros de busca inválidos.",
  "details": [
    {
      "field": "latitude",
      "issue": "valor fora do intervalo permitido"
    }
  ]
}
```

### 3.7 Resposta HTTP com recurso não encontrado
```http
HTTP/1.1 404 Not Found
Content-Type: application/json

{
  "error": "NOT_FOUND",
  "message": "Recurso ou rota não encontrada."
}
```

### 3.8 Resposta HTTP com falha interna
```http
HTTP/1.1 500 Internal Server Error
Content-Type: application/json

{
  "error": "INTERNAL_ERROR",
  "message": "Sistema temporariamente indisponível. Tente novamente em instantes."
}
```

## 4) Verificações do contrato JSON

### Verificações obrigatórias
- O corpo deve ser um JSON válido
- O corpo deve ser um array quando o endpoint retorna lista de usuários
- O array pode estar vazio, desde que a resposta seja 200 OK
- Cada item do array deve conter id, latitude e longitude
- latitude deve ser do tipo number
- longitude deve ser do tipo number
- latitude deve estar no intervalo [-90, 90]
- longitude deve estar no intervalo [-180, 180]
- campos opcionais, quando presentes, devem ser do tipo esperado
- id deve ser único por registro, quando o contrato exigir unicidade
- nenhum campo de coordenada pode ser null, undefined ou string numérica
- a estrutura deve evitar campos alternativos como position.lat / position.lng quando o contrato pré-definido exigir latitude e longitude diretamente

### Verificações adicionais sugeridas
- O valor de category deve ser uma string, quando informado
- O valor de title deve ser string, quando informado
- O valor de iconUrl deve respeitar formato de URL, quando informado
- Não deve haver campos obrigatórios ausentes em dados válidos
- Em pesquisa por cidade/escopo, a API deve retornar apenas registros compatíveis com o filtro solicitado

## 5) Resultados esperados por cenário

| Cenário | Exemplo | Status HTTP esperado | Resultado esperado |
|---|---|---|---|
| Lista válida | 3 usuários com coordenadas corretas | 200 OK | Mapa renderiza pins corretamente |
| Lista vazia | [] | 200 OK | Mapa continua visível e exibe mensagem de estado vazio |
| Campo ausente | longitude faltando | 200 OK ou 400 Bad Request, conforme regra do backend | Dados inválidos não devem renderizar no mapa |
| Valor nulo | latitude = null | 200 OK ou 400 Bad Request | Registro deve ser descartado ou rejeitado |
| Tipo incorreto | latitude como string | 200 OK ou 400 Bad Request | Validação falha e registro não é processado |
| Coordenada fora do intervalo | latitude 91 ou longitude 181 | 400 Bad Request ou descarte do item | Ponto inválido não aparece no mapa |
| Fora do geofence nacional | exemplo de coordenada fora do Brasil | 200 OK com descarte do item ou 400 Bad Request | Registro fora do escopo é ignorado |
| Erro de autenticação | token ausente/inválido | 401 Unauthorized | Redirecionamento para login e remoção do token |
| Erro de autorização | sem permissão | 403 Forbidden | Toast de acesso negado e sessão ativa |
| Not Found | rota inexistente | 404 Not Found | Mensagem explícita de ausência do recurso |
| Falha interna | erro do servidor | 500 Internal Server Error | Toast amigável com retry |
| Timeout | resposta lenta | 504 Gateway Timeout ou timeout de client | Mensagem de indisponibilidade e nova tentativa |

## 6) Requisitos que precisam ser confirmados
Antes da automação ou homologação final, os itens abaixo devem ser validados com o time responsável:

1. Contrato final do endpoint
   - Qual será o endpoint exato?
   - A resposta será um array simples ou um objeto com metadata + data?
   - Os campos obrigatórios e opcionais estão formalizados?

2. Autenticação
   - O token JWT será exigido em todas as requisições?
   - O header será Authorization: Bearer <token>?
   - O front-end deve limpar o token em 401?

3. Comportamento para lista vazia
   - O retorno vazio deve ser 200 OK?
   - O mapa deve permanecer visível e centralizado no padrão do sistema?

4. Política de validação
   - Os dados inválidos devem ser rejeitados pelo backend ou descartados pelo front-end?
   - A API deve retornar 400 para payload inválido ou o front-end deve filtrar em JS?

5. Geofencing
   - O sistema precisa filtrar apenas coordenadas do Brasil?
   - Se sim, qual é o Bounding Box exato e como a API deve sinalizar os dados descartados?

6. Tratamento de erro na UI
   - Quais mensagens exatas serão exibidas para cada status?
   - Há overlay/toast e botão de retry definido no design?

7. Timeout e retry
   - O timeout de 5.000 ms é aceitável?
   - O retry automático aplica-se somente para rede e 502/503/504?

8. Performance
   - Há um limite específico para número de marcadores a ser exibido?
   - A aplicação usa Advanced Markers ou Marker padrão?

## 7) Recomendação de execução de testes
- Validar sempre o cenário com payload válido antes dos cenários de erro
- Testar pelo menos um caso de cada categoria:
  - lista vazia
  - campos ausentes
  - null
  - tipo incorreto
  - coordenadas fora do intervalo
  - fora do geofence nacional
  - erro de autenticação
  - erro de autorização
  - timeout
  - falha interna
- Para cada caso, registrar:
  - status HTTP
  - corpo da resposta
  - comportamento do front-end
  - mensagem exibida ao usuário
  - decisão final do contrato

## 8) Conclusão
Este documento fornece uma massa de dados útil para testar a integração entre front-end e back-end em um mapa com pins georreferenciados. Ele foi desenhado para não presumir a existência de um endpoint concreto, mas sim servir como base para revisão, validação e alinhamento técnico antes da implementação final.

A validação final deve ser confirmada com o backend, o time de produto e o designer de UX para garantir que o contrato, os erros e os feedbacks visuais estejam totalmente alinhados.
