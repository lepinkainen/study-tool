# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
make dev          # run locally (DB_PATH defaults to anki.db, PORT to 8080)
make build        # compile binary
make test         # run tests
go test -run TestName ./...  # run a single test

docker compose up -d   # start via Docker (data persists in ./data/)
docker compose down
```

## Architecture

Single-package Go web app (`package main`) with an embedded frontend.

**Data flow:**
- `main.go` — wires the SQLite DB, registers HTTP routes, embeds `static/` via `go:embed`
- `db.go` — schema, model types (`Deck`, `Card`, `RevLog`, `DeckStats`), all DB functions
- `sm2.go` — pure SM-2 spaced repetition logic; `Review(card, rating, now)` returns the updated card
- `handlers.go` — HTTP handlers that call db + sm2 functions
- `static/` — vanilla JS/HTML/CSS frontend; no build step

**Study priority** (in `nextDueCard`): learning → review → new.

**Card states:** `new` → `learning` → `review` (lapses send back to `learning`).

**Ratings:** 0=Again, 1=Hard, 2=Good, 3=Easy.

**SQLite setup:** `modernc.org/sqlite` (pure Go, no CGO). `MaxOpenConns(1)` because SQLite is single-writer. WAL mode and foreign keys enabled at schema init.

**Docker:** multi-stage build, runs as `nobody`, DB stored in `/data` bind-mounted from `./data/`.

**Environment variables:** `DB_PATH`, `PORT`.
