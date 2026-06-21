# Исходный код сервиса `restaurant-auth-service` (auth_go)

> Полный текущий снимок исходного кода сервиса аутентификации в одном файле.
> Назначение файла — передавать его целиком AI-моделям как контекст для внесения изменений в сервис.

> Репозиторий: `https://github.com/pavelgr1408/auth_go` (ветка `master`).


## Содержание файлов

| # | Файл | Назначение |
|---|------|-----------|
| 1 | `go.mod` | Объявление Go-модуля и прямых/транзитивных зависимостей |
| 2 | `go.sum` | Контрольные суммы зависимостей |
| 3 | `cmd/auth-service/main.go` | Точка входа: загрузка конфигурации, инициализация, запуск HTTP-сервера, graceful shutdown |
| 4 | `internal/config/config.go` | Загрузка конфигурации из переменных окружения |
| 5 | `internal/domain/domain.go` | Доменная модель: сущности, перечисления, доменные ошибки |
| 6 | `internal/token/jwt.go` | Менеджер JWT: выпуск/валидация access-токена (RS256), публикация JWKS |
| 7 | `internal/token/jwt_test.go` | Юнит-тесты JWT-менеджера |
| 8 | `internal/store/postgres.go` | Слой доступа к данным (PostgreSQL, pgx) |
| 9 | `internal/service/service.go` | Бизнес-логика: регистрация, вход, ротация и отзыв токенов, introspection |
| 10 | `internal/httpapi/api.go` | HTTP-слой: маршруты, обработчики, middleware, валидация, маппинг ошибок |
| 11 | `migrations/embed.go` | Встроенный раннер версионированных миграций (advisory lock) |
| 12 | `migrations/001_create_auth_schema.sql` | Создание схемы БД |
| 13 | `migrations/002_seed_dev_users.sql` | Сидинг dev-пользователей |
| 14 | `config/keys/public.pem` | Публичный RSA-ключ для проверки подписи JWT |
| 15 | `config/keys/private.pem` | Приватный RSA-ключ (СЕКРЕТ — содержимое не включено) |
| 16 | `Dockerfile` | Сборка минимального distroless-образа |
| 17 | `docker-compose.yml` | Локальное окружение: PostgreSQL + сервис |
| 18 | `Makefile` | Команды разработки (генерация ключей, тесты, запуск) |
| 19 | `.env.example` | Шаблон переменных окружения |
| 20 | `README.md` | Описание и инструкции запуска |

---


## `go.mod`

```go
module github.com/example/restaurant-auth-service

go 1.24.0

require (
	github.com/golang-jwt/jwt/v5 v5.2.2
	github.com/google/uuid v1.6.0
	github.com/jackc/pgx/v5 v5.7.4
	golang.org/x/crypto v0.36.0
)

require (
	github.com/jackc/pgpassfile v1.0.0 // indirect
	github.com/jackc/pgservicefile v0.0.0-20240606120523-5a60cdf6a761 // indirect
	github.com/jackc/puddle/v2 v2.2.2 // indirect
	golang.org/x/sync v0.12.0 // indirect
	golang.org/x/text v0.23.0 // indirect
)
```


## `go.sum`

```text
github.com/davecgh/go-spew v1.1.0/go.mod h1:J7Y8YcW2NihsgmVo/mv3lAwl/skON4iLHjSsI+c5H38=
github.com/davecgh/go-spew v1.1.1 h1:vj9j/u1bqnvCEfJOwUhtlOARqs3+rkHYY13jYWTU97c=
github.com/davecgh/go-spew v1.1.1/go.mod h1:J7Y8YcW2NihsgmVo/mv3lAwl/skON4iLHjSsI+c5H38=
github.com/golang-jwt/jwt/v5 v5.2.2 h1:Rl4B7itRWVtYIHFrSNd7vhTiz9UpLdi6gZhZ3wEeDy8=
github.com/golang-jwt/jwt/v5 v5.2.2/go.mod h1:pqrtFR0X4osieyHYxtmOUWsAWrfe1Q5UVIyoH402zdk=
github.com/google/uuid v1.6.0 h1:NIvaJDMOsjHA8n1jAhLSgzrAzy1Hgr+hNrb57e+94F0=
github.com/google/uuid v1.6.0/go.mod h1:TIyPZe4MgqvfeYDBFedMoGGpEw/LqOeaOT+nhxU+yHo=
github.com/jackc/pgpassfile v1.0.0 h1:/6Hmqy13Ss2zCq62VdNG8tM1wchn8zjSGOBJ6icpsIM=
github.com/jackc/pgpassfile v1.0.0/go.mod h1:CEx0iS5ambNFdcRtxPj5JhEz+xB6uRky5eyVu/W2HEg=
github.com/jackc/pgservicefile v0.0.0-20240606120523-5a60cdf6a761 h1:iCEnooe7UlwOQYpKFhBabPMi4aNAfoODPEFNiAnClxo=
github.com/jackc/pgservicefile v0.0.0-20240606120523-5a60cdf6a761/go.mod h1:5TJZWKEWniPve33vlWYSoGYefn3gLQRzjfDlhSJ9ZKM=
github.com/jackc/pgx/v5 v5.7.4 h1:9wKznZrhWa2QiHL+NjTSPP6yjl3451BX3imWDnokYlg=
github.com/jackc/pgx/v5 v5.7.4/go.mod h1:ncY89UGWxg82EykZUwSpUKEfccBGGYq1xjrOpsbsfGQ=
github.com/jackc/puddle/v2 v2.2.2 h1:PR8nw+E/1w0GLuRFSmiioY6UooMp6KJv0/61nB7icHo=
github.com/jackc/puddle/v2 v2.2.2/go.mod h1:vriiEXHvEE654aYKXXjOvZM39qJ0q+azkZFrfEOc3H4=
github.com/pmezard/go-difflib v1.0.0 h1:4DBwDE0NGyQoBHbLQYPwSUPoCMWR5BEzIk/f1lZbAQM=
github.com/pmezard/go-difflib v1.0.0/go.mod h1:iKH77koFhYxTK1pcRnkKkqfTogsbg7gZNVY4sRDYZ/4=
github.com/stretchr/objx v0.1.0/go.mod h1:HFkY916IF+rwdDfMAkV7OtwuqBVzrE8GR6GFx+wExME=
github.com/stretchr/testify v1.3.0/go.mod h1:M5WIy9Dh21IEIfnGCwXGc5bZfKNJtfHm1UVUgZn+9EI=
github.com/stretchr/testify v1.7.0/go.mod h1:6Fq8oRcR53rry900zMqJjRRixrwX3KX962/h/Wwjteg=
github.com/stretchr/testify v1.8.1 h1:w7B6lhMri9wdJUVmEZPGGhZzrYTPvgJArz7wNPgYKsk=
github.com/stretchr/testify v1.8.1/go.mod h1:w2LPCIKwWwSfY2zedu0+kehJoqGctiVI29o6fzry7u4=
golang.org/x/crypto v0.36.0 h1:AnAEvhDddvBdpY+uR+MyHmuZzzNqXSe/GvuDeob5L34=
golang.org/x/crypto v0.36.0/go.mod h1:Y4J0ReaxCR1IMaabaSMugxJES1EpwhBHhv2bDHklZvc=
golang.org/x/sync v0.12.0 h1:MHc5BpPuC30uJk597Ri8TV3CNZcTLu6B6z4lJy+g6Jw=
golang.org/x/sync v0.12.0/go.mod h1:1dzgHSNfp02xaA81J2MS99Qcpr2w7fw1gpm99rleRqA=
golang.org/x/text v0.23.0 h1:D71I7dUrlY+VX0gQShAThNGHFxZ13dGLBHQLVl1mJlY=
golang.org/x/text v0.23.0/go.mod h1:/BLNzu4aZCJ1+kcD0DNRotWKage4q2rGVAg4o22unh4=
gopkg.in/check.v1 v0.0.0-20161208181325-20d25e280405/go.mod h1:Co6ibVJAznAaIkqp8huTwlJQCZ016jof/cbN4VW5Yz0=
gopkg.in/yaml.v3 v3.0.0-20200313102051-9f266ea9e77c/go.mod h1:K4uyk7z7BCEPqu6E+C64Yfv1cQ7kz7rIZviUmN+EgEM=
gopkg.in/yaml.v3 v3.0.1 h1:fxVm/GzAzEWqLHuvctI91KS9hhNmmWOoWu0XTYJS7CA=
gopkg.in/yaml.v3 v3.0.1/go.mod h1:K4uyk7z7BCEPqu6E+C64Yfv1cQ7kz7rIZviUmN+EgEM=
```


## `cmd/auth-service/main.go`

```go
package main

import (
	"context"
	"errors"
	"log/slog"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"

	"github.com/example/restaurant-auth-service/internal/config"
	"github.com/example/restaurant-auth-service/internal/httpapi"
	"github.com/example/restaurant-auth-service/internal/service"
	"github.com/example/restaurant-auth-service/internal/store"
	"github.com/example/restaurant-auth-service/internal/token"
	"github.com/example/restaurant-auth-service/migrations"
)

func main() {
	log := slog.New(slog.NewJSONHandler(os.Stdout, nil))
	ctx := context.Background()
	cfg, err := config.Load()
	if err != nil {
		log.Error("configuration error", "error", err)
		os.Exit(1)
	}
	db, err := store.Open(ctx, cfg.DatabaseURL)
	if err != nil {
		log.Error("database connection failed", "error", err)
		os.Exit(1)
	}
	defer db.Close()
	if err := migrations.Apply(ctx, db.Pool); err != nil {
		log.Error("database migration failed", "error", err)
		os.Exit(1)
	}
	tokens, err := token.New(cfg.PrivateKeyPath, cfg.PublicKeyPath, cfg.Issuer, cfg.Audience, cfg.KeyID, cfg.AccessTTL)
	if err != nil {
		log.Error("JWT initialization failed", "error", err)
		os.Exit(1)
	}
	svc := service.New(db, tokens, cfg.RefreshTTL)
	server := &http.Server{Addr: ":" + cfg.HTTPPort, Handler: httpapi.New(svc, db, log), ReadHeaderTimeout: 5 * time.Second, ReadTimeout: 15 * time.Second, WriteTimeout: 15 * time.Second, IdleTimeout: 60 * time.Second}
	go func() {
		log.Info("auth service started", "address", server.Addr)
		if err := server.ListenAndServe(); err != nil && !errors.Is(err, http.ErrServerClosed) {
			log.Error("HTTP server failed", "error", err)
			os.Exit(1)
		}
	}()
	stop := make(chan os.Signal, 1)
	signal.Notify(stop, syscall.SIGINT, syscall.SIGTERM)
	<-stop
	shutdown, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	defer cancel()
	if err := server.Shutdown(shutdown); err != nil {
		log.Error("graceful shutdown failed", "error", err)
	}
}
```


## `internal/config/config.go`

```go
package config

import (
	"fmt"
	"os"
	"time"
)

type Config struct {
	HTTPPort       string
	DatabaseURL    string
	Issuer         string
	Audience       string
	AccessTTL      time.Duration
	RefreshTTL     time.Duration
	KeyID          string
	PrivateKeyPath string
	PublicKeyPath  string
}

func Load() (Config, error) {
	accessTTL, err := duration("JWT_ACCESS_TTL", "15m")
	if err != nil {
		return Config{}, err
	}
	refreshTTL, err := duration("REFRESH_TOKEN_TTL", "720h")
	if err != nil {
		return Config{}, err
	}
	return Config{
		HTTPPort:       env("HTTP_PORT", "8081"),
		DatabaseURL:    env("DATABASE_URL", "postgres://restaurant_auth:restaurant_auth@localhost:5432/restaurant_auth?sslmode=disable"),
		Issuer:         env("JWT_ISSUER", "restaurant-auth-service"),
		Audience:       env("JWT_AUDIENCE", "restaurant-api"),
		AccessTTL:      accessTTL,
		RefreshTTL:     refreshTTL,
		KeyID:          env("JWT_KEY_ID", "local-dev-key"),
		PrivateKeyPath: env("JWT_PRIVATE_KEY_PATH", "./config/keys/private.pem"),
		PublicKeyPath:  env("JWT_PUBLIC_KEY_PATH", "./config/keys/public.pem"),
	}, nil
}

func env(key, fallback string) string {
	if value := os.Getenv(key); value != "" {
		return value
	}
	return fallback
}

func duration(key, fallback string) (time.Duration, error) {
	value := env(key, fallback)
	d, err := time.ParseDuration(value)
	if err != nil {
		return 0, fmt.Errorf("parse %s=%q: %w", key, value, err)
	}
	return d, nil
}
```


## `internal/domain/domain.go`

```go
package domain

import (
	"errors"
	"time"

	"github.com/google/uuid"
)

type UserStatus string

const (
	StatusActive              UserStatus = "ACTIVE"
	StatusBlocked             UserStatus = "BLOCKED"
	StatusDeleted             UserStatus = "DELETED"
	StatusPendingVerification UserStatus = "PENDING_VERIFICATION"
)

type UserRole string

const (
	RoleCustomer UserRole = "CUSTOMER"
	RoleAdmin    UserRole = "ADMIN"
	RoleCourier  UserRole = "COURIER"
	RoleKitchen  UserRole = "KITCHEN"
)

type User struct {
	ID           uuid.UUID
	Phone        string
	PasswordHash string
	Status       UserStatus
	Roles        []UserRole
	CreatedAt    time.Time
	UpdatedAt    time.Time
	LastLoginAt  *time.Time
}

func (u User) Active() bool { return u.Status == StatusActive }

type RefreshToken struct {
	ID        uuid.UUID
	UserID    uuid.UUID
	TokenHash string
	ExpiresAt time.Time
	RevokedAt *time.Time
	CreatedAt time.Time
	DeviceID  *uuid.UUID
}

var (
	ErrDuplicatePhone      = errors.New("Phone is already registered")
	ErrInvalidCredentials  = errors.New("Invalid phone or password")
	ErrInvalidAccessToken  = errors.New("Invalid access token")
	ErrInvalidRefreshToken = errors.New("Invalid refresh token")
	ErrExpiredRefreshToken = errors.New("Refresh token expired")
	ErrUserNotActive       = errors.New("User is not active")
	ErrNotFound            = errors.New("not found")
)
```


## `internal/token/jwt.go`

```go
package token

import (
	"crypto/rsa"
	"encoding/base64"
	"errors"
	"fmt"
	"math/big"
	"os"
	"sort"
	"time"

	"github.com/example/restaurant-auth-service/internal/domain"
	"github.com/golang-jwt/jwt/v5"
	"github.com/google/uuid"
)

type Manager struct {
	privateKey *rsa.PrivateKey
	publicKey  *rsa.PublicKey
	issuer     string
	audience   string
	keyID      string
	accessTTL  time.Duration
	now        func() time.Time
}

type AccessClaims struct {
	Phone string            `json:"phone"`
	Roles []domain.UserRole `json:"roles"`
	jwt.RegisteredClaims
}

func New(privatePath, publicPath, issuer, audience, keyID string, ttl time.Duration) (*Manager, error) {
	privatePEM, err := os.ReadFile(privatePath)
	if err != nil {
		return nil, fmt.Errorf("read JWT private key: %w", err)
	}
	privateKey, err := jwt.ParseRSAPrivateKeyFromPEM(privatePEM)
	if err != nil {
		return nil, fmt.Errorf("parse JWT private key: %w", err)
	}
	publicPEM, err := os.ReadFile(publicPath)
	if err != nil {
		return nil, fmt.Errorf("read JWT public key: %w", err)
	}
	publicKey, err := jwt.ParseRSAPublicKeyFromPEM(publicPEM)
	if err != nil {
		return nil, fmt.Errorf("parse JWT public key: %w", err)
	}
	if privateKey.PublicKey.N.Cmp(publicKey.N) != 0 || privateKey.PublicKey.E != publicKey.E {
		return nil, errors.New("JWT private and public keys do not match")
	}
	return &Manager{privateKey: privateKey, publicKey: publicKey, issuer: issuer, audience: audience, keyID: keyID, accessTTL: ttl, now: time.Now}, nil
}

func (m *Manager) Issue(user domain.User) (string, error) {
	now := m.now().UTC()
	roles := append([]domain.UserRole(nil), user.Roles...)
	sort.Slice(roles, func(i, j int) bool { return roles[i] < roles[j] })
	claims := AccessClaims{Phone: user.Phone, Roles: roles, RegisteredClaims: jwt.RegisteredClaims{
		Issuer: m.issuer, Subject: user.ID.String(), Audience: jwt.ClaimStrings{m.audience},
		ExpiresAt: jwt.NewNumericDate(now.Add(m.accessTTL)), IssuedAt: jwt.NewNumericDate(now), ID: uuid.NewString(),
	}}
	t := jwt.NewWithClaims(jwt.SigningMethodRS256, claims)
	t.Header["kid"] = m.keyID
	return t.SignedString(m.privateKey)
}

func (m *Manager) Validate(raw string) (uuid.UUID, error) {
	claims := &AccessClaims{}
	t, err := jwt.ParseWithClaims(raw, claims, func(t *jwt.Token) (any, error) {
		if t.Method != jwt.SigningMethodRS256 {
			return nil, errors.New("unexpected signing method")
		}
		return m.publicKey, nil
	}, jwt.WithIssuer(m.issuer), jwt.WithAudience(m.audience), jwt.WithExpirationRequired(), jwt.WithIssuedAt(), jwt.WithValidMethods([]string{"RS256"}))
	if err != nil || !t.Valid || claims.Phone == "" || len(claims.Roles) == 0 {
		return uuid.Nil, domain.ErrInvalidAccessToken
	}
	id, err := uuid.Parse(claims.Subject)
	if err != nil {
		return uuid.Nil, domain.ErrInvalidAccessToken
	}
	for _, role := range claims.Roles {
		switch role {
		case domain.RoleCustomer, domain.RoleAdmin, domain.RoleCourier, domain.RoleKitchen:
		default:
			return uuid.Nil, domain.ErrInvalidAccessToken
		}
	}
	return id, nil
}

func (m *Manager) ExpiresIn() int64 { return int64(m.accessTTL.Seconds()) }

func (m *Manager) JWKS() map[string]any {
	e := big.NewInt(int64(m.publicKey.E)).Bytes()
	return map[string]any{"keys": []map[string]string{{
		"kty": "RSA", "e": base64.RawURLEncoding.EncodeToString(e), "use": "sig", "kid": m.keyID,
		"alg": "RS256", "n": base64.RawURLEncoding.EncodeToString(m.publicKey.N.Bytes()),
	}}}
}
```


## `internal/token/jwt_test.go`

```go
package token

import (
	"crypto/rand"
	"crypto/rsa"
	"crypto/x509"
	"encoding/pem"
	"os"
	"path/filepath"
	"testing"
	"time"

	"github.com/example/restaurant-auth-service/internal/domain"
	"github.com/google/uuid"
)

func TestIssueValidateAndJWKS(t *testing.T) {
	dir := t.TempDir()
	privatePath := filepath.Join(dir, "private.pem")
	publicPath := filepath.Join(dir, "public.pem")
	key, err := rsa.GenerateKey(rand.Reader, 2048)
	if err != nil {
		t.Fatal(err)
	}
	privatePEM := pem.EncodeToMemory(&pem.Block{Type: "PRIVATE KEY", Bytes: x509.MarshalPKCS1PrivateKey(key)})
	publicDER, err := x509.MarshalPKIXPublicKey(&key.PublicKey)
	if err != nil {
		t.Fatal(err)
	}
	publicPEM := pem.EncodeToMemory(&pem.Block{Type: "PUBLIC KEY", Bytes: publicDER})
	if err := os.WriteFile(privatePath, privatePEM, 0600); err != nil {
		t.Fatal(err)
	}
	if err := os.WriteFile(publicPath, publicPEM, 0600); err != nil {
		t.Fatal(err)
	}
	m, err := New(privatePath, publicPath, "restaurant-auth-service", "restaurant-api", "test-key", 15*time.Minute)
	if err != nil {
		t.Fatal(err)
	}
	id := uuid.New()
	raw, err := m.Issue(domain.User{ID: id, Phone: "+79990000000", Roles: []domain.UserRole{domain.RoleCustomer}})
	if err != nil {
		t.Fatal(err)
	}
	got, err := m.Validate(raw)
	if err != nil {
		t.Fatal(err)
	}
	if got != id {
		t.Fatalf("id=%s, want %s", got, id)
	}
	keys := m.JWKS()["keys"].([]map[string]string)
	if len(keys) != 1 || keys[0]["kid"] != "test-key" || keys[0]["n"] == "" || keys[0]["e"] == "" {
		t.Fatalf("unexpected JWKS: %#v", keys)
	}
	if _, ok := keys[0]["d"]; ok {
		t.Fatal("private exponent leaked")
	}
}

func TestRejectsWrongAudience(t *testing.T) {
	dir := t.TempDir()
	key, _ := rsa.GenerateKey(rand.Reader, 2048)
	privatePath := filepath.Join(dir, "private.pem")
	publicPath := filepath.Join(dir, "public.pem")
	_ = os.WriteFile(privatePath, pem.EncodeToMemory(&pem.Block{Type: "RSA PRIVATE KEY", Bytes: x509.MarshalPKCS1PrivateKey(key)}), 0600)
	publicDER, _ := x509.MarshalPKIXPublicKey(&key.PublicKey)
	_ = os.WriteFile(publicPath, pem.EncodeToMemory(&pem.Block{Type: "PUBLIC KEY", Bytes: publicDER}), 0600)
	issuer, _ := New(privatePath, publicPath, "issuer", "audience-a", "key", time.Minute)
	verifier, _ := New(privatePath, publicPath, "issuer", "audience-b", "key", time.Minute)
	raw, _ := issuer.Issue(domain.User{ID: uuid.New(), Phone: "+79990000000", Roles: []domain.UserRole{domain.RoleCustomer}})
	if _, err := verifier.Validate(raw); err == nil {
		t.Fatal("expected wrong audience to be rejected")
	}
}
```


## `internal/store/postgres.go`

```go
package store

import (
	"context"
	"errors"
	"fmt"
	"time"

	"github.com/example/restaurant-auth-service/internal/domain"
	"github.com/google/uuid"
	"github.com/jackc/pgx/v5"
	"github.com/jackc/pgx/v5/pgxpool"
)

type Postgres struct{ Pool *pgxpool.Pool }

func Open(ctx context.Context, url string) (*Postgres, error) {
	config, err := pgxpool.ParseConfig(url)
	if err != nil {
		return nil, fmt.Errorf("parse database URL: %w", err)
	}
	config.MaxConns, config.MinConns, config.MaxConnLifetime = 20, 2, time.Hour
	pool, err := pgxpool.NewWithConfig(ctx, config)
	if err != nil {
		return nil, fmt.Errorf("open database: %w", err)
	}
	if err := pool.Ping(ctx); err != nil {
		pool.Close()
		return nil, fmt.Errorf("ping database: %w", err)
	}
	return &Postgres{Pool: pool}, nil
}

func (s *Postgres) Close()                                    { s.Pool.Close() }
func (s *Postgres) Ping(ctx context.Context) error            { return s.Pool.Ping(ctx) }
func (s *Postgres) Begin(ctx context.Context) (pgx.Tx, error) { return s.Pool.Begin(ctx) }

func (s *Postgres) PhoneExists(ctx context.Context, q pgx.Tx, phone string) (bool, error) {
	var exists bool
	err := q.QueryRow(ctx, `SELECT EXISTS(SELECT 1 FROM users WHERE phone=$1)`, phone).Scan(&exists)
	return exists, err
}

func (s *Postgres) InsertUser(ctx context.Context, q pgx.Tx, user domain.User) error {
	_, err := q.Exec(ctx, `INSERT INTO users(id,phone,password_hash,status,created_at,updated_at,last_login_at) VALUES($1,$2,$3,$4,$5,$6,$7)`, user.ID, user.Phone, user.PasswordHash, user.Status, user.CreatedAt, user.UpdatedAt, user.LastLoginAt)
	if err != nil {
		return err
	}
	for _, role := range user.Roles {
		if _, err = q.Exec(ctx, `INSERT INTO user_roles(user_id,role) VALUES($1,$2)`, user.ID, role); err != nil {
			return err
		}
	}
	return nil
}

func (s *Postgres) UserByPhone(ctx context.Context, q pgx.Tx, phone string) (domain.User, error) {
	return userBy(ctx, q, `SELECT id,phone,password_hash,status,created_at,updated_at,last_login_at FROM users WHERE phone=$1`, phone)
}

func (s *Postgres) UserByID(ctx context.Context, id uuid.UUID) (domain.User, error) {
	return userBy(ctx, s.Pool, `SELECT id,phone,password_hash,status,created_at,updated_at,last_login_at FROM users WHERE id=$1`, id)
}

type querier interface {
	Query(context.Context, string, ...any) (pgx.Rows, error)
	QueryRow(context.Context, string, ...any) pgx.Row
}

func userBy(ctx context.Context, q querier, query string, arg any) (domain.User, error) {
	var u domain.User
	err := q.QueryRow(ctx, query, arg).Scan(&u.ID, &u.Phone, &u.PasswordHash, &u.Status, &u.CreatedAt, &u.UpdatedAt, &u.LastLoginAt)
	if errors.Is(err, pgx.ErrNoRows) {
		return domain.User{}, domain.ErrNotFound
	}
	if err != nil {
		return domain.User{}, err
	}
	rows, err := q.Query(ctx, `SELECT role FROM user_roles WHERE user_id=$1 ORDER BY role`, u.ID)
	if err != nil {
		return domain.User{}, err
	}
	defer rows.Close()
	for rows.Next() {
		var role domain.UserRole
		if err := rows.Scan(&role); err != nil {
			return domain.User{}, err
		}
		u.Roles = append(u.Roles, role)
	}
	return u, rows.Err()
}

func (s *Postgres) UpdateLogin(ctx context.Context, q pgx.Tx, id uuid.UUID, now time.Time) error {
	_, err := q.Exec(ctx, `UPDATE users SET last_login_at=$2,updated_at=$2 WHERE id=$1`, id, now)
	return err
}

func (s *Postgres) InsertRefresh(ctx context.Context, q pgx.Tx, t domain.RefreshToken) error {
	_, err := q.Exec(ctx, `INSERT INTO refresh_tokens(id,user_id,token_hash,expires_at,revoked_at,created_at,device_id) VALUES($1,$2,$3,$4,$5,$6,$7)`, t.ID, t.UserID, t.TokenHash, t.ExpiresAt, t.RevokedAt, t.CreatedAt, t.DeviceID)
	return err
}

func (s *Postgres) RefreshForUpdate(ctx context.Context, q pgx.Tx, hash string) (domain.RefreshToken, domain.User, error) {
	var t domain.RefreshToken
	err := q.QueryRow(ctx, `SELECT id,user_id,token_hash,expires_at,revoked_at,created_at,device_id FROM refresh_tokens WHERE token_hash=$1 FOR UPDATE`, hash).Scan(&t.ID, &t.UserID, &t.TokenHash, &t.ExpiresAt, &t.RevokedAt, &t.CreatedAt, &t.DeviceID)
	if errors.Is(err, pgx.ErrNoRows) {
		return t, domain.User{}, domain.ErrNotFound
	}
	if err != nil {
		return t, domain.User{}, err
	}
	u, err := userBy(ctx, q, `SELECT id,phone,password_hash,status,created_at,updated_at,last_login_at FROM users WHERE id=$1`, t.UserID)
	return t, u, err
}

func (s *Postgres) RevokeRefresh(ctx context.Context, q pgx.Tx, id uuid.UUID, now time.Time) error {
	_, err := q.Exec(ctx, `UPDATE refresh_tokens SET revoked_at=$2 WHERE id=$1`, id, now)
	return err
}
```


## `internal/service/service.go`

```go
package service

import (
	"context"
	"crypto/rand"
	"crypto/sha256"
	"encoding/base64"
	"errors"
	"time"

	"github.com/example/restaurant-auth-service/internal/domain"
	"github.com/example/restaurant-auth-service/internal/store"
	"github.com/example/restaurant-auth-service/internal/token"
	"github.com/google/uuid"
	"github.com/jackc/pgx/v5"
	"golang.org/x/crypto/bcrypt"
)

type Service struct {
	store      *store.Postgres
	tokens     *token.Manager
	refreshTTL time.Duration
	now        func() time.Time
}
type TokenPair struct {
	AccessToken  string `json:"accessToken"`
	RefreshToken string `json:"refreshToken"`
	TokenType    string `json:"tokenType"`
	ExpiresIn    int64  `json:"expiresIn"`
}

func New(st *store.Postgres, tokens *token.Manager, refreshTTL time.Duration) *Service {
	return &Service{store: st, tokens: tokens, refreshTTL: refreshTTL, now: time.Now}
}

func (s *Service) Register(ctx context.Context, phone, password string) (domain.User, error) {
	tx, err := s.store.Begin(ctx)
	if err != nil {
		return domain.User{}, err
	}
	defer tx.Rollback(ctx)
	exists, err := s.store.PhoneExists(ctx, tx, phone)
	if err != nil {
		return domain.User{}, err
	}
	if exists {
		return domain.User{}, domain.ErrDuplicatePhone
	}
	hash, err := bcrypt.GenerateFromPassword([]byte(password), 12)
	if err != nil {
		return domain.User{}, err
	}
	now := s.now().UTC()
	u := domain.User{ID: uuid.New(), Phone: phone, PasswordHash: string(hash), Status: domain.StatusActive, Roles: []domain.UserRole{domain.RoleCustomer}, CreatedAt: now, UpdatedAt: now}
	if err := s.store.InsertUser(ctx, tx, u); err != nil {
		return domain.User{}, err
	}
	if err := tx.Commit(ctx); err != nil {
		return domain.User{}, err
	}
	return u, nil
}

func (s *Service) Login(ctx context.Context, phone, password string) (TokenPair, error) {
	tx, err := s.store.Begin(ctx)
	if err != nil {
		return TokenPair{}, err
	}
	defer tx.Rollback(ctx)
	u, err := s.store.UserByPhone(ctx, tx, phone)
	if errors.Is(err, domain.ErrNotFound) {
		return TokenPair{}, domain.ErrInvalidCredentials
	}
	if err != nil {
		return TokenPair{}, err
	}
	if bcrypt.CompareHashAndPassword([]byte(u.PasswordHash), []byte(password)) != nil {
		return TokenPair{}, domain.ErrInvalidCredentials
	}
	if !u.Active() {
		return TokenPair{}, domain.ErrUserNotActive
	}
	now := s.now().UTC()
	if err := s.store.UpdateLogin(ctx, tx, u.ID, now); err != nil {
		return TokenPair{}, err
	}
	pair, err := s.issue(ctx, tx, u, now)
	if err != nil {
		return TokenPair{}, err
	}
	if err := tx.Commit(ctx); err != nil {
		return TokenPair{}, err
	}
	return pair, nil
}

func (s *Service) Refresh(ctx context.Context, plain string) (TokenPair, error) {
	tx, err := s.store.Begin(ctx)
	if err != nil {
		return TokenPair{}, err
	}
	defer tx.Rollback(ctx)
	now := s.now().UTC()
	current, u, err := s.validRefresh(ctx, tx, plain, now)
	if err != nil {
		return TokenPair{}, err
	}
	if err := s.store.RevokeRefresh(ctx, tx, current.ID, now); err != nil {
		return TokenPair{}, err
	}
	pair, err := s.issue(ctx, tx, u, now)
	if err != nil {
		return TokenPair{}, err
	}
	if err := tx.Commit(ctx); err != nil {
		return TokenPair{}, err
	}
	return pair, nil
}

func (s *Service) Logout(ctx context.Context, plain string) error {
	tx, err := s.store.Begin(ctx)
	if err != nil {
		return err
	}
	defer tx.Rollback(ctx)
	now := s.now().UTC()
	current, _, err := s.validRefresh(ctx, tx, plain, now)
	if err != nil {
		return err
	}
	if err := s.store.RevokeRefresh(ctx, tx, current.ID, now); err != nil {
		return err
	}
	return tx.Commit(ctx)
}

func (s *Service) Me(ctx context.Context, id uuid.UUID) (domain.User, error) {
	u, err := s.store.UserByID(ctx, id)
	if errors.Is(err, domain.ErrNotFound) {
		return domain.User{}, domain.ErrInvalidAccessToken
	}
	if err != nil {
		return domain.User{}, err
	}
	if !u.Active() {
		return domain.User{}, domain.ErrUserNotActive
	}
	return u, nil
}

func (s *Service) Introspect(ctx context.Context, raw string) (domain.User, bool) {
	id, err := s.tokens.Validate(raw)
	if err != nil {
		return domain.User{}, false
	}
	u, err := s.store.UserByID(ctx, id)
	if err != nil || !u.Active() {
		return domain.User{}, false
	}
	return u, true
}
func (s *Service) ValidateAccess(raw string) (uuid.UUID, error) { return s.tokens.Validate(raw) }
func (s *Service) JWKS() map[string]any                         { return s.tokens.JWKS() }

func (s *Service) validRefresh(ctx context.Context, tx pgx.Tx, plain string, now time.Time) (domain.RefreshToken, domain.User, error) {
	t, u, err := s.store.RefreshForUpdate(ctx, tx, refreshHash(plain))
	if errors.Is(err, domain.ErrNotFound) {
		return t, u, domain.ErrInvalidRefreshToken
	}
	if err != nil {
		return t, u, err
	}
	if t.RevokedAt != nil {
		return t, u, domain.ErrInvalidRefreshToken
	}
	if !t.ExpiresAt.After(now) {
		return t, u, domain.ErrExpiredRefreshToken
	}
	if !u.Active() {
		return t, u, domain.ErrUserNotActive
	}
	return t, u, nil
}

func (s *Service) issue(ctx context.Context, tx pgx.Tx, u domain.User, now time.Time) (TokenPair, error) {
	access, err := s.tokens.Issue(u)
	if err != nil {
		return TokenPair{}, err
	}
	bytes := make([]byte, 64)
	if _, err := rand.Read(bytes); err != nil {
		return TokenPair{}, err
	}
	plain := base64.RawURLEncoding.EncodeToString(bytes)
	t := domain.RefreshToken{ID: uuid.New(), UserID: u.ID, TokenHash: refreshHash(plain), ExpiresAt: now.Add(s.refreshTTL), CreatedAt: now}
	if err := s.store.InsertRefresh(ctx, tx, t); err != nil {
		return TokenPair{}, err
	}
	return TokenPair{AccessToken: access, RefreshToken: plain, TokenType: "Bearer", ExpiresIn: s.tokens.ExpiresIn()}, nil
}

func refreshHash(plain string) string {
	sum := sha256.Sum256([]byte(plain))
	return base64.RawURLEncoding.EncodeToString(sum[:])
}
```


## `internal/httpapi/api.go`

```go
package httpapi

import (
	"context"
	"encoding/json"
	"errors"
	"io"
	"log/slog"
	"net/http"
	"regexp"
	"strings"
	"time"
	"unicode/utf8"

	"github.com/example/restaurant-auth-service/internal/domain"
	"github.com/example/restaurant-auth-service/internal/service"
	"github.com/example/restaurant-auth-service/internal/store"
	"github.com/google/uuid"
)

type API struct {
	service *service.Service
	store   *store.Postgres
	log     *slog.Logger
}
type ctxKey string

const userIDKey ctxKey = "userID"

var phonePattern = regexp.MustCompile(`^\+[1-9][0-9]{7,14}$`)

func New(svc *service.Service, st *store.Postgres, log *slog.Logger) http.Handler {
	a := &API{service: svc, store: st, log: log}
	mux := http.NewServeMux()
	mux.HandleFunc("POST /auth/register", a.register)
	mux.HandleFunc("POST /auth/login", a.login)
	mux.HandleFunc("POST /auth/refresh", a.refresh)
	mux.HandleFunc("POST /auth/logout", a.logout)
	mux.Handle("GET /auth/me", a.authenticate(http.HandlerFunc(a.me)))
	mux.HandleFunc("POST /auth/introspect", a.introspect)
	mux.HandleFunc("GET /auth/.well-known/jwks.json", a.jwks)
	mux.HandleFunc("GET /actuator/health", a.health)
	return a.recover(a.jsonContent(mux))
}

type credentials struct {
	Phone    string `json:"phone"`
	Password string `json:"password"`
}
type refreshRequest struct {
	RefreshToken string `json:"refreshToken"`
}
type introspectRequest struct {
	Token string `json:"token"`
}
type errorResponse struct {
	Timestamp time.Time `json:"timestamp"`
	Status    int       `json:"status"`
	Error     string    `json:"error"`
	Message   string    `json:"message"`
	Path      string    `json:"path"`
}

func (a *API) register(w http.ResponseWriter, r *http.Request) {
	var req credentials
	if !a.decode(w, r, &req) {
		return
	}
	if msg := validateCredentials(req); msg != "" {
		a.problem(w, r, 400, msg)
		return
	}
	u, err := a.service.Register(r.Context(), strings.TrimSpace(req.Phone), req.Password)
	if err != nil {
		a.handle(w, r, err)
		return
	}
	a.write(w, 201, map[string]any{"userId": u.ID, "phone": u.Phone, "status": u.Status})
}
func (a *API) login(w http.ResponseWriter, r *http.Request) {
	var req credentials
	if !a.decode(w, r, &req) {
		return
	}
	if msg := validateCredentials(req); msg != "" {
		a.problem(w, r, 400, msg)
		return
	}
	pair, err := a.service.Login(r.Context(), strings.TrimSpace(req.Phone), req.Password)
	if err != nil {
		a.handle(w, r, err)
		return
	}
	a.write(w, 200, pair)
}
func (a *API) refresh(w http.ResponseWriter, r *http.Request) {
	var req refreshRequest
	if !a.decode(w, r, &req) {
		return
	}
	if strings.TrimSpace(req.RefreshToken) == "" {
		a.problem(w, r, 400, "Validation failed: refreshToken must not be blank")
		return
	}
	pair, err := a.service.Refresh(r.Context(), req.RefreshToken)
	if err != nil {
		a.handle(w, r, err)
		return
	}
	a.write(w, 200, pair)
}
func (a *API) logout(w http.ResponseWriter, r *http.Request) {
	var req refreshRequest
	if !a.decode(w, r, &req) {
		return
	}
	if strings.TrimSpace(req.RefreshToken) == "" {
		a.problem(w, r, 400, "Validation failed: refreshToken must not be blank")
		return
	}
	if err := a.service.Logout(r.Context(), req.RefreshToken); err != nil {
		a.handle(w, r, err)
		return
	}
	a.write(w, 200, map[string]string{"message": "Logged out successfully"})
}
func (a *API) me(w http.ResponseWriter, r *http.Request) {
	id, _ := r.Context().Value(userIDKey).(uuid.UUID)
	u, err := a.service.Me(r.Context(), id)
	if err != nil {
		a.handle(w, r, err)
		return
	}
	a.write(w, 200, map[string]any{"userId": u.ID, "phone": u.Phone, "roles": u.Roles, "status": u.Status})
}
func (a *API) introspect(w http.ResponseWriter, r *http.Request) {
	var req introspectRequest
	if !a.decode(w, r, &req) {
		return
	}
	if strings.TrimSpace(req.Token) == "" {
		a.problem(w, r, 400, "Validation failed: token must not be blank")
		return
	}
	u, active := a.service.Introspect(r.Context(), req.Token)
	if !active {
		a.write(w, 200, map[string]any{"active": false})
		return
	}
	a.write(w, 200, map[string]any{"active": true, "userId": u.ID, "phone": u.Phone, "roles": u.Roles})
}
func (a *API) jwks(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Cache-Control", "no-store")
	a.write(w, 200, a.service.JWKS())
}
func (a *API) health(w http.ResponseWriter, r *http.Request) {
	if err := a.store.Ping(r.Context()); err != nil {
		a.write(w, 503, map[string]string{"status": "DOWN"})
		return
	}
	a.write(w, 200, map[string]string{"status": "UP"})
}

func (a *API) authenticate(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		header := r.Header.Get("Authorization")
		if !strings.HasPrefix(header, "Bearer ") || strings.TrimSpace(strings.TrimPrefix(header, "Bearer ")) == "" {
			a.problem(w, r, 401, "Invalid access token")
			return
		}
		id, err := a.service.ValidateAccess(strings.TrimSpace(strings.TrimPrefix(header, "Bearer ")))
		if err != nil {
			a.problem(w, r, 401, "Invalid access token")
			return
		}
		u, err := a.service.Me(r.Context(), id)
		if err != nil {
			a.handle(w, r, err)
			return
		}
		next.ServeHTTP(w, r.WithContext(context.WithValue(r.Context(), userIDKey, u.ID)))
	})
}
func validateCredentials(req credentials) string {
	if !phonePattern.MatchString(req.Phone) {
		return "Validation failed: phone must be a valid E.164 number"
	}
	n := utf8.RuneCountInString(req.Password)
	if strings.TrimSpace(req.Password) == "" || n < 6 || n > 72 {
		return "Validation failed: password size must be between 6 and 72"
	}
	return ""
}

func (a *API) decode(w http.ResponseWriter, r *http.Request, dst any) bool {
	dec := json.NewDecoder(http.MaxBytesReader(w, r.Body, 1<<20))
	if err := dec.Decode(dst); err != nil {
		a.problem(w, r, 400, "Malformed JSON request")
		return false
	}
	var extra any
	if err := dec.Decode(&extra); !errors.Is(err, io.EOF) {
		a.problem(w, r, 400, "Malformed JSON request")
		return false
	}
	return true
}
func (a *API) write(w http.ResponseWriter, status int, value any) {
	w.Header().Set("Content-Type", "application/json;charset=UTF-8")
	w.WriteHeader(status)
	_ = json.NewEncoder(w).Encode(value)
}
func (a *API) problem(w http.ResponseWriter, r *http.Request, status int, message string) {
	a.write(w, status, errorResponse{Timestamp: time.Now().UTC(), Status: status, Error: http.StatusText(status), Message: message, Path: r.URL.Path})
}
func (a *API) handle(w http.ResponseWriter, r *http.Request, err error) {
	switch {
	case errors.Is(err, domain.ErrDuplicatePhone):
		a.problem(w, r, 409, err.Error())
	case errors.Is(err, domain.ErrInvalidCredentials), errors.Is(err, domain.ErrInvalidAccessToken), errors.Is(err, domain.ErrInvalidRefreshToken), errors.Is(err, domain.ErrExpiredRefreshToken):
		a.problem(w, r, 401, err.Error())
	case errors.Is(err, domain.ErrUserNotActive):
		a.problem(w, r, 403, err.Error())
	default:
		a.log.Error("request failed", "method", r.Method, "path", r.URL.Path, "error", err)
		a.problem(w, r, 500, "Internal server error")
	}
}
func (a *API) recover(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		defer func() {
			if value := recover(); value != nil {
				a.log.Error("panic", "value", value)
				a.problem(w, r, 500, "Internal server error")
			}
		}()
		next.ServeHTTP(w, r)
	})
}
func (a *API) jsonContent(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) { next.ServeHTTP(w, r) })
}
```


## `migrations/embed.go`

```go
package migrations

import (
	"context"
	"embed"
	"fmt"
	"io/fs"
	"sort"
	"strconv"
	"strings"

	"github.com/jackc/pgx/v5/pgxpool"
)

//go:embed *.sql
var files embed.FS

func Apply(ctx context.Context, pool *pgxpool.Pool) error {
	conn, err := pool.Acquire(ctx)
	if err != nil {
		return err
	}
	defer conn.Release()
	if _, err := conn.Exec(ctx, `SELECT pg_advisory_lock(8142026)`); err != nil {
		return err
	}
	defer conn.Exec(context.Background(), `SELECT pg_advisory_unlock(8142026)`)
	if _, err := conn.Exec(ctx, `CREATE TABLE IF NOT EXISTS schema_migrations(version BIGINT PRIMARY KEY, applied_at TIMESTAMPTZ NOT NULL DEFAULT now())`); err != nil {
		return err
	}
	entries, err := fs.ReadDir(files, ".")
	if err != nil {
		return err
	}
	sort.Slice(entries, func(i, j int) bool { return entries[i].Name() < entries[j].Name() })
	for _, entry := range entries {
		if entry.IsDir() || !strings.HasSuffix(entry.Name(), ".sql") {
			continue
		}
		version, err := strconv.ParseInt(strings.SplitN(entry.Name(), "_", 2)[0], 10, 64)
		if err != nil {
			return fmt.Errorf("invalid migration %s", entry.Name())
		}
		var applied bool
		if err := conn.QueryRow(ctx, `SELECT EXISTS(SELECT 1 FROM schema_migrations WHERE version=$1)`, version).Scan(&applied); err != nil {
			return err
		}
		if applied {
			continue
		}
		sql, err := files.ReadFile(entry.Name())
		if err != nil {
			return err
		}
		tx, err := conn.Begin(ctx)
		if err != nil {
			return err
		}
		if _, err = tx.Exec(ctx, string(sql)); err != nil {
			tx.Rollback(ctx)
			return fmt.Errorf("apply migration %s: %w", entry.Name(), err)
		}
		if _, err = tx.Exec(ctx, `INSERT INTO schema_migrations(version) VALUES($1)`, version); err != nil {
			tx.Rollback(ctx)
			return err
		}
		if err = tx.Commit(ctx); err != nil {
			return err
		}
	}
	return nil
}
```


## `migrations/001_create_auth_schema.sql`

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    phone VARCHAR(32) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    status VARCHAR(32) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL,
    updated_at TIMESTAMPTZ NOT NULL,
    last_login_at TIMESTAMPTZ NULL,
    CONSTRAINT chk_users_status CHECK (status IN ('ACTIVE','BLOCKED','DELETED','PENDING_VERIFICATION'))
);
CREATE UNIQUE INDEX ux_users_phone ON users(phone);
CREATE TABLE user_roles (
    user_id UUID NOT NULL,
    role VARCHAR(64) NOT NULL,
    CONSTRAINT pk_user_roles PRIMARY KEY(user_id,role),
    CONSTRAINT fk_user_roles_user FOREIGN KEY(user_id) REFERENCES users(id) ON DELETE CASCADE,
    CONSTRAINT chk_user_roles_role CHECK (role IN ('CUSTOMER','ADMIN','COURIER','KITCHEN'))
);
CREATE TABLE refresh_tokens (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL,
    token_hash VARCHAR(255) NOT NULL,
    expires_at TIMESTAMPTZ NOT NULL,
    revoked_at TIMESTAMPTZ NULL,
    created_at TIMESTAMPTZ NOT NULL,
    device_id UUID NULL,
    CONSTRAINT fk_refresh_tokens_user FOREIGN KEY(user_id) REFERENCES users(id) ON DELETE CASCADE
);
CREATE INDEX ix_refresh_tokens_user_id ON refresh_tokens(user_id);
CREATE UNIQUE INDEX ux_refresh_tokens_token_hash ON refresh_tokens(token_hash);
CREATE INDEX ix_refresh_tokens_expires_at ON refresh_tokens(expires_at);
CREATE TABLE user_devices (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL,
    platform VARCHAR(32) NOT NULL,
    push_token TEXT NULL,
    created_at TIMESTAMPTZ NOT NULL,
    updated_at TIMESTAMPTZ NOT NULL,
    CONSTRAINT fk_user_devices_user FOREIGN KEY(user_id) REFERENCES users(id) ON DELETE CASCADE,
    CONSTRAINT chk_user_devices_platform CHECK (platform IN ('WEB','IOS','ANDROID'))
);
CREATE INDEX ix_user_devices_user_id ON user_devices(user_id);
```


## `migrations/002_seed_dev_users.sql`

```sql
INSERT INTO users(id,phone,password_hash,status,created_at,updated_at,last_login_at) VALUES
('00000000-0000-0000-0000-000000000001','+79990000000','$2y$12$ormoS9vxs1izKIkIgtlHaerSXx2.cNlI2lPq4hkr9qhemmEHkfhWe','ACTIVE',CURRENT_TIMESTAMP,CURRENT_TIMESTAMP,NULL),
('00000000-0000-0000-0000-000000000002','+79991111111','$2y$12$tZZ2W2COYwkA5tXeaFYdrulSL7ZwNV11xmTIZgKeWtBtO4no6d7ee','ACTIVE',CURRENT_TIMESTAMP,CURRENT_TIMESTAMP,NULL),
('00000000-0000-0000-0000-000000000003','+79992222222','$2y$12$7Ex2CvyxTYoOMslrxv6BYOIMYdhyHtcLHezuIHzEz.dQVma40mpJ.','BLOCKED',CURRENT_TIMESTAMP,CURRENT_TIMESTAMP,NULL);
INSERT INTO user_roles(user_id,role) VALUES
('00000000-0000-0000-0000-000000000001','CUSTOMER'),
('00000000-0000-0000-0000-000000000002','ADMIN'),
('00000000-0000-0000-0000-000000000003','CUSTOMER');
```


## `config/keys/public.pem`

```text
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAtHkyP/zyfXBOJK4ashit
2b4mz+rOwievPbTwjAdYv8Sgp40X8xnOuO1tbm92p+95Ymdzp0FecOOaO6KbpyVu
WmhNlIJkQxGdRTaZ2rKLeuJ4y3kAXpOxmU0Tuc4ghuSYYyKZS4u6P6kWWRjWExzN
uC3ImpEtvgFax5r7hpawHVNh1f8tmS9CDyvZMKSDteJdnOUzyw2p4Gssaal3xPzC
KXFnetXNg1B68ecIADVk4FluBgztjfU1/1Jau/B5+psfy+w15B0nB1xdVTfGopSP
qZ+zv83EOa6iBPD3aL2mU1hNoD/V3uD1VL+cNLzkCyDAjXRxg1SK95plvHNvVxGq
OwIDAQAB
-----END PUBLIC KEY-----
```


## `config/keys/private.pem`

> Это приватный RSA-ключ (PEM). Содержимое намеренно не включено в дамп исходного кода по соображениям безопасности — секреты не должны попадать в общий контекст.
> В рабочем окружении ключ генерируется командой `make keys` и монтируется в контейнер.

```text
-----BEGIN PRIVATE KEY-----
<... REDACTED — приватный ключ не публикуется ...>
-----END PRIVATE KEY-----
```


## `Dockerfile`

```dockerfile
FROM golang:1.24-alpine AS build

WORKDIR /src

COPY go.mod go.sum* ./
RUN go mod download

COPY ../../../Downloads .

RUN CGO_ENABLED=0 GOOS=linux go build -trimpath -ldflags="-s -w" -o /out/auth-service ./cmd/auth-service

FROM gcr.io/distroless/static-debian12:nonroot

WORKDIR /app

COPY --from=build /out/auth-service /app/auth-service

EXPOSE 8081

USER nonroot:nonroot

ENTRYPOINT ["/app/auth-service"]
```


## `docker-compose.yml`

```yaml
name: auth-go-local

services:
  auth-postgres:
    image: postgres:18
    environment:
      POSTGRES_DB: restaurant_auth
      POSTGRES_USER: restaurant_auth
      POSTGRES_PASSWORD: restaurant_auth
      PGDATA: /var/lib/postgresql/data/pgdata
    ports:
      - "${POSTGRES_PORT:-55432}:5432"
    volumes:
      - auth-go-local-postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U restaurant_auth -d restaurant_auth"]
      interval: 5s
      timeout: 5s
      retries: 10

  auth-service:
    build:
      context: .
      dockerfile: Dockerfile
    environment:
      HTTP_PORT: 8081
      DATABASE_URL: postgres://restaurant_auth:restaurant_auth@auth-postgres:5432/restaurant_auth?sslmode=disable
      JWT_PRIVATE_KEY_PATH: /run/keys/private.pem
      JWT_PUBLIC_KEY_PATH: /run/keys/public.pem
      JWT_ISSUER: restaurant-auth-service
      JWT_AUDIENCE: restaurant-api
      JWT_KEY_ID: local-dev-key
      JWT_ACCESS_TTL: 15m
      REFRESH_TOKEN_TTL: 720h
    ports:
      - "${AUTH_PORT:-18081}:8081"
    volumes:
      - ./config/keys:/run/keys:ro
    depends_on:
      auth-postgres:
        condition: service_healthy

volumes:
  auth-go-local-postgres-data:
```


## `Makefile`

```makefile
.PHONY: keys test run compose-up compose-down

keys:
	@mkdir -p config/keys
	@if [ ! -f config/keys/private.pem ] || [ ! -f config/keys/public.pem ]; then \
		openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out config/keys/private.pem; \
		openssl pkey -in config/keys/private.pem -pubout -out config/keys/public.pem; \
		chmod 600 config/keys/private.pem; \
	fi

test:
	go test ./...

run:
	go run ./cmd/auth-service

compose-up: keys
	docker compose up --build

compose-down:
	docker compose down
```


## `.env.example`

```ini
POSTGRES_PORT=5432
AUTH_PORT=8081

HTTP_PORT=8081
DATABASE_URL=postgres://restaurant_auth:restaurant_auth@localhost:5432/restaurant_auth?sslmode=disable

JWT_PRIVATE_KEY_PATH=./config/keys/private.pem
JWT_PUBLIC_KEY_PATH=./config/keys/public.pem
JWT_ISSUER=restaurant-auth-service
JWT_AUDIENCE=restaurant-api
JWT_KEY_ID=local-dev-key
JWT_ACCESS_TTL=15m
REFRESH_TOKEN_TTL=720h
```


## `README.md`

```markdown
# restaurant-auth-service (Go)

Поведенчески совместимый перенос Java/Spring Boot сервиса аутентификации на Go и PostgreSQL.

## Возможности

- регистрация и BCrypt-хэширование паролей;
- login с RSA/RS256 access JWT и opaque refresh-token;
- атомарная ротация и отзыв refresh-token через `SELECT FOR UPDATE`;
- `/auth/me`, introspection, публичный JWKS;
- встроенные версионированные PostgreSQL-миграции и dev seed;
- health check и graceful shutdown.

## Быстрый запуск

Нужны Docker, Docker Compose и OpenSSL:

```bash
make compose-up
```

Сервис будет доступен на `http://localhost:8081`, PostgreSQL — на `localhost:5432`. При первом запуске `make` создаст локальную RSA-пару в `config/keys`; ключи исключены из Git. Если порты заняты: `POSTGRES_PORT=55432 AUTH_PORT=18081 make compose-up`.

Проверка:

```bash
curl -s http://localhost:8081/actuator/health
curl -s -X POST http://localhost:8081/auth/login \
  -H 'Content-Type: application/json' \
  -d '{"phone":"+79990000000","password":"123456"}'
```

Dev-пользователи сохранены из Java-версии: customer `+79990000000 / 123456`, admin `+79991111111 / admin123`, blocked user `+79992222222 / blocked123`.

## Локальная разработка

Скопируйте `.env.example` в `.env` и экспортируйте переменные либо используйте значения по умолчанию. Затем:

```bash
make keys
docker compose up -d auth-postgres
go test ./...
go run ./cmd/auth-service
```

## API

| Метод | Путь | Доступ |
|---|---|---|
| POST | `/auth/register` | public |
| POST | `/auth/login` | public |
| POST | `/auth/refresh` | public |
| POST | `/auth/logout` | public |
| GET | `/auth/me` | Bearer JWT |
| POST | `/auth/introspect` | public |
| GET | `/auth/.well-known/jwks.json` | public |
| GET | `/actuator/health` | public |

Миграции применяются самим приложением при старте под PostgreSQL advisory lock. Каждая миграция выполняется транзакционно и отмечается в `schema_migrations`.
```
