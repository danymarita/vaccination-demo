FROM golang:1.17-buster as build-app
WORKDIR /opt/vaccination
ENV GO111MODULE=on
ADD go.mod .
ADD go.sum .
RUN go mod download
COPY . ./
RUN go build cmd/api.go
RUN go build cmd/consumer_registration.go
RUN go build cmd/consumer_vaccination.go
RUN go build cmd/kafka.go

FROM alpine as alpine
RUN apk --no-cache add tzdata ca-certificates

FROM bitnami/minideb:latest
ARG APPLICATION_NAME=vaccination
WORKDIR /opt/$APPLICATION_NAME
COPY --from=alpine /usr/share/zoneinfo /usr/share/zoneinfo
COPY --from=alpine /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt
COPY --from=build-app /opt/$APPLICATION_NAME/api .
COPY --from=build-app /opt/$APPLICATION_NAME/consumer_registration .
COPY --from=build-app /opt/$APPLICATION_NAME/consumer_vaccination .
COPY --from=build-app /opt/$APPLICATION_NAME/kafka .
COPY --from=build-app /opt/$APPLICATION_NAME/.env.sample .

RUN cp /opt/$APPLICATION_NAME/.env.sample .env
RUN sed -i 's/^mq_host=.*$/mq_host=rabbitmq:5672/' .env
RUN sed -i 's/^db_host=.*$/db_host=postgres/' .env