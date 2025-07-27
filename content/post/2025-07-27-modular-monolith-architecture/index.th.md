---
title: "Modular Monolith: สถาปัตยกรรมที่ผสมผสานข้อดีของ Monolith และ Microservices"
date: 2025-07-27
draft: false
description: "เรียนรู้เกี่ยวกับ Modular Monolith สถาปัตยกรรมที่ผสมผสานข้อดีของ Monolith และ Microservices ผ่านตัวอย่างการพัฒนาแอป E-commerce ด้วย C# และ ASP.NET Core"
categories: 
    - technology
    - software-development
tags:
    - modular-monolith
    - architecture
    - dotnet
    - microservices
    - domain-driven-design
    - clean-architecture
    - e-commerce
    - asp-net-core
author: สุริยา สนภู่
image: modular-monolith-cover.png
---

## Modular Monolith คืออะไร และทำไมถึงเป็นทางเลือกที่น่าสนใจ

Modular Monolith เป็นสถาปัตยกรรมซอฟต์แวร์ที่ผสมผสานข้อดีของ **Monolith** และ **Microservices** เข้าด้วยกัน โดยการจัดระเบียบโค้ดให้เป็นโมดูลที่มีขอบเขตชัดเจน (bounded context) แต่ยังคงการ deploy เป็นหน่วยเดียวกัน

### ปัญหาของ Traditional Monolith

**Traditional Monolith** มักจะประสบปัญหา:

- **Tightly Coupled Code**: โค้ดทุกส่วนผูกติดกันแน่น ยากต่อการแก้ไข
- **Single Point of Failure**: หากส่วนใดส่วนหนึ่งเสีย ระบบทั้งหมดล้มเหลว
- **Technology Lock-in**: ต้องใช้เทคโนโลยีเดียวกันทั้งแอปพลิเคชัน
- **Difficult to Scale**: ไม่สามารถปรับขนาดแค่ส่วนที่ต้องการได้

### ปัญหาของ Microservices

แม้ **Microservices** จะแก้ปัญหาหลายอย่างของ Monolith แต่ก็มีความซับซ้อนเพิ่มขึ้น:

- **Distributed System Complexity**: ความซับซ้อนของระบบกระจาย
- **Network Latency**: ความล่าช้าจากการสื่อสารผ่าน network
- **Data Consistency**: ปัญหาความสอดคล้องของข้อมูล
- **Operational Overhead**: ต้องการทีม DevOps ที่แข็งแกร่ง
- **Development Complexity**: ความซับซ้อนในการพัฒนาและ debug

### Modular Monolith: ทางออกที่สมดุล

Modular Monolith นำเสนอวิธีการที่สมดุล:

**✅ ข้อดีที่ได้รับ:**

- **Modularity**: การแบ่งโมดูลที่ชัดเจนตาม business domain
- **Loose Coupling**: โมดูลสื่อสารผ่าน interfaces ที่กำหนดไว้
- **Single Deployment**: ง่ายต่อการ deploy และ maintain
- **Easier Testing**: ทดสอบได้ง่ายกว่า distributed systems
- **Gradual Migration**: สามารถแยกเป็น microservices ได้ในอนาคต

**⚠️ ข้อควรพิจารณา:**

- **Technology Constraint**: ยังคงถูกจำกัดเทคโนโลยีหลัก
- **Shared Database**: อาจมีปัญหา data coupling
- **Resource Scaling**: ไม่สามารถ scale แต่ละโมดูลแยกได้

## โครงสร้างของ Modular Monolith E-commerce Application

ผมได้สร้างตัวอย่าง [Modular Monolith E-commerce Application](https://github.com/suriyasonp/modular-monolith-ecommerce) ด้วย C# และ ASP.NET Core เพื่อแสดงให้เห็นการประยุกต์ใช้หลักการนี้ในการพัฒนาระบบจริง

### 🏗️ Architecture Overview

```text
ECommerceApp/
├── src/
│   ├── ECommerceApp/                    # Main Web API Application
│   │   ├── Controllers/                 # API Controllers
│   │   ├── Program.cs                   # Application entry point
│   │   └── ECommerceApp.csproj
│   ├── ECommerceApp.Shared/             # Shared Kernel
│   │   ├── Events/                      # Domain Events
│   │   ├── Kernel/                      # Base entities, interfaces
│   │   └── ECommerceApp.Shared.csproj
│   └── Modules/
│       ├── Orders/                      # Orders Module
│       │   ├── Application/             # Use cases, commands, queries
│       │   ├── Domain/                  # Domain entities, events
│       │   ├── Infrastructure/          # Data access, repositories
│       │   ├── OrdersModule.cs          # Module registration
│       │   └── ECommerceApp.Modules.Orders.csproj
│       ├── Products/                    # Products Module
│       │   ├── Application/
│       │   ├── Domain/
│       │   ├── Infrastructure/
│       │   ├── ProductsModule.cs
│       │   └── ECommerceApp.Modules.Products.csproj
│       └── Customers/                   # Customers Module
│           ├── Domain/
│           ├── Infrastructure/
│           ├── CustomersModule.cs
│           └── ECommerceApp.Modules.Customers.csproj
```

### 🎯 หลักการสำคัญของ Modular Monolith

**1. Modules (โมดูล)**
แต่ละโมดูลแทนความสามารถทางธุรกิจที่แตกต่างกัน เช่น Orders, Products, Customers

**2. Encapsulation (การห่อหุ้ม)**
โมดูลซ่อนรายละเอียดภายในและเปิดเผยเฉพาะ functionality ที่จำเป็นผ่าน interfaces

**3. Shared Kernel (แกนร่วม)**
ส่วนที่ใช้ร่วมกัน เช่น base entities, common utilities, domain events

**4. Communication (การสื่อสาร)**
โมดูลสื่อสารผ่าน interfaces หรือ domain events หลีกเลี่ยงการพึ่งพาโดยตรง

**5. Single Deployment (การ Deploy เดียว)**
โมดูลทั้งหมดถูก deploy ร่วมกันเป็นหน่วยเดียว

## การออกแบบโมดูลตาม Domain-Driven Design

### Products Module

**Domain Layer:**
```csharp
public class Product : BaseEntity
{
    public string Name { get; private set; }
    public string Description { get; private set; }
    public decimal Price { get; private set; }
    public int StockQuantity { get; private set; }

    public void UpdatePrice(decimal newPrice)
    {
        if (newPrice <= 0)
            throw new ArgumentException("Price must be positive");
        
        Price = newPrice;
        // Raise domain event
        RaiseDomainEvent(new ProductPriceUpdatedEvent(Id, newPrice));
    }

    public void UpdateStock(int quantity)
    {
        if (StockQuantity + quantity < 0)
            throw new InvalidOperationException("Insufficient stock");
            
        StockQuantity += quantity;
        RaiseDomainEvent(new ProductStockUpdatedEvent(Id, StockQuantity));
    }
}
```

**Application Layer:**
```csharp
public class UpdateProductPriceCommand
{
    public Guid ProductId { get; set; }
    public decimal NewPrice { get; set; }
}

public class UpdateProductPriceHandler
{
    private readonly IProductRepository _repository;

    public async Task<Result> Handle(UpdateProductPriceCommand command)
    {
        var product = await _repository.GetByIdAsync(command.ProductId);
        if (product == null)
            return Result.Failure("Product not found");

        product.UpdatePrice(command.NewPrice);
        await _repository.UpdateAsync(product);
        
        return Result.Success();
    }
}
```

### Orders Module

**Domain Layer:**
```csharp
public class Order : BaseEntity
{
    public Guid CustomerId { get; private set; }
    public OrderStatus Status { get; private set; }
    public List<OrderItem> Items { get; private set; } = new();
    public decimal TotalAmount => Items.Sum(item => item.TotalPrice);

    public static Order Create(Guid customerId, List<OrderItem> items)
    {
        var order = new Order
        {
            CustomerId = customerId,
            Status = OrderStatus.Pending,
            Items = items
        };
        
        order.RaiseDomainEvent(new OrderCreatedEvent(order.Id, customerId));
        return order;
    }

    public void Confirm()
    {
        if (Status != OrderStatus.Pending)
            throw new InvalidOperationException("Only pending orders can be confirmed");
            
        Status = OrderStatus.Confirmed;
        RaiseDomainEvent(new OrderConfirmedEvent(Id));
    }
}
```

### การสื่อสารระหว่างโมดูล

**Interface-based Communication:**
```csharp
// ใน Orders Module
public interface IProductService
{
    Task<Product?> GetProductAsync(Guid productId);
    Task<bool> IsProductAvailableAsync(Guid productId, int quantity);
}

// ใน Products Module
public class ProductService : IProductService
{
    private readonly IProductRepository _repository;

    public async Task<Product?> GetProductAsync(Guid productId)
    {
        return await _repository.GetByIdAsync(productId);
    }

    public async Task<bool> IsProductAvailableAsync(Guid productId, int quantity)
    {
        var product = await _repository.GetByIdAsync(productId);
        return product != null && product.StockQuantity >= quantity;
    }
}
```

**Event-based Communication (สำหรับอนาคต):**
```csharp
public class OrderConfirmedEvent : IDomainEvent
{
    public Guid OrderId { get; }
    public DateTime OccurredOn { get; }

    public OrderConfirmedEvent(Guid orderId)
    {
        OrderId = orderId;
        OccurredOn = DateTime.UtcNow;
    }
}

// Event Handler ใน Products Module
public class OrderConfirmedEventHandler : IEventHandler<OrderConfirmedEvent>
{
    public async Task Handle(OrderConfirmedEvent @event)
    {
        // Update inventory, send notifications, etc.
    }
}
```

## การติดตั้งและรันแอปพลิเคชัน

### Prerequisites

- .NET 8.0 SDK
- Visual Studio Code หรือ Visual Studio

### การเริ่มต้นใช้งาน

**1. Clone Repository:**
```bash
git clone https://github.com/suriyasonp/modular-monolith-ecommerce.git
cd modular-monolith-ecommerce
```

**2. รันแอปพลิเคชัน:**
```bash
# วิธีที่ 1: ใช้ script (แนะนำ)
./run.sh

# วิธีที่ 2: รันด้วยตนเอง
dotnet build
dotnet run --project src/ECommerceApp/ECommerceApp.csproj
```

**3. เข้าถึง API:**

- **Swagger UI**: <http://localhost:5000/swagger/index.html>
- **API Base URL**: <http://localhost:5000/api>
- **Sample Products**: ข้อมูลจะถูกสร้างอัตโนมัติเมื่อเริ่มระบบ

### การทดสอบ API

**Quick Test Script:**

```bash
./test-api.sh
```

**การทดสอบด้วยตนเอง:**

```bash
# ดู Products ทั้งหมด
curl http://localhost:5000/api/products

# สร้าง Customer ใหม่
curl -X POST http://localhost:5000/api/customers \
  -H "Content-Type: application/json" \
  -d '{
    "firstName": "สมชาย",
    "lastName": "ใจดี", 
    "email": "somchai@example.com",
    "address": {
      "street": "123 ถนนสุขุมวิท",
      "city": "กรุงเทพฯ",
      "state": "กทม",
      "zipCode": "10110",
      "country": "ไทย"
    }
  }'

# สร้าง Order ใหม่
curl -X POST http://localhost:5000/api/orders \
  -H "Content-Type: application/json" \
  -d '{
    "customerId": "customer-guid-here",
    "items": [
      {
        "productId": "product-guid-here",
        "quantity": 2
      }
    ]
  }'
```

## API Endpoints ที่สำคัญ

### Products API

- `GET /api/products` - ดูสินค้าทั้งหมด
- `GET /api/products/{id}` - ดูสินค้าตาม ID
- `POST /api/products` - เพิ่มสินค้าใหม่
- `PUT /api/products/{id}/price` - อัปเดตราคาสินค้า
- `PUT /api/products/{id}/stock` - อัปเดตสต็อกสินค้า

### Customers API

- `GET /api/customers` - ดูลูกค้าทั้งหมด
- `GET /api/customers/{id}` - ดูลูกค้าตาม ID
- `POST /api/customers` - เพิ่มลูกค้าใหม่

### Orders API

- `GET /api/orders` - ดูออเดอร์ทั้งหมด
- `GET /api/orders/{id}` - ดูออเดอร์ตาม ID
- `GET /api/orders/customer/{customerId}` - ดูออเดอร์ของลูกค้า
- `POST /api/orders` - สร้างออเดอร์ใหม่
- `POST /api/orders/{id}/confirm` - ยืนยันออเดอร์
- `POST /api/orders/{id}/ship` - จัดส่งออเดอร์
- `POST /api/orders/{id}/deliver` - ส่งมอบออเดอร์

## ข้อดีและข้อเสียของ Modular Monolith

### ✅ ข้อดี

**1. ความเรียบง่ายในการ Deploy**
- Deploy เป็นหน่วยเดียว ลดความซับซ้อนของ infrastructure
- ไม่ต้องกังวลเรื่อง service discovery หรือ load balancing
- การ rollback ทำได้ง่าย

**2. ประสิทธิภาพ**
- ไม่มี network latency ระหว่างโมดูล
- การสื่อสารเป็น in-process calls
- Transaction ข้าม modules ทำได้ง่าย

**3. การพัฒนาและ Debug**
- Debug ได้ง่ายกว่า distributed systems
- IDE support ดีกว่า
- End-to-end testing ง่ายกว่า

**4. ต้นทุนการดำเนินงาน**
- Infrastructure requirements ต่ำกว่า
- ไม่ต้องมีทีม DevOps ขนาดใหญ่
- Monitoring และ logging ง่ายกว่า

**5. ความยืดหยุ่นในอนาคต**
- สามารถแยกเป็น microservices ได้เมื่อจำเป็น
- โมดูลที่ออกแบบดีสามารถ extract ออกมาได้ง่าย

### ⚠️ ข้อเสีย

**1. ข้อจำกัดด้านเทคโนโลยี**
- ต้องใช้เทคโนโลยีหลักเดียวกัน
- ยากต่อการใช้ different tech stacks

**2. การ Scale**
- ไม่สามารถ scale แต่ละโมดูลแยกได้
- Resource utilization อาจไม่เหมาะสม

**3. Database Coupling**
- หากใช้ shared database อาจมีปัญหา coupling
- Schema migration ต้องระวังผลกระทบ

**4. Team Independence**
- ทีมยังคงต้อง coordinate ใน deployment
- Shared codebase อาจเป็นปัญหาในทีมใหญ่

## เมื่อไหร่ควรใช้ Modular Monolith

### 🎯 เหมาะสำหรับ:

**1. ทีมขนาดเล็กถึงกลาง (2-20 คน)**
- มีทรัพยากรจำกัดสำหรับ infrastructure complexity
- ต้องการ development velocity สูง

**2. แอปพลิเคชันใหม่**
- ยังไม่แน่ใจเรื่อง domain boundaries
- ต้องการ rapid prototyping และ iteration

**3. Requirements ยังไม่ชัดเจน**
- Business requirements เปลี่ยนแปลงบ่อย
- ต้องการความยืดหยุ่นในการปรับเปลี่ยน

**4. Limited DevOps Capability**
- ทีมยังไม่พร้อมสำหรับ distributed systems
- Infrastructure automation ยังไม่เข้มแข็ง

### 🚫 ไม่เหมาะสำหรับ:

**1. องค์กรขนาดใหญ่ที่มีทีมมาก**
- ทีมต้องการ independence สูง
- มี different release cycles

**2. Requirements ที่แตกต่างกันมาก**
- แต่ละส่วนมี scalability requirements ต่างกัน
- ต้องการ different technologies

**3. Compliance และ Security ข้อจำกัดสูง**
- ต้องการ isolation ระหว่าง components
- มี regulatory requirements ที่เข้มงวด

## การวางแผน Migration Path

### Phase 1: เริ่มต้นด้วย Modular Monolith

```
Traditional Monolith → Modular Monolith
```

**Activities:**
- ระบุ domain boundaries
- สร้าง modules ตาม business capabilities
- Implement interfaces สำหรับ inter-module communication
- เพิ่ม domain events infrastructure

### Phase 2: ปรับปรุง Module Independence

```
Tightly Coupled Modules → Loosely Coupled Modules
```

**Activities:**
- แยก databases per module
- ใช้ event-driven communication
- Implement distributed tracing
- เพิ่ม monitoring per module

### Phase 3: Selective Microservices Extraction

```
Modular Monolith → Hybrid Architecture
```

**Activities:**
- ระบุ modules ที่เหมาะสมสำหรับ extraction
- Extract modules ที่มี different scaling needs
- Implement API gateways
- Setup service mesh

## Best Practices สำหรับ Modular Monolith

### 🏗️ การออกแบบ Architecture

**1. ใช้ Domain-Driven Design**
```csharp
// แยก bounded context ชัดเจน
namespace ECommerceApp.Modules.Orders.Domain
{
    public class Order { } // Order ใน Orders context
}

namespace ECommerceApp.Modules.Catalog.Domain  
{
    public class Product { } // Product ใน Catalog context
}
```

**2. Dependency Direction**
```csharp
// ❌ ผิด: High-level module พึ่งพา low-level
public class OrderService
{
    private SqlOrderRepository _repository; // Direct dependency
}

// ✅ ถูก: Dependency Inversion
public class OrderService
{
    private readonly IOrderRepository _repository; // Interface dependency
}
```

**3. Module Registration**
```csharp
public static class ModuleExtensions
{
    public static IServiceCollection AddOrdersModule(this IServiceCollection services)
    {
        services.AddScoped<IOrderRepository, SqlOrderRepository>();
        services.AddScoped<OrderService>();
        services.AddMediatR(typeof(OrdersModule));
        return services;
    }
}
```

### 🔄 การจัดการ Communication

**1. Synchronous Communication**
```csharp
// Interface-based communication สำหรับ immediate consistency
public interface IInventoryService
{
    Task<bool> IsAvailableAsync(Guid productId, int quantity);
    Task ReserveAsync(Guid productId, int quantity);
}
```

**2. Asynchronous Communication**
```csharp
// Event-based communication สำหรับ eventual consistency
public class OrderConfirmedEvent : IDomainEvent
{
    public Guid OrderId { get; }
    public List<OrderItem> Items { get; }
}

[EventHandler]
public class UpdateInventoryHandler : IEventHandler<OrderConfirmedEvent>
{
    public async Task Handle(OrderConfirmedEvent @event)
    {
        // Update inventory asynchronously
    }
}
```

### 🗃️ การจัดการ Data

**1. Shared Database with Schema Separation**
```sql
-- Orders schema
CREATE SCHEMA Orders;
CREATE TABLE Orders.Orders (...);
CREATE TABLE Orders.OrderItems (...);

-- Products schema  
CREATE SCHEMA Products;
CREATE TABLE Products.Products (...);
CREATE TABLE Products.Categories (...);
```

**2. Database per Module (Advanced)**
```csharp
public class OrdersDbContext : DbContext
{
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer(connectionString, options =>
        {
            options.MigrationsHistoryTable("__OrdersMigrationsHistory", "Orders");
        });
    }
}
```

### 🧪 การทดสอบ

**1. Module Integration Tests**
```csharp
[Test]
public async Task CreateOrder_WithValidProducts_ShouldSucceed()
{
    // Arrange
    var productService = ServiceProvider.GetService<IProductService>();
    var orderService = ServiceProvider.GetService<IOrderService>();
    
    // Act
    var result = await orderService.CreateOrderAsync(customerId, orderItems);
    
    // Assert
    Assert.IsTrue(result.IsSuccess);
}
```

**2. Contract Tests**
```csharp
[Test]
public void ProductService_ShouldImplementIProductService()
{
    // Verify that ProductService implements all required interfaces
    var productService = new ProductService();
    Assert.IsInstanceOf<IProductService>(productService);
}
```

## Real-world Considerations

### 🔧 Production Enhancements

**1. Database Integration**
```csharp
// Replace in-memory repositories with Entity Framework Core
services.AddDbContext<OrdersDbContext>(options =>
    options.UseSqlServer(connectionString));

services.AddScoped<IOrderRepository, EfOrderRepository>();
```

**2. Authentication & Authorization**
```csharp
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true
        };
    });
```

**3. Input Validation**
```csharp
public class CreateOrderCommandValidator : AbstractValidator<CreateOrderCommand>
{
    public CreateOrderCommandValidator()
    {
        RuleFor(x => x.CustomerId).NotEmpty();
        RuleFor(x => x.Items).NotEmpty();
        RuleForEach(x => x.Items).SetValidator(new OrderItemValidator());
    }
}
```

**4. Error Handling**
```csharp
public class GlobalExceptionMiddleware
{
    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        try
        {
            await next(context);
        }
        catch (DomainException ex)
        {
            await HandleDomainExceptionAsync(context, ex);
        }
        catch (Exception ex)
        {
            await HandleGenericExceptionAsync(context, ex);
        }
    }
}
```

### 📊 Monitoring และ Observability

**1. Health Checks**
```csharp
services.AddHealthChecks()
    .AddDbContextCheck<OrdersDbContext>("orders-db")
    .AddDbContextCheck<ProductsDbContext>("products-db")
    .AddUrlGroup(new Uri("https://external-api.com/health"), "external-api");
```

**2. Logging**
```csharp
services.AddSerilog((context, config) =>
{
    config.ReadFrom.Configuration(context.Configuration)
          .Enrich.WithProperty("Module", "Orders")
          .WriteTo.Console()
          .WriteTo.File("logs/orders-.log", rollingInterval: RollingInterval.Day);
});
```

**3. Metrics**
```csharp
public class OrderMetrics
{
    private static readonly Counter OrdersCreated = Metrics
        .CreateCounter("orders_created_total", "Total number of orders created");
        
    public void IncrementOrdersCreated() => OrdersCreated.Inc();
}
```

## ข้อสรุป

Modular Monolith เป็นสถาปัตยกรรมที่เหมาะสมสำหรับองค์กรหลายประเภท โดยเฉพาะทีมที่ต้องการประโยชน์ของ modularity แต่ยังไม่พร้อมสำหรับความซับซ้อนของ microservices

### 🔑 Key Takeaways:

**1. เริ่มต้นอย่างเรียบง่าย**
- ใช้ Modular Monolith เป็นจุดเริ่มต้น
- Focus บน domain modeling และ clean boundaries
- ปรับปรุงแบบค่อยเป็นค่อยไป

**2. ออกแบบให้พร้อมสำหรับการเปลี่ยนแปลง**
- ใช้ interfaces สำหรับ inter-module communication
- Implement domain events infrastructure
- เตรียม monitoring และ observability

**3. วางแผน Migration Path**
- ระบุ modules ที่อาจต้อง extract ในอนาคต
- ออกแบบ data access patterns ที่รองรับการแยก
- สร้าง culture ของ modular thinking

**4. Focus บน Business Value**
- อย่าให้ technical complexity บดบัง business goals
- ใช้สถาปัตยกรรมที่เหมาะสมกับทีมและ context
- Remember: Architecture คือ means ไม่ใช่ end

### 🚀 Next Steps:

1. **ศึกษาโค้ด**: ดู [repository](https://github.com/suriyasonp/modular-monolith-ecommerce) เพื่อเข้าใจ implementation details
2. **ทดลองเรียกใช้**: รัน `./test-api.sh` เพื่อดูฟีเจอร์ทั้งหมด
3. **เพิ่มฟีเจอร์**: ลองสร้างโมดูลใหม่ตามแพทเทิร์นที่กำหนด
4. **นำไปใช้จริง**: ปรับแต่งสำหรับ production environment

Modular Monolith ไม่ใช่ silver bullet แต่เป็นเครื่องมือที่มีประสิทธิภาพสำหรับการสร้างระบบที่ maintainable, scalable และ evolvable ในระยะยาว

## แหล่งอ้างอิง

1. [Modular Monolith E-commerce Repository](https://github.com/suriyasonp/modular-monolith-ecommerce) - ตัวอย่างการ implement
2. [Domain-Driven Design](https://domainlanguage.com/ddd/) - Eric Evans
3. [Modular Monoliths](https://www.kamilgrzybek.com/design/modular-monolith-primer/) - Kamil Grzybek
4. [Building Microservices](https://samnewman.io/books/building_microservices/) - Sam Newman
5. [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) - Robert C. Martin
6. [ASP.NET Core Documentation](https://docs.microsoft.com/en-us/aspnet/core/) - Microsoft Docs
7. [Monolith vs Microservices vs Modular Monoliths](https://blog.bytebytego.com/p/monolith-vs-microservices-vs-modular) - Monolith vs Microservices vs Modular Monoliths

---
