# Сервис пользователей и велосипедов

## Архитектура
Система состоит из двух микросервисов:
1. **User Service** - управляет данными пользователей
2. **Bike Service** - управляет информацией о велосипедах и их компонентах

Сервисы общаются через gRPC, используя Protocol Buffers для сериализации данных.

---

## 1. Примеры Proto-структур

### bike.proto
```protobuf
syntax = "proto3";

package bike;

service BikeService {
  // Получить полную информацию о велосипеде
  rpc GetBikeDetails(BikeRequest) returns (BikeResponse);
  
  // Обновить состояние компонента
  rpc UpdateComponentStatus(ComponentStatusUpdate) returns (UpdateResponse);
}

message BikeRequest {
  string bike_id = 1;
}

message BikeResponse {
  Bike bike = 1;
}

message Bike {
  string id = 1;
  string model = 2;
  repeated Component components = 3;
}

message Component {
  string type = 1;
  string serial_number = 2;
  ComponentStatus status = 3;
  string last_maintenance = 4;
}

enum ComponentStatus {
  UNKNOWN = 0;
  GOOD = 1;
  NEEDS_MAINTENANCE = 2;
  DAMAGED = 3;
}

message ComponentStatusUpdate {
  string component_id = 1;
  ComponentStatus new_status = 2;
}

message UpdateResponse {
  bool success = 1;
  string message = 2;
}

```

Scheme

```
┌─────────────────┐       gRPC call       ┌──────────────────┐
│                 │ ────────────────────> │                  │
│   User Service  │                       │ Bicycle Service  │
│  (gRPC Client)  │ <──────────────────── │ (gRPC Server)    │
└─────────────────┘      Response         └──────────────────┘
```

## Пример workflow
1. Клиент запрашивает данные пользователя:  
   `GET /user/{id}`
2. User Service инициирует gRPC-запрос к Bicycle Service:
   - Метод: `GetBicycleDetails`
   - Bicycle Service возвращает информацию о велосипеде с деталями
3. User Service агрегирует данные и возвращает ответ клиенту

---

## Технические детали
- **Сетевые порты**:
  - Bicycle Service: `:50051`
- **Генерация кода**:
  ```bash
  protoc -I . bike_rpc/bike.proto --go_out=./github.com/TolkienRools/gRPC/bike_rpc --go_opt=paths=source_relative --go-grpc_out=./github.com/TolkienRools/gRPC/bike_rpc --go-grpc_opt=paths=source_relative


