# Service Layer Pattern

## 1. Tổng quan

Service Layer tách business logic ra khỏi Controller, tạo layer trung gian xử lý nghiệp vụ.

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌──────────┐
│ Controller  │────►│   Service   │────►│ Repository  │────►│ Database │
│  (HTTP)     │     │  (Business) │     │   (Data)    │     │          │
└─────────────┘     └─────────────┘     └─────────────┘     └──────────┘
```

## 2. Vấn đề: Fat Controller

```javascript
// ❌ Bad: Controller chứa quá nhiều logic
class OrderController {
  async createOrder(req, res) {
    const { userId, items } = req.body;

    // Validation
    if (!items || items.length === 0) {
      return res.status(400).json({ error: "Items required" });
    }

    // Check user exists
    const user = await User.findById(userId);
    if (!user) {
      return res.status(404).json({ error: "User not found" });
    }

    // Check stock
    for (const item of items) {
      const product = await Product.findById(item.productId);
      if (!product) {
        return res.status(404).json({ error: `Product ${item.productId} not found` });
      }
      if (product.stock < item.quantity) {
        return res.status(400).json({ error: `${product.name} out of stock` });
      }
    }

    // Calculate total
    let total = 0;
    for (const item of items) {
      const product = await Product.findById(item.productId);
      total += product.price * item.quantity;
    }

    // Apply discount
    if (user.membershipLevel === "gold") {
      total *= 0.9;
    }

    // Process payment
    const paymentResult = await PaymentGateway.charge(user.paymentMethod, total);
    if (!paymentResult.success) {
      return res.status(400).json({ error: "Payment failed" });
    }

    // Create order
    const order = await Order.create({
      userId,
      items,
      total,
      paymentId: paymentResult.id,
    });

    // Update stock
    for (const item of items) {
      await Product.findByIdAndUpdate(item.productId, {
        $inc: { stock: -item.quantity },
      });
    }

    // Send email
    await EmailService.send(user.email, "Order Confirmation", { order });

    res.status(201).json({ data: order });
  }
}
```

## 3. Giải pháp: Service Layer

### Service

```javascript
// services/OrderService.js
class OrderService {
  constructor(orderRepository, userRepository, productRepository, paymentService, emailService) {
    this.orderRepository = orderRepository;
    this.userRepository = userRepository;
    this.productRepository = productRepository;
    this.paymentService = paymentService;
    this.emailService = emailService;
  }

  async createOrder(userId, items) {
    // Validate items
    if (!items || items.length === 0) {
      throw new ValidationError("Items are required");
    }

    // Get user
    const user = await this.userRepository.findById(userId);
    if (!user) {
      throw new NotFoundError("User not found");
    }

    // Validate stock and get products
    const products = await this.validateAndGetProducts(items);

    // Calculate total with discount
    const total = this.calculateTotal(items, products, user);

    // Process payment
    const paymentResult = await this.paymentService.charge(user.paymentMethod, total);

    // Create order
    const order = await this.orderRepository.create({
      userId,
      items,
      total,
      paymentId: paymentResult.id,
      status: "confirmed",
    });

    // Update stock
    await this.updateStock(items);

    // Send confirmation email (async, don't wait)
    this.emailService.sendOrderConfirmation(user.email, order).catch((err) => console.error("Email failed:", err));

    return order;
  }

  async validateAndGetProducts(items) {
    const products = [];

    for (const item of items) {
      const product = await this.productRepository.findById(item.productId);

      if (!product) {
        throw new NotFoundError(`Product ${item.productId} not found`);
      }

      if (product.stock < item.quantity) {
        throw new ValidationError(`${product.name} out of stock`);
      }

      products.push(product);
    }

    return products;
  }

  calculateTotal(items, products, user) {
    let total = 0;

    for (let i = 0; i < items.length; i++) {
      total += products[i].price * items[i].quantity;
    }

    // Apply membership discount
    if (user.membershipLevel === "gold") {
      total *= 0.9;
    } else if (user.membershipLevel === "silver") {
      total *= 0.95;
    }

    return Math.round(total * 100) / 100;
  }

  async updateStock(items) {
    for (const item of items) {
      await this.productRepository.decrementStock(item.productId, item.quantity);
    }
  }

  async getOrderById(orderId) {
    const order = await this.orderRepository.findById(orderId);
    if (!order) {
      throw new NotFoundError("Order not found");
    }
    return order;
  }

  async getOrdersByUser(userId) {
    return this.orderRepository.findByUserId(userId);
  }

  async cancelOrder(orderId) {
    const order = await this.getOrderById(orderId);

    if (order.status !== "confirmed") {
      throw new ValidationError("Cannot cancel this order");
    }

    // Refund payment
    await this.paymentService.refund(order.paymentId);

    // Restore stock
    for (const item of order.items) {
      await this.productRepository.incrementStock(item.productId, item.quantity);
    }

    // Update order status
    return this.orderRepository.update(orderId, { status: "cancelled" });
  }
}

module.exports = OrderService;
```

### Controller (Thin)

```javascript
// controllers/OrderController.js
class OrderController {
  constructor(orderService) {
    this.orderService = orderService;
  }

  async create(req, res, next) {
    try {
      const { items } = req.body;
      const userId = req.user.id;

      const order = await this.orderService.createOrder(userId, items);

      res.status(201).json({ data: order });
    } catch (error) {
      next(error);
    }
  }

  async getById(req, res, next) {
    try {
      const order = await this.orderService.getOrderById(req.params.id);
      res.json({ data: order });
    } catch (error) {
      next(error);
    }
  }

  async getMyOrders(req, res, next) {
    try {
      const orders = await this.orderService.getOrdersByUser(req.user.id);
      res.json({ data: orders });
    } catch (error) {
      next(error);
    }
  }

  async cancel(req, res, next) {
    try {
      const order = await this.orderService.cancelOrder(req.params.id);
      res.json({ data: order });
    } catch (error) {
      next(error);
    }
  }
}

module.exports = OrderController;
```

## 4. Error Handling

```javascript
// errors/AppError.js
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
  }
}

class ValidationError extends AppError {
  constructor(message) {
    super(message, 400);
  }
}

class NotFoundError extends AppError {
  constructor(message) {
    super(message, 404);
  }
}

class UnauthorizedError extends AppError {
  constructor(message = "Unauthorized") {
    super(message, 401);
  }
}

// middleware/errorHandler.js
const errorHandler = (err, req, res, next) => {
  if (err.isOperational) {
    return res.status(err.statusCode).json({
      error: err.message,
    });
  }

  // Unexpected error
  console.error("ERROR:", err);
  res.status(500).json({ error: "Internal server error" });
};
```

## 5. Dependency Injection Setup

```javascript
// container.js
const OrderRepository = require("./repositories/OrderRepository");
const UserRepository = require("./repositories/UserRepository");
const ProductRepository = require("./repositories/ProductRepository");
const PaymentService = require("./services/PaymentService");
const EmailService = require("./services/EmailService");
const OrderService = require("./services/OrderService");
const OrderController = require("./controllers/OrderController");

// Repositories
const orderRepository = new OrderRepository();
const userRepository = new UserRepository();
const productRepository = new ProductRepository();

// External Services
const paymentService = new PaymentService(process.env.STRIPE_KEY);
const emailService = new EmailService(process.env.SMTP_CONFIG);

// Business Services
const orderService = new OrderService(orderRepository, userRepository, productRepository, paymentService, emailService);

// Controllers
const orderController = new OrderController(orderService);

module.exports = { orderController };
```

## 6. Routes Setup

```javascript
// routes/orders.js
const express = require("express");
const router = express.Router();
const { orderController } = require("../container");
const { authenticate } = require("../middleware/auth");

router.use(authenticate);

router.post("/", (req, res, next) => orderController.create(req, res, next));
router.get("/", (req, res, next) => orderController.getMyOrders(req, res, next));
router.get("/:id", (req, res, next) => orderController.getById(req, res, next));
router.post("/:id/cancel", (req, res, next) => orderController.cancel(req, res, next));

module.exports = router;
```

## 7. Unit Testing Service

```javascript
// __tests__/OrderService.test.js
describe("OrderService", () => {
  let orderService;
  let mockOrderRepo, mockUserRepo, mockProductRepo;
  let mockPaymentService, mockEmailService;

  beforeEach(() => {
    mockOrderRepo = { create: jest.fn(), findById: jest.fn() };
    mockUserRepo = { findById: jest.fn() };
    mockProductRepo = {
      findById: jest.fn(),
      decrementStock: jest.fn(),
    };
    mockPaymentService = { charge: jest.fn() };
    mockEmailService = { sendOrderConfirmation: jest.fn() };

    orderService = new OrderService(mockOrderRepo, mockUserRepo, mockProductRepo, mockPaymentService, mockEmailService);
  });

  describe("createOrder", () => {
    it("should create order successfully", async () => {
      // Arrange
      const user = { id: "1", membershipLevel: "gold", paymentMethod: "card" };
      const product = { id: "p1", name: "iPhone", price: 1000, stock: 10 };
      const items = [{ productId: "p1", quantity: 2 }];

      mockUserRepo.findById.mockResolvedValue(user);
      mockProductRepo.findById.mockResolvedValue(product);
      mockPaymentService.charge.mockResolvedValue({ id: "pay_123", success: true });
      mockOrderRepo.create.mockResolvedValue({ id: "order_1", items, total: 1800 });
      mockEmailService.sendOrderConfirmation.mockResolvedValue();

      // Act
      const result = await orderService.createOrder("1", items);

      // Assert
      expect(result.id).toBe("order_1");
      expect(mockPaymentService.charge).toHaveBeenCalledWith("card", 1800);
      expect(mockProductRepo.decrementStock).toHaveBeenCalledWith("p1", 2);
    });

    it("should throw when items empty", async () => {
      await expect(orderService.createOrder("1", [])).rejects.toThrow(ValidationError);
    });

    it("should throw when user not found", async () => {
      mockUserRepo.findById.mockResolvedValue(null);

      await expect(orderService.createOrder("1", [{ productId: "p1" }])).rejects.toThrow(NotFoundError);
    });

    it("should throw when product out of stock", async () => {
      mockUserRepo.findById.mockResolvedValue({ id: "1" });
      mockProductRepo.findById.mockResolvedValue({
        id: "p1",
        name: "iPhone",
        stock: 1,
      });

      await expect(orderService.createOrder("1", [{ productId: "p1", quantity: 5 }])).rejects.toThrow(ValidationError);
    });
  });
});
```

## 8. Cấu trúc thư mục hoàn chỉnh

```
src/
├── controllers/
│   ├── OrderController.js
│   ├── UserController.js
│   └── ProductController.js
├── services/
│   ├── OrderService.js
│   ├── UserService.js
│   ├── PaymentService.js
│   └── EmailService.js
├── repositories/
│   ├── OrderRepository.js
│   ├── UserRepository.js
│   └── ProductRepository.js
├── models/
│   ├── Order.js
│   ├── User.js
│   └── Product.js
├── errors/
│   └── AppError.js
├── middleware/
│   ├── auth.js
│   ├── errorHandler.js
│   └── validate.js
├── routes/
│   ├── index.js
│   ├── orders.js
│   └── users.js
├── container.js
└── app.js
```

## 9. Ưu và Nhược điểm

### Ưu điểm

- ✅ Tách biệt concerns rõ ràng
- ✅ Business logic dễ test
- ✅ Reusable across controllers
- ✅ Thin controllers, dễ maintain
- ✅ Transaction management tập trung

### Nhược điểm

- ❌ Thêm layer abstraction
- ❌ Có thể overkill cho simple CRUD
- ❌ Service có thể phình to

## 10. Khi nào dùng?

- ✅ Controller có > 50 lines logic
- ✅ Logic cần dùng ở nhiều nơi
- ✅ Cần unit test business logic
- ✅ Multiple entry points (API, CLI, Queue)
- ❌ Simple CRUD operations
- ❌ Logic chỉ dùng 1 chỗ
