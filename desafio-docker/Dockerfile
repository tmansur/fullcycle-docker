FROM ubuntu AS builder

WORKDIR /app

COPY . .

RUN apt-get update && \
  apt-get -y install golang-go

RUN go build hello.go


# Imagem vazia
FROM scratch 

WORKDIR /app

COPY --from=builder /app .

CMD ["./hello"]
