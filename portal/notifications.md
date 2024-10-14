# Módulo de notificações do portal (BackEnd)

O módulo de integrações enviará notificações em tempo real para um serviço chamado [Pusher](https://pusher.com/channels/). Este serviço é uma simplificação de implementação para uma arquitetura Pub/Sub.

Resumidamente, o backend enviará requisições para esse serviço e este, por sua vez, enviará os dados da requisição para os seus "ouvintes".

## Integração a partir do front

O front poderá utilizar o seguinte código (semelhante) para ouvir as notificações que chegarão do Back End.

1. Na rota de ``GET: User`` (já roda ao carregar a home, por exemplo), será possível obter o dado ```notification_channel```.
    <details>
    <summary>JSON de User</summary>
    
    ```json
    {
        "id": 12,
        "name": "Teste Updated",
        "email": "teste123@teste.com",
        "tipo": "master",
        "cliente_id": 90,
        "email_verified_at": null,
        "last_login_at": "2024-08-14 10:21:52",
        "last_login_ip": "127.0.0.1",
        "created_at": "2024-07-10T14:27:45.000000Z",
        "updated_at": "2024-08-15T13:46:52.000000Z",
        "phone": "123",
        "cargo": "Desenvolvedor xD",
        "created_by": null,
        "notification_channel": "private-480cfb2c968f631bf29d6dbbc1147a0131cd0f73d4d86d699ce692b943eccdd9"
    }
    ```
    </details>
2. Ao obtê-la, poderá usá-la para ouvir o canal de notificações do cliente (explicado abaixo).
3. Desta forma, poderá utilizar a biblioteca do Pusher para JS (React), a qual precisará de três dados essenciais: PUSHER_APP_KEY, PUSHER_NOTIFICATION_EVENT e o notification_channel (já explicado anteriormente).

Exemplo de código para implementação:

```javascript
<script src="https://js.pusher.com/8.2.0/pusher.min.js"></script>
  <script>

    // Enable pusher logging - don't include this in production
    Pusher.logToConsole = true;

    var pusher = new Pusher(PUSHER_APP_KEY, {
      cluster: 'sa1'
    });

    var channel = pusher.subscribe(userData.notification_channel);
    channel.bind(PUSHER_NOTIFICATION_EVENT, function(data) {
      alert(JSON.stringify(data));
    });
  </script>
```

## Esquemas de dados

O dado obtido a partir das notificações (Pusher) terá o seguinte schema:
```json
{
  "type": "expired_preventive",
  "title": "Manutenção preventiva atrasada!",
  "text": "A manutenção do veículo EJT3H93 está atrasada.",
  "related_identification": "EJT3H93"
}
```
Claro, todos os dados acima podem variar, no entanto, sempre serão strings. No front, sugiro adicionar um tratamento para o dado "type", onde cada um deles poderá ter uma ação diferente. Até então, todas as ações serão de abrir um link (do veículo) em nova guia, por isso, adicionei o dado ``related_identification``, o qual poderá ser utilizado como query param para a requisição de veículo. Exemplo:
``https://portal.itafrotas.com/{{manutencao}}?placa=``+``notificationData.related_identification``.\
Onde `{{manutencao}}` poderá variar de acordo com cada "type" recebido na notificação.

Lista de todos os tipos (type) de notificação implementadas até agora:
1. `expired_preventive`
2. `preventive_close_to_expire`
2. `new_traffic_ticket`
2. `available_accident_report`

## Rotas

Para o recurso das notificações, as seguintes rotas foram criadas:
A rota base para o consumo será `portal_url/notifications/`.

### GET - /
A rota base enviará uma lista paginada e ordenada (por ordem de criação) das notificações, seguindo o seguinte schema:
<details>
<summary>Notifications Schema</summary>
    
```json
    {
    "current_page": 1,
    "data": [
        {
            "id": "b14ce249-29e7-4170-bc97-f795d51fd10f",
            "type": "expired_preventive",
            "read_at": null,
            "created_at": "2024-09-06T13:06:51.000000Z",
            "title": "Manutenção preventiva atrasada!",
            "text": "A manutenção do veículo RCE2F07 está atrasada.",
            "related_identification": "RCE2F07"
        },
        {
            "id": "62e14aad-0d6e-4fb7-bba6-f58f332ecba8",
            "type": "expired_preventive",
            "read_at": null,
            "created_at": "2024-09-06T13:06:50.000000Z",
            "title": "Manutenção preventiva atrasada!",
            "text": "A manutenção do veículo EXH2F93 está atrasada.",
            "related_identification": "EXH2F93"
        },
        {
            "id": "ca466267-b521-44a6-8fc6-30c467e0ca91",
            "type": "preventive_close_to_expire",
            "read_at": null,
            "created_at": "2024-09-06T13:06:49.000000Z",
            "title": "Manutenção preventiva perto do vencimento!",
            "text": "A manutenção do veículo RCE1H04 está próxima de expirar.",
            "related_identification": "RCE1H04"
        },
        ...
    ],
    "first_page_url": "http://portal-do-cliente-backend.test/api/notifications?page=1",
    "from": 1,
    "last_page": 121,
    "last_page_url": "http://portal-do-cliente-backend.test/api/notifications?page=121",
    "links": [
        {
            "url": null,
            "label": "&laquo; Anterior",
            "active": false
        },
        {
            "url": "http://portal-do-cliente-backend.test/api/notifications?page=1",
            "label": "1",
            "active": true
        },
        {
            "url": "http://portal-do-cliente-backend.test/api/notifications?page=2",
            "label": "2",
            "active": false
        },
        {
            "url": "http://portal-do-cliente-backend.test/api/notifications?page=3",
            "label": "3",
            "active": false
        },
        {
            "url": "http://portal-do-cliente-backend.test/api/notifications?page=4",
            "label": "4",
            "active": false
        },
        {
            "url": "http://portal-do-cliente-backend.test/api/notifications?page=5",
            "label": "5",
            "active": false
        },
        {
            "url": "http://portal-do-cliente-backend.test/api/notifications?page=6",
            "label": "6",
            "active": false
        },
        {
            "url": "http://portal-do-cliente-backend.test/api/notifications?page=7",
            "label": "7",
            "active": false
        },
        {
            "url": "http://portal-do-cliente-backend.test/api/notifications?page=8",
            "label": "8",
            "active": false
        },
        {
            "url": "http://portal-do-cliente-backend.test/api/notifications?page=9",
            "label": "9",
            "active": false
        },
        {
            "url": "http://portal-do-cliente-backend.test/api/notifications?page=10",
            "label": "10",
            "active": false
        },
        {
            "url": null,
            "label": "...",
            "active": false
        },
        {
            "url": "http://portal-do-cliente-backend.test/api/notifications?page=120",
            "label": "120",
            "active": false
        },
        {
            "url": "http://portal-do-cliente-backend.test/api/notifications?page=121",
            "label": "121",
            "active": false
        },
        {
            "url": "http://portal-do-cliente-backend.test/api/notifications?page=2",
            "label": "Próximo &raquo;",
            "active": false
        }
    ],
    "next_page_url": "http://portal-do-cliente-backend.test/api/notifications?page=2",
    "path": "http://portal-do-cliente-backend.test/api/notifications",
    "per_page": 12,
    "prev_page_url": null,
    "to": 12,
    "total": 1442
}
```
</details>

Nessa rota, será possível informar dois parâmetros:\
`per_page` (Opcional): O front pode decidir através desse parâmetro, quantas notificações serão retornadas na paginação (padrão = 12).\
`readed` (Opcional): Se for = 1, exibirá apenas lidas, se = 0, apenas não-lidas. Se não informar, levará todas.

### GET - /count

Exibirá dois contadores para as notificações, um para a quantidade total de notificações, a outra para as não-lidas.
<details>
  <summary>Count Schema</summary>
  
  ```json
{
    "unread_count": 3,
    "total_count": 1442
}
  ```
  
</details>

### POST - /read

Aqui servirá para enviar as notificações lidas, o schema (body) esperado pela rota é o seguinte:

```json
{
    "ids": [
        "c1e64a8f-541c-4f00-9a3a-97ad289b06d5",
        "1b1dbcfa-66f0-4da8-8eaf-61b0ac2fce8e",
        "d4cf2348-5e47-411a-bfe6-8a194b87ba3f",
        "32238b5d-7893-4574-ac00-2edf938a5404",
        "08d23bfe-21e5-4981-b3c2-dfa0b750eaf6",
    ]
}

```
Cada ID pode ser obtido a partir da rotas de notificações.

## Sugestões

### Recepção de notificações
Ao receber uma notificação via Pusher, recomendo que incremente o contador das notificações que aparecerá na topbar, além disso, poderia seguir por um dos 2 caminhos:
1. Incluir os dados da notificação Pusher no própio popup de notificações.\
OU
2. Refazer a requisição de notificações, afinal, quando você recebê-la no Pusher, ela já estará disponível no BackEnd também. (Mais recomendado, visto que irá precisar do ID da notificação para marcá-lo como lida).