# 📽️ Starsoft Backend Challenge

O **Starsoft-backend-challenge** é uma API inteligente que simula um sistema de compra de ingressos de cinema.  
A aplicação foi desenvolvida com foco em **eficiência**, **consistência de dados** e **persistência**, utilizando uma arquitetura moderna, escalável e preparada para múltiplas instâncias.

---

## 🧱 Stack utilizada

- **NestJS (Node.js + TypeScript)** — Construção da API
- **Piro** — Sistema de logs (log, debug, warn e error)
- **TypeORM** — ORM para persistência de dados
- **Docker** — Containerização e múltiplas instâncias
- **PostgreSQL** — Banco de dados relacional
- **Redis** — Cache e coordenação entre instâncias
- **Nginx** — Balanceador de carga
- **Husky** — Gerenciador de commits.

---

## 🚀 Iniciando o projeto na sua máquina

### 1️⃣ Clonar o repositório

```bash
git clone https://github.com/LuizHenriqueGomesRibeiro/starsoft
```

### 2️⃣ Instalar as dependências

```bash
npm install
```

### 3️⃣ Subir a aplicação (primeira execução)

```bash
docker-compose up --build
```

---

### 🔁 Escalando instâncias da API

O número de instâncias gerenciadas pelo **Nginx** pode ser ajustado no build:

```bash
docker-compose up --build --scale api=5
```

> Cria **5 instâncias** da API

```bash
docker-compose up --build --scale api=1
```

> Cria apenas **1 instância** da API

Em ambos os casos, um container **Nginx** é iniciado para distribuir dinamicamente as requisições entre as instâncias.

---

### 🪵 Visualização de logs (opcional)

```bash
docker-compose logs -f
```

Exibe em tempo real **somente os logs gerados pelo Piro**:
- `log`
- `debug`
- `warn`
- `error`

### 📦 Commit

```bash
npm run commit
```

Comando roda **eslint** para barrar commit caso haja algum erro no código. Roda também **prettier** para barrar qualquer discrepância estilística no commit. Rodando ***npm run commit***, aparecem quatro opções de flags de commit:
- `feat`: uma nova funcionalidade.
- `fix`: correção de um erro.
- `chore`: configurações e ferramentas.
- `error`: documentação.

---

## 🔐 Autenticação

### `POST /auth/login`

**Params**
```json
{
  "email": "string",
  "password": "string"
}
```

**Response**
```json
{
  "access_token": "string",
  "instanceId": "string"
}
```

---

## 👤 Usuários

### `POST /user`

**Params**
```json
{
  "email": "string",
  "password": "string"
}
```

**Response**
```json
{
  "id": "uuid",
  "email": "string",
  "instanceId": "string"
}
```

---

## 🎬 Sessões de Cinema

### `POST /sessions`

**Params**
```json
{
  "movieTitle": "string",
  "startTime": "Date",
  "price": 0,
  "seats": ["string"]
}
```

**Response**
```json
{
  "id": "uuid",
  "movieTitle": "string",
  "startTime": "Date",
  "seats": [
    {
      "id": "uuid",
      "row": "string",
      "number": 0,
      "bookingId": "uuid",
      "lockedAt": 0,
      "reservationId": "uuid"
    }
  ]
}
```

---

### `GET /seats/session/:sessionId`

**Response**
```json
[
  {
    "id": "uuid",
    "row": "string",
    "number": 0,
    "bookingId": "uuid",
    "lockedAt": 0,
    "reservationId": "uuid"
  }
]
```

---

## 🎟️ Reservas

### `POST /reservations/lock/:id`

Bloqueia os assentos temporariamente.

**Response**
```json
{
  "id": "uuid",
  "userId": "uuid",
  "price": 0,
  "status": "string",
  "seats": [
    {
      "id": "uuid",
      "row": "string",
      "number": 0,
      "bookingId": "uuid",
      "lockedAt": 0,
      "reservationId": "uuid"
    }
  ],
  "createdAt": "Date",
  "instanceId": "string"
}
```

---

### `POST /reservations/confirm/:id`

Confirma a reserva e finaliza a compra.

**Response**
```json
{
  "id": "uuid",
  "userId": "uuid",
  "price": 0,
  "status": "string",
  "seats": [
    {
      "id": "uuid",
      "row": "string",
      "number": 0,
      "bookingId": "uuid",
      "lockedAt": 0,
      "reservationId": "uuid"
    }
  ],
  "createdAt": "Date",
  "instanceId": "string"
}
```
 Ao confirmar compra do ticket, um **Proxy** de **Microsserviço** publica o evento **reservation_confirmed**.

---

### `GET /reservations/my-history`

Retorna o histórico de reservas do usuário autenticado.

**Response**
```json
[
  {
    "id": "uuid",
    "userId": "uuid",
    "price": 0,
    "status": "string",
    "seats": [
      {
        "id": "uuid",
        "row": "string",
        "number": 0,
        "bookingId": "uuid",
        "lockedAt": 0,
        "reservationId": "uuid"
      }
    ]
  }
]
```

## 📊 Teste de Concorrência – Lock de Assentos

 O teste abaixo simula **100 tentativas concorrentes** de reserva para o mesmo recurso.
Apenas a primeira requisição foi bem-sucedida (`201`), enquanto as demais retornaram `400`,
demonstrando consistência no mecanismo de lock distribuído.

 Comando:

```
node teste-concorrencia.js [seatId] [access_token]
```

 Resultado:

```
| Tentativa | Instância        | Status | Tempo |
|-----------|------------------|--------|-------|
| 1         | aa7e47feff86     | 400    | 236ms |
| 2         | 0c03eef54469     | 400    | 217ms |
| 3         | c3f2a4652fc7     | 400    | 185ms |
| 4         | 557dfa268687     | 201    | 206ms |
| 5         | 8c066f9f120e     | 400    | 181ms |
| 6         | aa7e47feff86     | 400    | 208ms |
| 7         | 0c03eef54469     | 400    | 217ms |
| 8         | c3f2a4652fc7     | 400    | 190ms |
| 9         | 557dfa268687     | 400    | 192ms |
| 10        | 8c066f9f120e     | 400    | 191ms |
| 11        | aa7e47feff86     | 400    | 214ms |
| 12        | 0c03eef54469     | 400    | 223ms |
| 13        | c3f2a4652fc7     | 400    | 196ms |
| 14        | 557dfa268687     | 400    | 192ms |
| 15        | 8c066f9f120e     | 400    | 182ms |
| 16        | aa7e47feff86     | 400    | 257ms |
| 17        | 0c03eef54469     | 400    | 223ms |
| 18        | c3f2a4652fc7     | 400    | 219ms |
| 19        | 557dfa268687     | 400    | 212ms |
| 20        | 8c066f9f120e     | 400    | 225ms |
| 21        | 8c066f9f120e     | 400    | 198ms |
| 22        | aa7e47feff86     | 400    | 244ms |
| 23        | 0c03eef54469     | 400    | 264ms |
| 24        | c3f2a4652fc7     | 400    | 221ms |
| 25        | 0c03eef54469     | 400    | 262ms |
| 26        | aa7e47feff86     | 400    | 270ms |
| 27        | 557dfa268687     | 400    | 210ms |
| 28        | c3f2a4652fc7     | 400    | 246ms |
| 29        | 8c066f9f120e     | 400    | 227ms |
| 30        | 557dfa268687     | 400    | 245ms |
| 31        | aa7e47feff86     | 400    | 268ms |
| 32        | 0c03eef54469     | 400    | 262ms |
| 33        | c3f2a4652fc7     | 400    | 245ms |
| 34        | 557dfa268687     | 400    | 246ms |
| 35        | 8c066f9f120e     | 400    | 226ms |
| 36        | 0c03eef54469     | 400    | 276ms |
| 37        | aa7e47feff86     | 400    | 278ms |
| 38        | 557dfa268687     | 400    | 268ms |
| 39        | c3f2a4652fc7     | 400    | 263ms |
| 40        | aa7e47feff86     | 400    | 277ms |
| 41        | 8c066f9f120e     | 400    | 244ms |
| 42        | 0c03eef54469     | 400    | 280ms |
| 43        | c3f2a4652fc7     | 400    | 271ms |
| 44        | 8c066f9f120e     | 400    | 248ms |
| 45        | 557dfa268687     | 400    | 270ms |
| 46        | aa7e47feff86     | 400    | 281ms |
| 47        | 0c03eef54469     | 400    | 282ms |
| 48        | 557dfa268687     | 400    | 268ms |
| 49        | c3f2a4652fc7     | 400    | 268ms |
| 50        | 8c066f9f120e     | 400    | 251ms |
| 51        | aa7e47feff86     | 400    | 208ms |
| 52        | 0c03eef54469     | 400    | 246ms |
| 53        | c3f2a4652fc7     | 400    | 199ms |
| 54        | 557dfa268687     | 400    | 217ms |
| 55        | 8c066f9f120e     | 400    | 191ms |
| 56        | aa7e47feff86     | 400    | 249ms |
| 57        | 0c03eef54469     | 400    | 247ms |
| 58        | c3f2a4652fc7     | 400    | 213ms |
| 59        | 557dfa268687     | 400    | 218ms |
| 60        | 8c066f9f120e     | 400    | 205ms |
| 61        | aa7e47feff86     | 400    | 259ms |
| 62        | 0c03eef54469     | 400    | 255ms |
| 63        | 557dfa268687     | 400    | 239ms |
| 64        | c3f2a4652fc7     | 400    | 239ms |
| 65        | 8c066f9f120e     | 400    | 212ms |
| 66        | aa7e47feff86     | 400    | 260ms |
| 67        | c3f2a4652fc7     | 400    | 246ms |
| 68        | 0c03eef54469     | 400    | 256ms |
| 69        | 557dfa268687     | 400    | 247ms |
| 70        | 8c066f9f120e     | 400    | 225ms |
| 71        | aa7e47feff86     | 400    | 268ms |
| 72        | c3f2a4652fc7     | 400    | 255ms |
| 73        | 0c03eef54469     | 400    | 269ms |
| 74        | 557dfa268687     | 400    | 248ms |
| 75        | 8c066f9f120e     | 400    | 230ms |
| 76        | aa7e47feff86     | 400    | 271ms |
| 77        | 0c03eef54469     | 400    | 270ms |
| 78        | c3f2a4652fc7     | 400    | 255ms |
| 79        | 557dfa268687     | 400    | 253ms |
| 80        | 8c066f9f120e     | 400    | 236ms |
| 81        | 0c03eef54469     | 400    | 273ms |
| 82        | aa7e47feff86     | 400    | 271ms |
| 83        | c3f2a4652fc7     | 400    | 255ms |
| 84        | 557dfa268687     | 400    | 254ms |
| 85        | 8c066f9f120e     | 400    | 241ms |
| 86        | aa7e47feff86     | 400    | 275ms |
| 87        | 0c03eef54469     | 400    | 273ms |
| 88        | c3f2a4652fc7     | 400    | 259ms |
| 89        | 557dfa268687     | 400    | 256ms |
| 90        | 8c066f9f120e     | 400    | 239ms |
| 91        | c3f2a4652fc7     | 400    | 259ms |
| 92        | 0c03eef54469     | 400    | 272ms |
| 93        | aa7e47feff86     | 400    | 273ms |
| 94        | 557dfa268687     | 400    | 257ms |
| 95        | 8c066f9f120e     | 400    | 240ms |
| 96        | aa7e47feff86     | 400    | 274ms |
| 97        | 0c03eef54469     | 400    | 271ms |
| 98        | c3f2a4652fc7     | 400    | 259ms |
| 99        | 557dfa268687     | 400    | 258ms |
| 100       | 8c066f9f120e     | 400    | 240ms |
```

  Observe que somente um disparo realmente faz a ***Reserva*** do assento. O ***Nginx*** distribui as requisições entre as instâncias e o ***Redis*** comunica a reservas entre as diferentes instâncias da api.

---

## 🧠 Observações Técnicas

- O **Redis** garante sincronização de bloqueio de assentos entre múltiplas instâncias
- O **Nginx** distribui requisições automaticamente
- O **instanceId** permite rastrear qual instância atendeu cada requisição
- A arquitetura suporta **escala horizontal real**

---

## 📌 Autor

**Luiz Henrique Gomes Ribeiro**  
Desafio técnico — Starsoft
