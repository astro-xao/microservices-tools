---
sidebar_position: 4
---
# 日志

## gRPC 拦截器
实现 gRPC UnaryInterceptor（单次请求）和 StreamInterceptor（流式请求）来捕获日志。
```go
import (
    "context"
    "google.golang.org/grpc"
    "log"
)

// Unary Interceptor for logging
func LoggingInterceptor(
    ctx context.Context,
    req interface{},
    info *grpc.UnaryServerInfo,
    handler grpc.UnaryHandler,
) (interface{}, error) {
    log.Printf("Request - Method:%s, Request:%v", info.FullMethod, req)
    resp, err := handler(ctx, req)
    if err != nil {
        log.Printf("Error - Method:%s, Error:%v", info.FullMethod, err)
    }
    return resp, err
}

// Register the interceptor
server := grpc.NewServer(
    grpc.UnaryInterceptor(LoggingInterceptor),
)
```

## Logrus
使用 Logrus（一个流行的 Go 日志库）与 gRPC 拦截器集成，提供结构化日志。
```go
import (
    "github.com/grpc-ecosystem/go-grpc-middleware"
    "github.com/grpc-ecosystem/go-grpc-middleware/logging/logrus"
    "github.com/sirupsen/logrus"
    "google.golang.org/grpc"
)

func main() {
    logger := logrus.New()
    logrusEntry := logrus.NewEntry(logger)

    // Configure gRPC server with Logrus middleware
    server := grpc.NewServer(
        grpc_middleware.WithUnaryServerChain(
            grpc_logrus.UnaryServerInterceptor(logrusEntry),
        ),
    )
    log.Println("gRPC server with Logrus logging started")
}
```
