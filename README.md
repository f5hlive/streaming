# F5HLIVE Stream Server

Servidor de streaming com **WebRTC (baixa latência)** e **RTMP (compatibilidade com OBS/encoders)**. 

Inclui SDKs prontos em HTML standalone — basta abrir no navegador, configurar as credenciais e transmitir.

São exemplos avançados, prontos para testes e integrações.

## Credenciais para teste

Inclui SDKs prontos em HTML standalone — basta abrir no navegador, configurar as credenciais e transmitir.

São exemplos avançados, prontos para testes e integrações.

### ⚠️ Atenção

Essas credenciais são apenas para testes. Não é recomendado utilizá-las em produção. O token sempre deve ser passado via backend seguro.

As credenciais funcionam em ambos os protocolos: WebRTC e RTMP.

---

### Streaming Video

- **Account ID:** `a3RD6P`
- **Token:** `56234289apd4a487asgddatq85789h40032n542p2wer9op1w3r7a1j653aerbsa`
- **Publisher:** 10
- **Views:** 20
- **Consumo de minutos:** 10.000
- **Bitrate:** 2.5
- **Record Live:** Não (Plano)
- **Cluster padrão:** `director-ca` (opcional de acordo com o plano)

---

### Streaming Audio

- **Account ID:** `g9FP3A`
- **Token:** `k9x2m471wb8p2q94jsfgtbyz36819r50432m652v8pqr1op7w9r2a4j553bdrntf`
- **Publisher:** 10
- **Views:** 20
- **Consumo de minutos:** 10.000
- **Record Live:** Não (Plano)

---

### Pusher Realtime

- **App ID:** `2437524`
- **API Key:** `11215d6822a0854b4550`
- **API Secret:** `704a78fd977a85392f52`
- **Cluster padrão:** `custom` (opcional de acordo com o plano)
- **Máximo de domínios:** 100
- **Máximo de conexões simultâneas:** 100
- **Máximo de mensagens:** 50.000 (reseta todos os dias)

Cada evento enviado consome uma mensagem, independentemente da quantidade de caracteres.

Se desejar, poderá gerar novas credenciais Free diretamente pelo site.

---

> O servidor detecta automaticamente o tipo de stream pelas credenciais utilizadas. Não é necessário — nem recomendado — configurar modos de transmissão no client.

**URL base:** `https://director-ca.f5hlive.com.br`

---

## API WebRTC

### Publicar (criar live)

```
POST {SERVER_URL}/api/director/publish
Content-Type: application/json

{ "token": "seu-token", "streamName": "seu-stream" }
```

**Resposta:**

```json
{
  "jwt": "...",
  "urls": ["ws://..."]
}
```

### Assistir

```
POST {SERVER_URL}/api/director/subscribe
Content-Type: application/json

{ "streamAccountId": "id-da-conta", "streamName": "seu-stream" }
```

Mesma estrutura de resposta do publish.

### Fluxo de conexão (publish e subscribe)

1. Faça a requisição HTTP e leia `jwt` e `urls` da resposta (o SDK também aceita as variações `data.urls` / `data.url`).
2. Pegue `urls[0]`, converta o prefixo `http` → `ws` (se necessário) e anexe `?token=<jwt>`.
3. Abra o `WebSocket` com essa URL — ele é o canal de signaling.
4. Execute o handshake WebRTC (offer/answer) pelo WebSocket.
5. No publish, o stream local (`getUserMedia`) é enviado pela `RTCPeerConnection`; no subscribe, o stream remoto chega pronto para o elemento `<video>` / `<audio>`.

### Estatísticas

```
GET {SERVER_URL}/api/webrtc/stats
```

Filtros opcionais por query string: `?accountId=` ou `?token=`.

---

## API RTMP

### Ingest via OBS / encoder externo

| Campo | Valor |
|---|---|
| Servidor | `rtmp://{host}:1935/live/{streamName}` |
| Chave de transmissão | `seu-token` |
| Autenticação user/senha | Desmarcada (não usa — o token é a chave) |

- Também funciona o formato `rtmp://{host}:1935/live?stream={streamName}`.
- O servidor valida o token, descobre a conta automaticamente e o público assiste pela URL de view (abaixo) — **o token nunca aparece em URLs públicas**.

Exemplo simples de uso RTMP
ManyCam/OBS/STREAMLABS OBS outros
rtmp://director-ca.f5hlive.com.br/live/teste1
rtmp://director-ca.f5hlive.com.br/live/teste2
rtmp://director-ca.f5hlive.com.br/live/teste3 

Ou 

rtmp://director-ca.f5hlive.com.br:1935/live/teste1
rtmp://director-ca.f5hlive.com.br:1935/live/teste2
rtmp://director-ca.f5hlive.com.br:1935/live/teste3 

### Publish via navegador (SDK JS)

```html
<script src="https://sdk.f5hlive.com.br/rtmp-sdk/f5live-publisher.js"></script>
```

```js
const pub = ...; // instância do SDK
await pub.connect(host, streamName, token);
```

O SDK abre internamente `wss://{host}/api/publish?token=...&stream=...` e transmite o `getUserMedia` por WebSocket.

### Assistir (URLs públicas, sem token)

| Protocolo | URL |
|---|---|
| WS-FLV | `wss://{host}/live/{accountId}/{streamName}.flv` |
| HTTP-FLV | `https://{host}/live/{accountId}/{streamName}.flv` |
| HLS | `{host}/live/{accountId}/{streamName}/index.m3u8` (quando transcode ativo) |

### Endpoints de gerenciamento

| Método | Endpoint | Descrição |
|---|---|---|
| GET | `/api/rtmp/config` | Config e dica de OBS para painéis |
| GET | `/api/rtmp/streams` | Lista de streams ativas |
| GET | `/api/rtmp/stats` | Estatísticas — filtros: `?accountId=` `?streamName=` `?token=` |
| POST | `/api/rtmp/stop` | Encerra sessão — body: `{ "streamKey": "..." }` |

---

## Realtime (eventos de live)

Atualização de status de live/chat em tempo real via cliente Pusher-compatível:

```html
<script src="https://sdk.f5hlive.com.br/pusher/8.5.0/pusher.min.js"></script>
```

```js
const pusher = new Pusher('62c3590ede33abc376ee', { cluster: 'custom' });
```

**Importante:** use exatamente essa config mínima — o SDK resolve host, porta e transporte sozinho. Nunca adicione `wsHost`, `wssPort`, `pusherHost`, `enabledTransports` ou `forceTLS`.

**Regra de canais:** não utilize canais com prefixo `presence-` (limite de 100 usuários e 128 caracteres). Use canais comuns para eventos de live e chat.

---

## SDKs incluídos

| Categoria | Arquivos |
|---|---|
| WebRTC — vídeo | `live-creator.html`, `live-view.html`, `live-creator-simulcast.html`, `live-view-simulcast.html` |
| WebRTC — áudio | `live-creator-audio.html`, `live-view-audio.html`, `live-audio-creator-simulcast.html`, `live-audio-view-simulcast.html` |
| RTMP | `rtmp-creator.html`, `rtmp-player.html`, `rtmp-live.html`, `rtmp-player-live.html`, `rtmp-player-live-audio.html` |
| Painéis de stats | `webrtc-stats-panel.html`, `rtmp-stats-panel.html` |
| Palco / interação | `live-stage.html` |
| SDK JS de publish | `f5live-publisher.js` |

Todos os SDKs já vêm com credenciais de demonstração pré-preenchidas — abra o arquivo, clique no engrenagem de configurações e troque pelo seu `host`, `token`, `account` e `streamName`.

## Modos de utilização que as APIs suportam

### Video Call

Chamadas de vídeo solo peer-to-peer.

Ideal para aplicativos ou plataformas que necessitam de chamadas de vídeo.

---

### Voz Call

Chamadas de áudio solo peer-to-peer.

Ideal para aplicativos ou plataformas que necessitam de chamadas de áudio.

---

### Live Stream Solo — Vídeo

**Salas de vídeo + áudio.**

Live solo de vídeo com público assistindo.

Ideal para aplicativos e plataformas que necessitam de lives ao vivo, no estilo YouTube, Twitch, Bigo e outros aplicativos e sites.

---

### Live Stream Solo — Áudio

**Salas de áudio.**

Live solo de áudio com público ouvinte.

Ideal para aplicativos e plataformas que necessitam de transmissões ao vivo e salas de áudio.

---

### Live Stream Multi — Vídeo

**Salas de vídeo + áudio.**

Live de vídeo com público assistindo ou múltiplas chamadas em reuniões.

Ideal para aplicativos e plataformas no estilo Bigo, Meet, Zoom e outros aplicativos e sites.

---

### Live Stream Multi — Áudio

**Salas de áudio.**

Live de áudio com público ouvinte ou múltiplas chamadas em reuniões somente com áudio.

Ideal para aplicativos e plataformas que necessitam de muitos participantes, no estilo de aplicativos de salas de áudio.

---

## Ideias extras

- Lives
- PK
- Estilos personalizados
- Aplicativos de transmissão
- Lives com convidados em janelas

**API sem limites para a sua imaginação.**
