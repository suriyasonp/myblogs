---
title: "สอง .NET Backends จะคุยกันอย่างไร? REST สำหรับ Request และ SignalR สำหรับ Realtime"
date: 2026-07-29
draft: false
slug: "dotnet-backend-communication-rest-signalr"
author: "สุริยา สนภู่"
categories:
    - Technology
    - Software Development
tags:
    - .NET
    - ASP.NET Core
    - Backend
    - REST API
    - SignalR
    - Distributed Systems
    - System Design
image: cover.png
description: "มาดูกันว่าสอง .NET backends สามารถแลกเปลี่ยนข้อมูลกันด้วย protocol อะไรบ้าง พร้อมตัวอย่าง REST สำหรับ request-response และ SignalR สำหรับ realtime"
---

เวลาเราเริ่มแยกระบบออกเป็น backend มากกว่าหนึ่งตัว สิ่งที่ตามมาทันทีคือ แล้วแต่ละระบบจะคุยกันอย่างไร?

จริงๆ แล้วมี protocol ให้เลือกหลายแบบ ทั้ง REST, gRPC, WebSocket, SignalR รวมถึง Message broker แต่ก่อนจะเลือกใช้ตัวไหน เราต้องตอบให้ได้ก่อนว่า การสื่อสารครั้งนั้นต้องการคำตอบกลับมาทันทีหรือไม่ และข้อมูลที่ส่งออกไปยอมให้หายได้หรือเปล่า

ในบทความนี้ผมจะยกตัวอย่างสองระบบ เพื่อให้เห็นภาพง่ายขึ้น:

- **Order API** รับผิดชอบการสร้างและติดตามคำสั่งซื้อ
- **Inventory API** เป็นเจ้าของข้อมูลสต็อกสินค้า

กรณีที่ Order API ต้องตรวจสอบว่าสินค้ามีพอหรือไม่ ระบบต้องส่งคำถามไปหา Inventory API และรอคำตอบกลับมา แบบนี้เรียกว่า **request-response** ซึ่ง REST เหมาะกับงานลักษณะนี้

แต่ถ้าจำนวนสินค้าเปลี่ยน แล้วเราต้องการให้ Inventory API แจ้ง Order API ทันที โดยไม่ต้องคอยยิง request ถามซ้ำทุกวินาที แบบนี้เราต้องการการสื่อสารแบบ **realtime** ซึ่ง SignalR เข้ามาช่วยตรงนี้ได้

อย่าเพิ่งคิดว่าเราต้องเลือก REST หรือ SignalR อย่างใดอย่างหนึ่ง เพราะในระบบจริงเราสามารถใช้ทั้งสองแบบร่วมกันได้ โดยให้แต่ละตัวทำงานที่ตัวเองถนัด

## ก่อนเลือก Protocol ลองดูรูปแบบการสื่อสารก่อน

การสื่อสารระหว่าง backend สามารถแบ่งรูปแบบหลักๆ ได้ดังนี้

| รูปแบบ | ผู้ส่งรอคำตอบหรือไม่ | ตัวอย่าง | ทางเลือกที่พบบ่อย |
|---|---:|---|---|
| Request-response | รอ | ตรวจสต็อก, อ่านข้อมูลลูกค้า, คำนวณราคา | HTTP/REST, gRPC, GraphQL |
| Realtime push | ไม่ต้องถามซ้ำ | แจ้ง stock changed, progress, live status | SignalR, WebSocket, Server-Sent Events |
| Asynchronous messaging | ไม่รอและต้องส่งให้ถึง | order created, payment completed | RabbitMQ, Azure Service Bus, Kafka |
| Streaming | รับข้อมูลต่อเนื่อง | telemetry, log stream, market feed | gRPC streaming, Kafka, WebSocket |

จุดที่หลายคนอาจเข้าใจผิดคือคำว่า **realtime** ไม่ได้แปลว่า **reliable** เสมอไป SignalR ส่งข้อมูลไปยัง connection ที่กำลังเชื่อมต่ออยู่ได้รวดเร็วก็จริง แต่ถ้าผู้รับ offline อยู่ในเวลานั้น ข้อความไม่ได้ถูกเก็บไว้รอส่งให้อัตโนมัติ

ดังนั้นถ้า event ทุกตัวต้องส่งถึงผู้รับอย่างน้อยหนึ่งครั้ง ควรพิจารณา Message broker เช่น RabbitMQ หรือ Azure Service Bus หรืออาจใช้ broker ร่วมกับ SignalR ก็ได้ (เดี๋ยวเราจะกลับมาที่เรื่องนี้อีกครั้ง)

## มาดูกันว่าแต่ละ Protocol เหมาะกับอะไรบ้าง

### HTTP/REST

REST เป็นแบบที่ Developer ส่วนใหญ่น่าจะคุ้นเคยกันดี โดยใช้ HTTP เป็นพื้นฐาน ข้อมูลมักอยู่ในรูป JSON และสื่อความหมายผ่าน resource, URL, HTTP method และ status code เช่น:

```http
GET /api/products/P-100/stock
```

ข้อดีของ REST คือเข้าใจง่าย ทดสอบด้วยเครื่องมือทั่วไปได้ มี logging และ observability รองรับค่อนข้างดี รวมถึงทำงานข้ามภาษาและ platform ได้สะดวก ส่วนข้อแลกเปลี่ยนคือ JSON มีขนาดใหญ่กว่า binary protocol และ contract อาจไม่เข้มเท่า gRPC หากเราไม่ได้ใช้ OpenAPI หรือ generate client จาก schema

โดยส่วนใหญ่ REST เหมาะกับ public API, CRUD, การ query ข้อมูล และ operation ที่ผู้เรียกต้องรู้ผลทันที

### gRPC

gRPC ใช้ Protocol Buffers ในการกำหนด contract และโดยทั่วไปทำงานบน HTTP/2 ข้อมูลเป็น binary ทำให้ payload มีขนาดเล็กและเร็วกว่า JSON นอกจากนี้ยังรองรับทั้ง unary call, client streaming, server streaming และ bidirectional streaming

ตัวนี้เหมาะกับ internal service-to-service communication ที่เราควบคุมได้ทั้งสองฝั่ง ต้องการ type-safe contract หรือมี request ปริมาณสูง แต่ข้อแลกเปลี่ยนคือเวลา debug เราจะเปิดดู payload ด้วยตาเปล่าไม่ง่ายเหมือน REST และการเชื่อมต่อจาก browser ก็มีข้อจำกัดมากกว่า

### WebSocket

WebSocket จะเปิด connection สองทิศทางค้างไว้ ทำให้ทั้ง client และ server ส่งข้อมูลหาอีกฝ่ายได้ตลอดเวลา มี overhead ต่อ message ค่อนข้างต่ำ ฟังดูดี แต่สิ่งที่ตามมาคือเราต้องออกแบบ message format, routing, connection lifecycle, reconnect และ error handling เองมากขึ้น

WebSocket จึงเหมาะกับกรณีที่เราต้องการ protocol เฉพาะ หรือต้องการควบคุมในระดับ transport อย่างละเอียดจริงๆ

### SignalR

SignalR เป็น abstraction สำหรับ realtime communication ใน ASP.NET Core โดยมี Hub เป็นจุดรับส่งข้อความ เราสามารถส่งข้อมูลไปยัง connection รายตัว, user, group หรือทุก connection ได้ และ SignalR จะเลือก transport ที่เหมาะสมให้ เช่น ใช้ WebSocket เมื่อ environment รองรับ

ข้อดีคือ integrate กับ dependency injection, authentication, authorization และ logging ของ ASP.NET Core ได้ดี มี client library สำหรับ .NET, JavaScript, Java และ Swift ในกรณีของเราที่เป็นสอง .NET backends ฝั่งรับ event สามารถใช้ `Microsoft.AspNetCore.SignalR.Client` เชื่อมต่อได้โดยตรง

SignalR เหมาะกับการ push notification และข้อมูลสด แต่ต้องย้ำอีกครั้งว่า SignalR ไม่ใช่ durable queue และ Hub มีอายุสั้น เราจึงไม่ควรเก็บ application state ไว้ใน instance ของ Hub

### Server-Sent Events หรือ SSE

SSE ส่ง event จาก server ไป client ทางเดียวผ่าน HTTP เหมาะกับ progress, notification หรือ feed ที่ client ไม่จำเป็นต้องส่งข้อมูลกลับมาทาง connection เดียวกัน ถ้าเป็น one-way push แบบง่ายๆ SSE ก็เป็นตัวเลือกที่น่าสนใจ แต่สำหรับระบบ .NET ที่ต้องสื่อสารสองทิศทาง SignalR มักจะใช้งานได้สะดวกกว่า

### Message broker

RabbitMQ, Azure Service Bus และ Kafka ไม่ได้เข้ามาแทน REST ในทุกกรณี แต่จะตอบโจทย์เมื่อผู้ส่งไม่ควรผูก availability ไว้กับผู้รับ หรือต้องการเก็บ message เอาไว้จนกว่าจะประมวลผลสำเร็จ

ลองนึกภาพว่า Inventory API ต้องรับประกันว่า Order API จะได้รับ `StockChanged` แม้ในเวลาที่ Order API ล่มอยู่ กรณีนี้ SignalR อย่างเดียวไม่พอ เราควร publish event ไปยัง broker แล้วให้ Order API กลับมา consume เมื่อพร้อมทำงาน ส่วน SignalR ยังสามารถใช้ push event ไปยัง dashboard หรือ connection ที่ online อยู่ได้ตามปกติ

## แบบที่ผมเลือก: REST เป็นคำตอบ SignalR เป็นสัญญาณ

![แผนภาพการสื่อสารระหว่าง Order API และ Inventory API ด้วย REST และ SignalR](architecture-diagram.svg)

จากภาพผมแบ่งหน้าที่ของทั้งสอง protocol ไว้ดังนี้:

1. Order API เรียก REST เพื่ออ่านสต็อกหรือสั่ง reserve สินค้า และรอผลลัพธ์ที่ชัดเจน
2. Order API เปิด SignalR connection ค้างไว้เพื่อรับ `StockChanged` ทันที
3. ถ้า SignalR หลุด client จะ reconnect อัตโนมัติ
4. หลัง reconnect ให้ Order API query snapshot ล่าสุดผ่าน REST อีกครั้ง เพราะ event ที่เกิดตอน connection หลุดอาจหายไป

จำง่ายๆ คือ **event บอกว่ามีบางอย่างเปลี่ยน แต่ REST ใช้ถามสถานะล่าสุดที่เชื่อถือได้**

## ลองสร้างฝั่ง Inventory API

ตัวอย่างนี้ผมเลือกใช้ Minimal API เพื่อให้เราเห็นเฉพาะส่วนสำคัญก่อน โดยไม่ต้องสนใจโครงสร้างอื่นมากจนเกินไป

เริ่มจากสร้าง model และ strongly typed Hub contract:

```csharp
using Microsoft.AspNetCore.SignalR;

public sealed record StockResponse(
    string ProductId,
    int AvailableQuantity,
    DateTimeOffset UpdatedAt);

public sealed record UpdateStockRequest(int AvailableQuantity);

public sealed record StockChanged(
    string ProductId,
    int AvailableQuantity,
    DateTimeOffset OccurredAt);

public interface IInventoryClient
{
    Task StockChanged(StockChanged message);
}

public sealed class InventoryHub : Hub<IInventoryClient>
{
}
```

เหตุผลที่เลือกใช้ strongly typed Hub เพราะช่วยตรวจชื่อ method และชนิดข้อมูลได้ตั้งแต่ตอน compile แทนที่จะกระจาย magic string ไว้หลายจุดใน codebase (พิมพ์ชื่อผิดก็รู้ก่อนรัน)

จากนั้นสร้าง REST endpoint และเปิด SignalR Hub:

```csharp
using System.Collections.Concurrent;
using Microsoft.AspNetCore.SignalR;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddSignalR();

var app = builder.Build();
var stocks = new ConcurrentDictionary<string, StockResponse>();

app.MapGet("/api/products/{productId}/stock", (
    string productId) =>
{
    return stocks.TryGetValue(productId, out var stock)
        ? Results.Ok(stock)
        : Results.NotFound();
});

app.MapPut("/api/products/{productId}/stock", async (
    string productId,
    UpdateStockRequest request,
    IHubContext<InventoryHub, IInventoryClient> hub,
    CancellationToken cancellationToken) =>
{
    if (request.AvailableQuantity < 0)
    {
        return Results.ValidationProblem(new Dictionary<string, string[]>
        {
            ["availableQuantity"] = ["Quantity must be zero or greater."]
        });
    }

    var now = DateTimeOffset.UtcNow;
    var stock = new StockResponse(
        productId,
        request.AvailableQuantity,
        now);

    stocks[productId] = stock;

    await hub.Clients.All.StockChanged(
        new StockChanged(productId, stock.AvailableQuantity, now));

    return Results.Ok(stock);
});

app.MapHub<InventoryHub>("/hubs/inventory");

app.Run();
```

จุดที่อยากให้สังเกตคือ code ไม่ได้สร้าง `InventoryHub` ขึ้นมาเอง แต่ใช้ `IHubContext` เพื่อส่ง event จาก endpoint หรือ application service เนื่องจาก Hub เป็น transient object และไม่ควรถูกใช้เป็นที่เก็บ state

เพื่อให้ตัวอย่างไม่ยาวเกินไป ผมเก็บข้อมูลไว้ใน memory ก่อน แต่ในระบบจริงเราควรเขียน database ให้สำเร็จแล้วจึง push event และยังต้องคิดถึงปัญหา dual write ด้วย เช่น database commit สำเร็จ แต่ process หยุดก่อนส่ง event แบบนี้ผู้รับจะไม่รู้เลยว่าข้อมูลเปลี่ยน หากเป็น event สำคัญควรใช้ **Transactional Outbox** ร่วมกับ Message broker

## ให้ Order API เรียกข้อมูลผ่าน REST

ฝั่ง Order API จะเรียก REST ไปหา Inventory API สิ่งหนึ่งที่ควรระวังคือไม่ควรสร้าง `new HttpClient()` ใหม่ทุก request เพราะอาจเจอปัญหา socket/port exhaustion และการจัดการ DNS ที่ไม่เหมาะสม ในแอปที่ใช้ dependency injection เราสามารถใช้ `IHttpClientFactory` หรือ typed client ได้

เริ่มจากสร้าง client ที่ซ่อนรายละเอียด HTTP ไว้ใน class เดียว:

```csharp
using System.Net;
using System.Net.Http.Json;

public sealed class InventoryClient(HttpClient httpClient)
{
    public async Task<StockResponse?> GetStockAsync(
        string productId,
        CancellationToken cancellationToken)
    {
        using var response = await httpClient.GetAsync(
            $"/api/products/{Uri.EscapeDataString(productId)}/stock",
            cancellationToken);

        if (response.StatusCode is HttpStatusCode.NotFound)
        {
            return null;
        }

        response.EnsureSuccessStatusCode();

        return await response.Content.ReadFromJsonAsync<StockResponse>(
            cancellationToken);
    }
}
```

แล้วลงทะเบียน typed client ใน `Program.cs`:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddHttpClient<InventoryClient>(httpClient =>
{
    httpClient.BaseAddress = new Uri(
        builder.Configuration["Services:InventoryApi"]
        ?? throw new InvalidOperationException(
            "Services:InventoryApi is not configured."));

    httpClient.Timeout = TimeSpan.FromSeconds(3);
});
```

จากนั้นนำไปเรียกใช้จาก endpoint:

```csharp
app.MapPost("/api/orders", async (
    CreateOrderRequest request,
    InventoryClient inventory,
    CancellationToken cancellationToken) =>
{
    var stock = await inventory.GetStockAsync(
        request.ProductId,
        cancellationToken);

    if (stock is null ||
        stock.AvailableQuantity < request.Quantity)
    {
        return Results.Conflict(new
        {
            code = "INSUFFICIENT_STOCK",
            message = "สินค้าไม่เพียงพอ"
        });
    }

    // Create the order and reserve stock here.
    return Results.Accepted();
});
```

เมื่อนำไปใช้บน production เราควรเพิ่ม resilience handler เพื่อรับมือ transient failure แต่อย่าเพิ่งใส่ retry ให้ทุกอย่าง เพราะ operation แต่ละแบบมีผลไม่เหมือนกัน:

- Retry `GET` มักปลอดภัยเพราะเป็น idempotent operation
- ไม่ควร retry `POST` ที่สร้างข้อมูลโดยอัตโนมัติ ถ้ายังไม่มี idempotency key
- กำหนด timeout เสมอ เพื่อไม่ให้ request ค้างจน thread และ connection ถูกใช้หมด
- แยกผลลัพธ์ business error เช่น stock ไม่พอ ออกจาก technical error เช่น timeout หรือ service unavailable
- ใช้ cancellation token ต่อเนื่องจาก incoming request ไปยัง outgoing request

## ต่อ Realtime ด้วย SignalR

ต่อไปเราจะให้ Order API รับ event แบบ realtime เริ่มจากติดตั้ง client package:

```bash
dotnet add package Microsoft.AspNetCore.SignalR.Client
```

แล้วสร้าง `BackgroundService` ไว้ดูแล connection:

```csharp
using Microsoft.AspNetCore.SignalR.Client;

public sealed class InventoryRealtimeWorker(
    IConfiguration configuration,
    InventorySnapshotStore snapshotStore,
    ILogger<InventoryRealtimeWorker> logger)
    : BackgroundService
{
    private HubConnection? _connection;

    protected override async Task ExecuteAsync(
        CancellationToken stoppingToken)
    {
        var inventoryUrl =
            configuration["Services:InventoryApi"]
            ?? throw new InvalidOperationException(
                "Services:InventoryApi is not configured.");

        _connection = new HubConnectionBuilder()
            .WithUrl($"{inventoryUrl.TrimEnd('/')}/hubs/inventory")
            .WithAutomaticReconnect()
            .Build();

        _connection.On<StockChanged>(
            nameof(IInventoryClient.StockChanged),
            message =>
            {
                snapshotStore.Upsert(message);
                logger.LogInformation(
                    "Stock {ProductId} changed to {Quantity}",
                    message.ProductId,
                    message.AvailableQuantity);
            });

        _connection.Reconnected += async connectionId =>
        {
            logger.LogInformation(
                "SignalR reconnected with connection {ConnectionId}",
                connectionId);

            // Re-sync current state through REST here. Events that happened
            // while disconnected aren't guaranteed to be replayed.
            await snapshotStore.MarkForRefreshAsync(stoppingToken);
        };

        await StartWithRetryAsync(_connection, stoppingToken);
        await Task.Delay(Timeout.Infinite, stoppingToken);
    }

    private static async Task StartWithRetryAsync(
        HubConnection connection,
        CancellationToken cancellationToken)
    {
        while (!cancellationToken.IsCancellationRequested)
        {
            try
            {
                await connection.StartAsync(cancellationToken);
                return;
            }
            catch when (!cancellationToken.IsCancellationRequested)
            {
                await Task.Delay(TimeSpan.FromSeconds(5), cancellationToken);
            }
        }
    }

    public override async Task StopAsync(
        CancellationToken cancellationToken)
    {
        if (_connection is not null)
        {
            await _connection.DisposeAsync();
        }

        await base.StopAsync(cancellationToken);
    }
}
```

สุดท้ายลงทะเบียน worker และ store:

```csharp
builder.Services.AddSingleton<InventorySnapshotStore>();
builder.Services.AddHostedService<InventoryRealtimeWorker>();
```

`WithAutomaticReconnect()` ช่วย reconnect ในกรณีที่ connection เคยเชื่อมต่อสำเร็จแล้วหลุด แต่ตอนเริ่มเชื่อมต่อครั้งแรกเรายังต้องจัดการ retry เอง จึงมี `StartWithRetryAsync` อยู่ในตัวอย่าง

ข้อมูลใน memory snapshot เหมาะกับการแสดงผลหรือช่วยลดจำนวน query แต่ไม่ควรใช้เป็นข้อมูลสุดท้ายสำหรับการตัดสินใจสำคัญ เช่น การตัดสต็อกจริง เพราะข้อมูลอาจเก่าหรือมี event หลุดได้ ก่อนทำ transaction เราควรยืนยันสถานะกับ Inventory API อีกครั้งเสมอ

## อย่าลืมเรื่อง Security

ถึงแม้สอง backends จะอยู่ใน private network ก็ไม่ได้หมายความว่าทุก request เชื่อถือได้โดยอัตโนมัติ อย่างน้อยระบบควรมี:

- HTTPS เพื่อเข้ารหัสข้อมูลระหว่างทาง
- OAuth 2.0 Client Credentials, workload identity หรือ mTLS เพื่อยืนยันตัวตนของ service
- Authorization policy จำกัดว่า service ใดเรียก endpoint หรือเชื่อม Hub ใดได้
- Secret และ certificate rotation โดยไม่ฝังค่าไว้ใน source code
- Rate limit และ payload validation ป้องกัน service ที่ผิดปกติทำให้ระบบอื่นล่มตาม

สำหรับ SignalR .NET client สามารถส่ง access token ผ่าน `AccessTokenProvider`:

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl(hubUrl, options =>
    {
        options.AccessTokenProvider = tokenProvider.GetAccessTokenAsync;
    })
    .WithAutomaticReconnect()
    .Build();
```

ทั้ง Hub และ REST endpoint ควรใช้ authorization policy ที่สอดคล้องกัน แต่แยก scope ตามหน้าที่ เช่น `inventory.read`, `inventory.write` และ `inventory.subscribe` เพื่อไม่ให้ service ได้สิทธิ์มากเกินความจำเป็น

## ก่อนขึ้น Production ยังต้องคิดเรื่องอะไรอีกบ้าง

### Contract และ versioning

การ deploy สอง backends ไม่ได้เกิดพร้อมกันทุกครั้ง ดังนั้น contract ควร backward compatible:

- เพิ่ม field ใหม่แบบ optional
- อย่าเปลี่ยนความหมายของ field เดิม
- version REST endpoint เมื่อมี breaking change
- version event name หรือ payload เมื่อ consumer เก่ายังทำงานอยู่
- ใช้ OpenAPI หรือ shared contract package อย่างระมัดระวัง อย่าให้ package กลายเป็น coupling ที่บังคับให้ทุก service deploy พร้อมกัน

### Observability

เวลาระบบมีปัญหา เราควรตามให้ได้ว่า request หรือ event เดินทางผ่าน service ใดมาบ้าง:

- ส่ง correlation ID หรือใช้ W3C Trace Context
- เก็บ latency, error rate, retry count และ reconnect count
- log business identifier เช่น `OrderId` และ `ProductId` แต่ไม่ log token หรือข้อมูลส่วนบุคคลเกินจำเป็น
- ตั้ง alert จากอาการที่ผู้ใช้ได้รับผลกระทบ ไม่ใช่ดูเพียง CPU

### Scale-out

เมื่อ Inventory API มีหลาย instance ตัว connection ของ SignalR จะกระจายอยู่คนละเครื่อง การส่งผ่าน `Clients.All` จาก instance หนึ่งจึงต้องมี scale-out solution เช่น Redis backplane หรือ managed SignalR service เพื่อให้ข้อความไปถึง connection ที่อยู่บน instance อื่นด้วย

ส่วน REST ควรวางไว้หลัง load balancer และไม่ควรพึ่ง in-memory state ของ instance ใด instance หนึ่ง

### Failure และ consistency

พอระบบแยกออกจากกันแล้ว เราจะไม่มี transaction เดียวที่ครอบทั้งสอง database จึงต้องตกลง behavior ให้ชัดว่า:

- ถ้า Inventory API timeout Order API จะ reject, retry หรือรับ order ไว้รอตรวจภายหลัง
- ถ้าตัดสต็อกสำเร็จแต่สร้าง order ไม่สำเร็จ จะชดเชยอย่างไร
- event ซ้ำจะถูกประมวลผลแบบ idempotent อย่างไร
- ข้อมูลยอมเก่าได้กี่วินาที
- ถ้า SignalR หลุดนานเท่าไรจึงถือว่า snapshot ใช้งานไม่ได้

คำถามพวกนี้อาจดูเยอะ แต่คำตอบมีผลต่อความถูกต้องของระบบมากกว่าการเลือก transport เสียอีก

## สรุปว่าเลือกอะไรในสถานการณ์ไหน

| ความต้องการ | ตัวเลือกแรกที่ควรพิจารณา |
|---|---|
| CRUD หรือ query ที่ต้องตอบทันที | REST |
| Internal call ปริมาณสูงและต้องการ strict contract | gRPC |
| Push ข้อมูลสดให้ .NET backend หรือ UI ที่ online | SignalR |
| Push ทางเดียวผ่าน HTTP แบบเรียบง่าย | SSE |
| ทุก message ต้องถูกเก็บและส่งถึงแม้ consumer offline | Message broker |
| Event stream ปริมาณสูงและต้อง replay ย้อนหลัง | Kafka หรือ streaming platform |

สำหรับตัวอย่าง Order API และ Inventory API ผมจะเลือกใช้แบบนี้:

- ใช้ **REST** สำหรับ `GetStock`, `ReserveStock` และ operation ที่ต้องรู้ผลสำเร็จหรือล้มเหลว
- ใช้ **SignalR** สำหรับ `StockChanged` เพื่อให้ผู้รับที่ online อัปเดตได้ทันที
- หลัง reconnect ให้ **sync ผ่าน REST**
- ถ้า `StockChanged` ห้ามสูญหาย ให้เพิ่ม **message broker และ Outbox** ไม่ใช่พยายามทำให้ SignalR กลายเป็น queue

## โดยส่วนตัวคิดว่า

การเลือก protocol ไม่ควรเริ่มจากคำถามว่าตัวไหนเร็วที่สุด หรือตัวไหนกำลังเป็นที่นิยม แต่ควรเริ่มจากข้อมูลของเราต้องการอะไร

ถ้าเป็น operation ที่ต้องรู้ผลสำเร็จหรือล้มเหลวทันที REST เป็นจุดเริ่มต้นที่เข้าใจง่ายและเพียงพอสำหรับระบบส่วนใหญ่ ส่วน SignalR เหมาะกับการแจ้งว่ามีบางอย่างเปลี่ยน เพื่อให้ระบบที่ online อยู่ตอบสนองได้ทันที

สิ่งสำคัญคือเราต้องแยก **คำตอบที่ต้องเชื่อถือ** ออกจาก **สัญญาณที่ต้องรวดเร็ว** ให้ได้ก่อน เมื่อแบ่งหน้าที่ชัดเจน ระบบจะเข้าใจง่าย ทดสอบง่าย และขยายต่อได้โดยไม่ผูกทุกอย่างไว้กับเทคโนโลยีเดียว (ไม่จำเป็นต้องเริ่มด้วย architecture ที่ซับซ้อนเกินความต้องการ)

## อ่านเพิ่มเติม

- [Make HTTP requests using IHttpClientFactory in ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/http-requests)
- [HttpClient guidelines for .NET](https://learn.microsoft.com/dotnet/fundamentals/networking/http/httpclient-guidelines)
- [Overview of ASP.NET Core SignalR](https://learn.microsoft.com/aspnet/core/signalr/introduction)
- [Use hubs in ASP.NET Core SignalR](https://learn.microsoft.com/aspnet/core/signalr/hubs)
- [ASP.NET Core SignalR .NET client](https://learn.microsoft.com/aspnet/core/signalr/dotnet-client)
