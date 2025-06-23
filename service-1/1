# Build stage
FROM golang:1.20-alpine AS builder

WORKDIR /app
COPY . .

RUN go mod tidy
RUN go build -o service1 main.go

# Final image
FROM alpine
COPY --from=builder /app/service1 /service1
EXPOSE 8081
CMD ["/service1"]

