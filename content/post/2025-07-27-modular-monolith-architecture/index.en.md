---
title: "Modular Monolith: Architecture that combines the benefits of Monolith and Microservices"
date: 2025-07-27
draft: false
description: "Learn about Modular Monolith architecture that combines the benefits of Monolith and Microservices through a practical E-commerce application example built with C# and ASP.NET Core"
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
author: Suriya Sonpub
image: modular-monolith-cover.png
---

## What is Modular Monolith and Why It's an Interesting Alternative

Modular Monolith is a software architecture that combines the benefits of **Monolith** and **Microservices** by organizing code into modules with clear bounded contexts while maintaining single deployment unit.

### Problems with Traditional Monolith

**Traditional Monolith** often suffers from:

- **Tightly Coupled Code**: All parts are tightly bound, making changes difficult
- **Single Point of Failure**: If one part fails, the entire system fails
- **Technology Lock-in**: Must use the same technology stack throughout
- **Difficult to Scale**: Cannot scale specific parts independently

### Problems with Microservices

While **Microservices** solve many monolith problems, they introduce additional complexity:

- **Distributed System Complexity**: Complexity of distributed systems
- **Network Latency**: Delays from network communication
- **Data Consistency**: Data consistency challenges
- **Operational Overhead**: Requires strong DevOps teams
- **Development Complexity**: Complexity in development and debugging

### Modular Monolith: A Balanced Solution

Modular Monolith offers a balanced approach:

**✅ Benefits:**

- **Modularity**: Clear module separation by business domain
- **Loose Coupling**: Modules communicate through defined interfaces
- **Single Deployment**: Easy to deploy and maintain
- **Easier Testing**: Easier to test than distributed systems
- **Gradual Migration**: Can be extracted to microservices in the future

**⚠️ Considerations:**

- **Technology Constraint**: Still limited by primary technology
- **Shared Database**: May have data coupling issues
- **Resource Scaling**: Cannot scale individual modules separately

## Architecture of Modular Monolith E-commerce Application

I created a [Modular Monolith E-commerce Application](https://github.com/suriyasonp/modular-monolith-ecommerce) using C# and ASP.NET Core to demonstrate the application of these principles in real system development.

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

### 🎯 Key Principles of Modular Monolith

**1. Modules**
Each module represents a distinct business capability such as Orders, Products, Customers

**2. Encapsulation**
Modules hide internal details and expose only necessary functionality through interfaces

**3. Shared Kernel**
Common functionality such as base entities, common utilities, domain events

**4. Communication**
Modules communicate through interfaces or domain events, avoiding direct dependencies

**5. Single Deployment**
All modules are deployed together as a single unit

## Getting Started

### Prerequisites

- .NET 8.0 SDK
- Visual Studio Code or Visual Studio

### Quick Start

**1. Clone Repository:**

```bash
git clone https://github.com/suriyasonp/modular-monolith-ecommerce.git
cd modular-monolith-ecommerce
```

**2. Run Application:**

```bash
# Method 1: Using script (Recommended)
./run.sh

# Method 2: Manual run
dotnet build
dotnet run --project src/ECommerceApp/ECommerceApp.csproj
```

**3. Access API:**

- **Swagger UI**: <http://localhost:5000/swagger/index.html>
- **API Base URL**: <http://localhost:5000/api>
- **Sample Products**: Data will be automatically seeded on startup

### Testing the Application

**Quick Test Script:**

```bash
./test-api.sh
```

## Key API Endpoints

### Products API

- `GET /api/products` - Get all products
- `GET /api/products/{id}` - Get product by ID
- `POST /api/products` - Create new product
- `PUT /api/products/{id}/price` - Update product price
- `PUT /api/products/{id}/stock` - Update product stock

### Customers API

- `GET /api/customers` - Get all customers
- `GET /api/customers/{id}` - Get customer by ID
- `POST /api/customers` - Create new customer

### Orders API

- `GET /api/orders` - Get all orders
- `GET /api/orders/{id}` - Get order by ID
- `GET /api/orders/customer/{customerId}` - Get orders by customer
- `POST /api/orders` - Create new order
- `POST /api/orders/{id}/confirm` - Confirm order
- `POST /api/orders/{id}/ship` - Ship order
- `POST /api/orders/{id}/deliver` - Deliver order

## Benefits and Drawbacks of Modular Monolith

### ✅ Benefits

#### 1. Deployment Simplicity

- Deploy as single unit, reducing infrastructure complexity
- No need to worry about service discovery or load balancing
- Easy rollbacks

#### 2. Performance

- No network latency between modules
- Communication is in-process calls
- Cross-module transactions are straightforward

#### 3. Development and Debugging

- Easier to debug than distributed systems
- Better IDE support
- Simpler end-to-end testing

#### 4. Operational Costs

- Lower infrastructure requirements
- No need for large DevOps teams
- Simpler monitoring and logging

#### 5. Future Flexibility

- Can be extracted to microservices when necessary
- Well-designed modules are easily extractable

### ⚠️ Drawbacks

#### 1. Technology Constraints

- Must use the same primary technology
- Difficult to use different tech stacks

#### 2. Scaling

- Cannot scale individual modules separately
- Resource utilization may not be optimal

#### 3. Database Coupling

- Shared database may create coupling issues
- Schema migrations need careful consideration

#### 4. Team Independence

- Teams still need to coordinate deployments
- Shared codebase may be problematic for large teams

## When to Use Modular Monolith

### 🎯 Suitable for

#### 1. Small to Medium Teams (2-20 people)

- Limited resources for infrastructure complexity
- Need high development velocity

#### 2. New Applications

- Uncertain about domain boundaries
- Need rapid prototyping and iteration

#### 3. Unclear Requirements

- Business requirements change frequently
- Need flexibility for adjustments

#### 4. Limited DevOps Capability

- Team not ready for distributed systems
- Infrastructure automation not mature

### 🚫 Not Suitable for

#### 1. Large Organizations with Many Teams

- Teams need high independence
- Different release cycles

#### 2. Very Different Requirements

- Different scalability requirements per module
- Need different technologies

#### 3. High Compliance and Security Constraints

- Need isolation between components
- Strict regulatory requirements

## Migration Path Planning

### Phase 1: Start with Modular Monolith

```text
Traditional Monolith → Modular Monolith
```

**Activities:**
- Identify domain boundaries
- Create modules by business capabilities
- Implement interfaces for inter-module communication
- Add domain events infrastructure

### Phase 2: Improve Module Independence

```text
Tightly Coupled Modules → Loosely Coupled Modules
```

**Activities:**
- Separate databases per module
- Use event-driven communication
- Implement distributed tracing
- Add per-module monitoring

### Phase 3: Selective Microservices Extraction

```text
Modular Monolith → Hybrid Architecture
```

**Activities:**
- Identify modules suitable for extraction
- Extract modules with different scaling needs
- Implement API gateways
- Setup service mesh

## Conclusion

Modular Monolith is a suitable architecture for many types of organizations, especially teams that want the benefits of modularity but are not ready for the complexity of microservices.

### 🔑 Key Takeaways

**1. Start Simple**
- Use Modular Monolith as starting point
- Focus on domain modeling and clean boundaries
- Improve gradually

**2. Design for Change**
- Use interfaces for inter-module communication
- Implement domain events infrastructure
- Prepare monitoring and observability

**3. Plan Migration Path**
- Identify modules that may need extraction in the future
- Design data access patterns that support separation
- Build culture of modular thinking

**4. Focus on Business Value**
- Don't let technical complexity overshadow business goals
- Use architecture appropriate for team and context
- Remember: Architecture is means, not end

### 🚀 Next Steps

1. **Study the Code**: Visit the [repository](https://github.com/suriyasonp/modular-monolith-ecommerce) to understand implementation details
2. **Try Running**: Execute `./test-api.sh` to see all functionality
3. **Add Features**: Try creating new modules following established patterns
4. **Apply to Production**: Adapt for production environment

Modular Monolith is not a silver bullet, but it's an effective tool for building maintainable, scalable, and evolvable systems in the long term.

## References

1. [Modular Monolith E-commerce Repository](https://github.com/suriyasonp/modular-monolith-ecommerce) - Implementation Example
2. [Domain-Driven Design](https://domainlanguage.com/ddd/) - Eric Evans
3. [Modular Monoliths](https://www.kamilgrzybek.com/design/modular-monolith-primer/) - Kamil Grzybek
4. [Building Microservices](https://samnewman.io/books/building_microservices/) - Sam Newman
5. [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) - Robert C. Martin
6. [ASP.NET Core Documentation](https://docs.microsoft.com/en-us/aspnet/core/) - Microsoft Docs
7. [Monolith vs Microservices vs Modular Monoliths](https://blog.bytebytego.com/p/monolith-vs-microservices-vs-modular) - Monolith vs Microservices vs Modular Monoliths

---
