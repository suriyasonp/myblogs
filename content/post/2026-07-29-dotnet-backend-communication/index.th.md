---
title: "ออกแบบการสื่อสารระหว่างสอง .NET Backends: REST สำหรับ Request และ SignalR สำหรับ Realtime"
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
description: "แนวทางเลือก protocol สำหรับให้สอง .NET backends แลกเปลี่ยนข้อมูล ทั้งแบบ request-response และ realtime พร้อมตัวอย่าง ASP.NET Core ที่ใช้ REST ร่วมกับ SignalR และข้อควรระวังสำหรับระบบจริง"
---

เมื่อระบบมี backend มากกว่าหนึ่งตัว คำถามแรกมักเป็น “จะให้สองระบบคุยกันด้วย protocol อะไร” แต่คำถามที่ช่วยให้เราออกแบบได้ดีกว่าคือ “ผู้ส่งต้องการคำตอบทันทีหรือไม่ และข้อมูลที่ส่งต้องห้ามสูญหายหรือเปล่า”

บทความนี้ใช้ตัวอย่างสองระบบ:

- **Order API** รับผิดชอบการสร้างและติดตามคำสั่งซื้อ
- **Inventory API** เป็นเจ้าของข้อมูลสต็อกสินค้า

เมื่อ Order API ต้องตรวจว่าสินค้ามีพอหรือไม่ มันต้องส่งคำถามและรอคำตอบ นี่คือการสื่อสารแบบ **request-response** ซึ่ง REST เหมาะกับงานนี้

แต่เมื่อจำนวนสินค้าเปลี่ยน และ Order API ต้องรู้การเปลี่ยนแปลงโดยไม่ต้องยิง request ถามซ้ำทุกวินาที เราต้องการช่องทางแบบ **realtime** ซึ่ง SignalR เหมาะกับการแจ้งเตือนลักษณะนี้

สองวิธีนี้ไม่จำเป็นต้องเลือกอย่างใดอย่างหนึ่ง ระบบที่ดีมักใช้ทั้งคู่ โดยให้แต่ละวิธีรับผิดชอบงานที่ตัวเองถนัด

## เริ่มจากรูปแบบการสื่อสาร ไม่ใช่ชื่อ protocol

การสื่อสารระหว่าง backend แบ่งเป็นรูปแบบหลักได้ดังนี้

| รูปแบบ | ผู้ส่งรอคำตอบหรือไม่ | ตัวอย่าง | ทางเลือกที่พบบ่อย |
|---|---:|---|---|
| Request-response | รอ | ตรวจสต็อก, อ่านข้อมูลลูกค้า, คำนวณราคา | HTTP/REST, gRPC, GraphQL |
| Realtime push | ไม่ต้องถามซ้ำ | แจ้ง stock changed, progress, live status | SignalR, WebSocket, Server-Sent Events |
| Asynchronous messaging | ไม่รอและต้องส่งให้ถึง | order created, payment completed | RabbitMQ, Azure Service Bus, Kafka |
| Streaming | รับข้อมูลต่อเนื่อง | telemetry, log stream, market feed | gRPC streaming, Kafka, WebSocket |

คำว่า **realtime** ไม่ได้แปลว่า **reliable** เสมอไป SignalR ส่งข้อมูลไปยัง connection ที่เชื่อมต่ออยู่ได้รวดเร็ว แต่ถ้าผู้รับ offline ในขณะนั้น ข้อความอาจไม่ถูกเก็บไว้รอส่งภายหลัง ถ้าทุก event ต้องส่งถึงผู้รับอย่างน้อยหนึ่งครั้ง ควรใช้ message broker เช่น RabbitMQ หรือ Azure Service Bus แทน หรือใช้ broker ร่วมกับ SignalR

## Protocol ที่ควรรู้จัก

### HTTP/REST

REST ใช้ HTTP เป็นพื้นฐาน ข้อมูลมักอยู่ในรูป JSON และสื่อความหมายผ่าน resource, URL, HTTP method และ status code เช่น:

```http
GET /api/products/P-100/stock
```

จุดแข็งของ REST คือเข้าใจง่าย ทดสอบด้วยเครื่องมือทั่วไปได้ มี logging และ observability รองรับดี รวมถึงทำงานข้ามภาษาและ platform ได้สะดวก ข้อแลกเปลี่ยนคือ JSON มีขนาดใหญ่กว่า binary protocol และ contract อาจไม่เข้มเท่า gRPC หากไม่มี OpenAPI หรือการสร้าง client จาก schema

เหมาะกับ public API, CRUD, การ query ข้อมูล และคำสั่งที่ผู้เรียกต้องรู้ผลทันที

### gRPC

gRPC ใช้ Protocol Buffers กำหนด contract และโดยทั่วไปทำงานบน HTTP/2 มี payload แบบ binary ที่เล็กและเร็ว รองรับ unary call รวมถึง client, server และ bidirectional streaming

เหมาะกับ internal service-to-service communication ที่ทั้งสองฝั่งควบคุมได้ ต้องการ type-safe contract หรือมีปริมาณ request สูง ข้อแลกเปลี่ยนคือ debug ด้วยตาเปล่ายากกว่า REST และการเชื่อมต่อจาก browser มีข้อจำกัดมากกว่า

### WebSocket

WebSocket เปิด connection สองทิศทางค้างไว้ ทั้ง client และ server จึงส่งข้อมูลหาอีกฝ่ายได้ตลอดเวลา มี overhead ต่อ message ต่ำ แต่เราต้องออกแบบ message format, routing, connection lifecycle, reconnect และ error handling เองมากขึ้น

เหมาะเมื่อเราต้องการ protocol เฉพาะหรือควบคุมระดับ transport อย่างละเอียด

### SignalR

SignalR เป็น abstraction สำหรับ realtime communication ใน ASP.NET Core โดยมี Hub เป็นจุดรับส่งข้อความ รองรับการส่งไปยัง connection รายตัว user, group หรือทุก connection และเลือก transport ที่เหมาะสมให้ เช่น WebSocket เมื่อ environment รองรับ

จุดเด่นคือ integrate กับ dependency injection, authentication, authorization และ logging ของ ASP.NET Core ได้ดี มี client library สำหรับ .NET, JavaScript, Java และ Swift สำหรับสอง .NET backends ฝั่งที่รับ event สามารถใช้ `Microsoft.AspNetCore.SignalR.Client` เชื่อมต่อเป็น .NET client ได้โดยตรง

SignalR เหมาะกับการ push notification และข้อมูลสด แต่ไม่ใช่ durable queue และ Hub มีอายุสั้น จึงไม่ควรเก็บ application state ไว้ใน instance ของ Hub

### Server-Sent Events หรือ SSE

SSE ส่ง event จาก server ไป client ทางเดียวผ่าน HTTP เหมาะกับ progress, notification หรือ feed ที่ client ไม่จำเป็นต้องส่งข้อมูลกลับผ่าน connection เดียวกัน ใช้ง่ายกว่า WebSocket ในกรณี one-way push แต่ SignalR มักสะดวกกว่าสำหรับระบบ .NET ที่ต้องรองรับหลาย transport และสื่อสารสองทิศทาง

### Message broker

RabbitMQ, Azure Service Bus และ Kafka ไม่ใช่ทางเลือกแทน REST ทุกกรณี แต่เป็นคำตอบเมื่อผู้ส่งไม่ควรผูก availability กับผู้รับ หรือต้องการเก็บ message จนกว่าจะประมวลผลสำเร็จ

ถ้า Inventory API ต้องรับประกันว่า Order API จะได้รับ `StockChanged` แม้ Order API ล่มอยู่ SignalR อย่างเดียวไม่พอ ควร publish event ไปยัง broker แล้วให้ Order API consume เมื่อกลับมาทำงาน ส่วน SignalR อาจยังใช้สำหรับ push event ไปยัง dashboard หรือ connection ที่ online อยู่

## แบบที่แนะนำ: REST เป็นคำตอบ SignalR เป็นสัญญาณ

![แผนภาพการสื่อสารระหว่าง Order API และ Inventory API ด้วย REST และ SignalR](architecture-diagram.svg)

แนวทางในภาพแบ่งหน้าที่ชัดเจน:

1. Order API เรียก REST เพื่ออ่านสต็อกหรือสั่ง reserve สินค้า และรอผลลัพธ์ที่ชัดเจน
2. Order API เปิด SignalR connection ค้างไว้เพื่อรับ `StockChanged` ทันที
3. ถ้า SignalR หลุด client จะ reconnect อัตโนมัติ
4. หลัง reconnect ให้ Order API query snapshot ล่าสุดผ่าน REST อีกครั้ง เพราะ event ที่เกิดตอน connection หลุดอาจหายไป

หัวใจของแบบนี้คือ **event บอกว่ามีบางอย่างเปลี่ยน แต่ REST บอกสถานะล่าสุดที่เชื่อถือได้**

## ตัวอย่างฝั่ง Inventory API

ตัวอย่างนี้ใช้ Minimal API เพื่อให้เห็นส่วนสำคัญโดยไม่ถูกกลบด้วยโครงสร้างอื่น

เริ่มจาก model และ strongly typed Hub contract:

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

Strongly typed Hub ช่วยตรวจชื่อ method และชนิดข้อมูลตอน compile แทนการกระจาย magic string หลายจุดใน codebase

จากนั้นเปิด REST endpoint และ SignalR Hub:

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

จุดที่ควรสังเกตคือ code ไม่ได้สร้าง `InventoryHub` ขึ้นมาเอง แต่ใช้ `IHubContext` เพื่อส่ง event จาก endpoint หรือ application service เพราะ Hub เป็น transient object และไม่ควรถูกใช้เป็นที่เก็บ state

ตัวอย่างเก็บข้อมูลใน memory เพื่อให้สั้น ในระบบจริงควรเขียน database ให้สำเร็จก่อน push event และต้องพิจารณาปัญหา dual write: ถ้า database commit สำเร็จ แต่ process หยุดก่อนส่ง event ผู้รับจะไม่รู้ว่าข้อมูลเปลี่ยน สำหรับ event สำคัญให้ใช้ **Transactional Outbox** ร่วมกับ message broker

## ตัวอย่าง REST client ใน Order API

อย่าสร้าง `new HttpClient()` ใหม่ทุก request เพราะอาจนำไปสู่ปัญหา socket/port exhaustion และการจัดการ DNS ที่ไม่เหมาะสม ในแอปที่ใช้ dependency injection ให้ใช้ `IHttpClientFactory` หรือ typed client

สร้าง client ที่ซ่อนรายละเอียด HTTP ไว้ใน class เดียว:

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

ลงทะเบียน typed client ใน `Program.cs`:

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

และเรียกใช้จาก endpoint:

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

สำหรับ production ควรเพิ่ม resilience handler เพื่อรับมือ transient failure แต่ต้องระวังการ retry:

- Retry `GET` มักปลอดภัยเพราะเป็น idempotent operation
- อย่า retry `POST` ที่สร้างข้อมูลโดยอัตโนมัติ ถ้ายังไม่มี idempotency key
- กำหนด timeout เสมอ เพื่อไม่ให้ request ค้างจน thread และ connection ถูกใช้หมด
- แยกผลลัพธ์ business error เช่น stock ไม่พอ ออกจาก technical error เช่น timeout หรือ service unavailable
- ใช้ cancellation token ต่อเนื่องจาก incoming request ไปยัง outgoing request

## ตัวอย่าง SignalR client ใน Order API

ติดตั้ง client package:

```bash
dotnet add package Microsoft.AspNetCore.SignalR.Client
```

สร้าง `BackgroundService` เพื่อดูแล connection:

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

ลงทะเบียน worker และ store:

```csharp
builder.Services.AddSingleton<InventorySnapshotStore>();
builder.Services.AddHostedService<InventoryRealtimeWorker>();
```

`WithAutomaticReconnect()` ช่วย reconnect หลัง connection ที่เริ่มสำเร็จแล้วหลุด แต่การเชื่อมต่อครั้งแรกยังต้องจัดการ retry เอง จึงมี `StartWithRetryAsync` ในตัวอย่าง

อย่าใช้ข้อมูลใน memory snapshot เป็นหลักฐานสุดท้ายสำหรับการตัดสินใจที่สำคัญ เช่น การตัดสต็อกจริง เพราะข้อมูลอาจเก่าหรือมี event หลุด ให้ใช้ snapshot เพื่อแสดงผลหรือช่วยลดการ query และยืนยันสถานะกับ Inventory API ในขั้นตอนทำธุรกรรมเสมอ

## Security ระหว่างสอง backends

การอยู่ใน private network ไม่ได้แปลว่า request เชื่อถือได้โดยอัตโนมัติ อย่างน้อยควรมี:

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

Hub และ REST endpoint ควรใช้ authorization policy ที่สอดคล้องกัน แต่แยก scope ตามหน้าที่ เช่น `inventory.read`, `inventory.write` และ `inventory.subscribe`

## สิ่งที่ต้องออกแบบเพิ่มก่อนขึ้น production

### Contract และ versioning

การ deploy สอง backends อาจไม่เกิดพร้อมกัน contract จึงต้อง backward compatible:

- เพิ่ม field ใหม่แบบ optional
- อย่าเปลี่ยนความหมายของ field เดิม
- version REST endpoint เมื่อมี breaking change
- version event name หรือ payload เมื่อ consumer เก่ายังทำงานอยู่
- ใช้ OpenAPI หรือ shared contract package อย่างระมัดระวัง อย่าให้ package กลายเป็น coupling ที่บังคับให้ทุก service deploy พร้อมกัน

### Observability

ทุก request และ event ควรตามรอยข้าม service ได้:

- ส่ง correlation ID หรือใช้ W3C Trace Context
- เก็บ latency, error rate, retry count และ reconnect count
- log business identifier เช่น `OrderId` และ `ProductId` แต่ไม่ log token หรือข้อมูลส่วนบุคคลเกินจำเป็น
- ตั้ง alert จากอาการที่ผู้ใช้ได้รับผลกระทบ ไม่ใช่ดูเพียง CPU

### Scale-out

เมื่อ Inventory API มีหลาย instance connection ของ SignalR จะกระจายอยู่คนละเครื่อง การส่งผ่าน `Clients.All` จาก instance หนึ่งจึงต้องมี scale-out solution เช่น Redis backplane หรือ managed SignalR service เพื่อให้ข้อความไปถึง connection ที่อยู่บน instance อื่น

ส่วน REST ควรวางหลัง load balancer และหลีกเลี่ยงการพึ่ง in-memory state ของ instance ใด instance หนึ่ง

### Failure และ consistency

Distributed system ไม่มี transaction เดียวครอบทั้งสอง database เราจึงต้องกำหนดให้ชัดว่า:

- ถ้า Inventory API timeout Order API จะ reject, retry หรือรับ order ไว้รอตรวจภายหลัง
- ถ้าตัดสต็อกสำเร็จแต่สร้าง order ไม่สำเร็จ จะชดเชยอย่างไร
- event ซ้ำจะถูกประมวลผลแบบ idempotent อย่างไร
- ข้อมูลยอมเก่าได้กี่วินาที
- ถ้า SignalR หลุดนานเท่าไรจึงถือว่า snapshot ใช้งานไม่ได้

คำตอบเหล่านี้มีผลต่อความถูกต้องของระบบมากกว่าการเลือก transport เพียงอย่างเดียว

## เลือกอะไรในสถานการณ์ไหน

| ความต้องการ | ตัวเลือกแรกที่ควรพิจารณา |
|---|---|
| CRUD หรือ query ที่ต้องตอบทันที | REST |
| Internal call ปริมาณสูงและต้องการ strict contract | gRPC |
| Push ข้อมูลสดให้ .NET backend หรือ UI ที่ online | SignalR |
| Push ทางเดียวผ่าน HTTP แบบเรียบง่าย | SSE |
| ทุก message ต้องถูกเก็บและส่งถึงแม้ consumer offline | Message broker |
| Event stream ปริมาณสูงและต้อง replay ย้อนหลัง | Kafka หรือ streaming platform |

สำหรับกรณีตัวอย่างนี้ คำตอบที่ใช้งานได้จริงคือ:

- ใช้ **REST** สำหรับ `GetStock`, `ReserveStock` และ operation ที่ต้องรู้ผลสำเร็จหรือล้มเหลว
- ใช้ **SignalR** สำหรับ `StockChanged` เพื่อให้ผู้รับที่ online อัปเดตได้ทันที
- หลัง reconnect ให้ **sync ผ่าน REST**
- ถ้า `StockChanged` ห้ามสูญหาย ให้เพิ่ม **message broker และ Outbox** ไม่ใช่พยายามทำให้ SignalR กลายเป็น queue

การเลือก protocol ที่ดีไม่ใช่เลือกตัวที่เร็วที่สุด แต่คือการทำให้ failure behavior ตรงกับความสำคัญของข้อมูล เมื่อแยก “คำตอบที่ต้องเชื่อถือ” ออกจาก “สัญญาณที่ต้องรวดเร็ว” ได้ ระบบจะเข้าใจง่าย ทดสอบง่าย และขยายต่อได้โดยไม่ผูกทุกอย่างไว้กับเทคโนโลยีเดียว

## อ่านเพิ่มเติม

- [Make HTTP requests using IHttpClientFactory in ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/http-requests)
- [HttpClient guidelines for .NET](https://learn.microsoft.com/dotnet/fundamentals/networking/http/httpclient-guidelines)
- [Overview of ASP.NET Core SignalR](https://learn.microsoft.com/aspnet/core/signalr/introduction)
- [Use hubs in ASP.NET Core SignalR](https://learn.microsoft.com/aspnet/core/signalr/hubs)
- [ASP.NET Core SignalR .NET client](https://learn.microsoft.com/aspnet/core/signalr/dotnet-client)

