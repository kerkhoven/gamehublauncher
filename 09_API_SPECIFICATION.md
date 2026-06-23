# API Specification

Version: 1.0

## Purpose

API digunakan untuk komunikasi antara:

* Launcher Client
* Analytics Service
* Admin Panel
* Future Multi Branch System

Prinsip utama:

* Optional
* Offline First
* Local Network Friendly

Launcher harus tetap berfungsi walaupun API tidak tersedia.

---

# Architecture

```text
Launcher
    │
    ▼
REST API
    │
    ▼
SQLite / Future Database
```

---

# Base URL

Development

http://localhost:5000/api

Production

http://server-ip:5000/api

---

# Authentication

Version 1.0

No Authentication

Version 2.0

API Key

Version 3.0

JWT Authentication

---

# Games API

## Get All Games

GET /games

Response:

[
{
"id":1,
"name":"Valorant",
"genre":"FPS"
}
]

---

## Get Game

GET /games/{id}

---

## Search Game

GET /games/search?q=valor

---

## Create Game

POST /games

---

## Update Game

PUT /games/{id}

---

## Delete Game

DELETE /games/{id}

---

# Launch API

## Record Launch

POST /launches

Request

{
"game_id":1,
"client_name":"PC-01",
"launch_time":"2026-01-01T10:00:00"
}

---

## Get Launch History

GET /launches

---

## Get Launch By Game

GET /launches/game/{id}

---

# Analytics API

## Trending Games

GET /analytics/trending

Response

[
{
"game":"Valorant",
"launches":120
}
]

---

## Most Played

GET /analytics/most-played

---

## Daily Statistics

GET /analytics/daily

---

## Monthly Statistics

GET /analytics/monthly

---

# Favorites API

GET /favorites

POST /favorites

DELETE /favorites/{id}

---

# Collections API

GET /collections

POST /collections

PUT /collections/{id}

DELETE /collections/{id}

---

# Recommendation API

GET /recommendations

Response

[
{
"name":"Counter Strike 2"
},
{
"name":"Delta Force"
}
]

---

# Health Check

GET /health

Response

{
"status":"healthy"
}

---

# Future Endpoints

/api/branches

/api/events

/api/tournaments

/api/cloudsync

/api/recommendation-engine

---

# Performance Targets

Response Time:

< 100ms

Concurrent Requests:

100+

Payload Size:

< 100 KB per request

---

# Versioning Strategy

/api/v1

/api/v2

/api/v3

Never break existing clients.
