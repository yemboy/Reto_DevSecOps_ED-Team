FROM golang:1.23-alpine AS builder

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY main.go .
RUN CGO_ENABLED=0 GOOS=linux go build -o server .

FROM alpine:3.20

RUN addgroup -g 1000 appgroup && \
    adduser -u 1000 -G appgroup -D appuser

WORKDIR /app
COPY --from=builder /app/server .

USER appuser
EXPOSE 8080

CMD ["./server"]
