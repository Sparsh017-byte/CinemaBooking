# Cinema Booking

A Go-based backend for managing movie seat reservations in a cinema booking system. The project focuses on handling seat holds, confirmations, releases, and concurrent booking behavior in a way that prevents duplicate seat reservations.

## Main objective

The service allows clients to:

- list available movies
- view seats for a specific movie
- temporarily hold a seat for a user session
- confirm a held seat as a final booking
- release a held seat when it is no longer needed

The booking flow is built around session-based reservations so that a seat can be held for a short period of time and then either confirmed or released automatically if it expires.

## Why this project exists

The core challenge is concurrency safety: multiple users can attempt to book the same seat at the same time. The implementation uses Redis-backed atomic operations with TTLs to ensure only one request wins for a seat while others fail gracefully.

## Technologies used

- Go 1.27
- net/http for the HTTP server and routing
- Redis 7 via Docker Compose for state storage and concurrency control
- github.com/redis/go-redis/v9 for Redis client integration
- github.com/google/uuid for generating unique session IDs
- Docker Compose for local Redis setup
- Go testing for concurrent booking validation

## Project structure

```text
CinemaBooking/
├── cmd/
│   └── main.go                 # Application entry point and HTTP route setup
├── internal/
│   ├── adapters/
│   │   └── redis/
│   │       └── redis.go        # Redis client factory and connection setup
│   ├── booking/
│   │   ├── concurrent_store.go  # Concurrent in-memory store implementation
│   │   ├── domain.go           # Booking model and store interface
│   │   ├── handler.go          # HTTP handlers for booking actions
│   │   ├── memory_store.go     # Basic in-memory booking store
│   │   ├── redis_store.go      # Redis-backed booking logic and seat/session management
│   │   ├── service.go          # Booking service layer
│   │   └── service_test.go     # Concurrency test for seat booking
│   └── utils/
│       └── utils.go            # Utility helpers for JSON responses
├── docker-compose.yaml         # Redis service definition
├── go.mod                      # Go module definition and dependencies
├── go.sum                      # Dependency lock file
├── .gitignore                  # Git ignore rules
└── README.md                   # Project documentation
```

## Main application flow

The entrypoint in `cmd/main.go` initializes an HTTP router and registers the booking endpoints. A Redis client is created and passed into the booking domain service, which handles seat holds and confirmations via the Redis store.

### HTTP endpoints

- `GET /movies` — returns the list of movies
- `GET /movies/{movieID}/seats` — lists booked/held seat information for a movie
- `POST /movies/{movieID}/seats/{seatID}/hold` — holds a seat for a user
- `PUT /sessions/{sessionID}/confirm` — confirms a previously held session
- `DELETE /sessions/{sessionID}` — releases a seat reservation by session

## Storage and concurrency model

The repository includes both in-memory and Redis-backed implementations:

- `MemoryStore` and `ConcurrentStore` are lightweight local implementations for testing or simple scenarios
- `RedisStore` is the production-style storage layer used in the app
- Redis keys are used to represent seat ownership and session reverse lookups
- Set operations with `NX` and TTL expiry help guarantee that only one user can hold a seat at a time

## Local setup

1. Start Redis with Docker:

```bash
docker compose up -d
```

2. Run the application:

```bash
go run ./cmd
```

3. The server listens on port `8080` by default.


