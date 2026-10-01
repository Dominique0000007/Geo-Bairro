# Geo Bairro — Projeto completo (Front-end + Back-end + Banco)

Projeto dividido em duas pastas:

```
front-end/          → HTML, CSS e JS (o que o navegador exibe)
├── index.html
├── css/style.css
└── js/script.js

backend/             → Flask (Python) + SQLite
├── app.py
├── database.py
├── requirements.txt
├── iniciar.bat / iniciar.sh
└── static/          → cópia do front-end, opcional (ver nota abaixo)
```

## Ordem atual da página

1. **Estação Local (ESP32)** — no topo. Cada sensor (DHT22, BMP280, MQ-135,
   BH1750) aparece em um card com a especificação técnica e, logo abaixo,
   a leitura mais recente enviada pelo hardware. Atualiza sozinha a cada
   8 segundos (tempo real).
2. **Boletim Meteorológico** e **Previsão de 7 dias** — o formulário de
   busca por localidade foi retirado da tela; essas duas seções agora
   carregam automaticamente para uma localidade fixa, definida na
   constante `LOCAL_PADRAO` no topo do `script.js`. Troque esse valor
   pela cidade/bairro reais do projeto.
3. **Histórico** — agora é o histórico das *leituras da estação*
   (tabela `leituras_esp32`), não mais de buscas por cidade. Fica por
   último, no fim da página.

## Como as duas pastas se conversam

O `front-end/js/script.js` chama sempre `http://127.0.0.1:5000/...` (ver a
constante `API_BASE` no topo do arquivo). O back-end precisa estar
rodando para o site funcionar. CORS já liberado (`flask_cors`) para
aceitar chamadas de qualquer porta/origem (ex: Live Server).

## Como rodar

1. **Ligue o back-end primeiro:**
   - Windows: dois cliques em `backend/iniciar.bat`
   - Mac/Linux: dois cliques em `backend/iniciar.sh`
2. **Abra o front-end:** Live Server na pasta `front-end/`, ou duplo
   clique em `front-end/index.html`.

## Banco de dados

**`leituras_esp32`** — leituras reais da estação (a mais usada agora):

| coluna         | tipo    | descrição                          |
|-----------------|---------|--------------------------------------|
| id              | INTEGER | chave primária                       |
| temperatura     | REAL    | °C (DHT22, -40 a 80)                 |
| umidade         | REAL    | % UR (DHT22, 0 a 100)                |
| pressao         | REAL    | hPa (BMP280, 300 a 1100)             |
| qualidade_ar    | REAL    | ppm (MQ-135)                          |
| luminosidade    | REAL    | lux (BH1750, 1 a 65.535)             |
| data_hora       | TEXT    | timestamp ISO da leitura              |

**`consultas`** — histórico de buscas por localidade (não usado mais na
tela, mas mantido no banco/rota `/api/historico` caso queiram reativar
a busca por cidade no futuro).

```bash
sqlite3 backend/geo_bairro.db "SELECT * FROM leituras_esp32;"
```

## Enviando dados do ESP32

Firmware em MicroPython faz um `POST` para
`http://<ip-do-computador>:5000/api/leituras`:

```json
{
  "temperatura": 24.5,
  "umidade": 61.2,
  "pressao": 1013.4,
  "qualidade_ar": 180,
  "luminosidade": 320
}
```

- `GET /api/leituras/ultima` → leitura mais recente (card do topo).
- `GET /api/leituras?limite=20` → histórico de leituras (seção de baixo).

Todos os campos do POST são opcionais — envie só o que o hardware já
estiver medindo.
