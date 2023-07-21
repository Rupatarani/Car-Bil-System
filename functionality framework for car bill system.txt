// Add necessary imports

// Customer entity class
@Entity
public class Customer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String email;
    // Other customer attributes, getters, and setters
}

// Product entity class
@Entity
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private double price;
    // Other product attributes, getters, and setters
}

// Order entity class
@Entity
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @ManyToOne
    @JoinColumn(name = "customer_id")
    private Customer customer;
    @ManyToMany
    @JoinTable(name = "order_product",
            joinColumns = @JoinColumn(name = "order_id"),
            inverseJoinColumns = @JoinColumn(name = "product_id"))
    private List<Product> products;
    // Other order attributes, getters, and setters
}

// Invoice entity class
@Entity
public class Invoice {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @OneToOne
    @JoinColumn(name = "order_id")
    private Order order;
    private double amount;
    // Other invoice attributes, getters, and setters
}

// Payment entity class
@Entity
public class Payment {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @OneToOne
    @JoinColumn(name = "invoice_id")
    private Invoice invoice;
    private double amount;
    // Other payment attributes, getters, and setters
}

// Customer repository interface
@Repository
public interface CustomerRepository extends JpaRepository<Customer, Long> {
    // Custom query methods if needed
}

// Product repository interface
@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
    // Custom query methods if needed
}

// Order repository interface
@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {
    // Custom query methods if needed
}

// Invoice repository interface
@Repository
public interface InvoiceRepository extends JpaRepository<Invoice, Long> {
    // Custom query methods if needed
}

// Payment repository interface
@Repository
public interface PaymentRepository extends JpaRepository<Payment, Long> {
    // Custom query methods if needed
}

// Customer service class
@Service
public class CustomerService {
    private final CustomerRepository customerRepository;

    public CustomerService(CustomerRepository customerRepository) {
        this.customerRepository = customerRepository;
    }

    public List<Customer> getAllCustomers() {
        return customerRepository.findAll();
    }

    public Customer getCustomerById(Long id) {
        return customerRepository.findById(id).orElse(null);
    }

    public Customer saveCustomer(Customer customer) {
        return customerRepository.save(customer);
    }

    public void deleteCustomer(Long id) {
        customerRepository.deleteById(id);
    }
}

// Product service class
@Service
public class ProductService {
    private final ProductRepository productRepository;

    public ProductService(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    public List<Product> getAllProducts() {
        return productRepository.findAll();
    }

    public Product getProductById(Long id) {
        return productRepository.findById(id).orElse(null);
    }

    public Product saveProduct(Product product) {
        return productRepository.save(product);
    }

    public void deleteProduct(Long id) {
        productRepository.deleteById(id);
    }
}

// Order service class
@Service
public class OrderService {
    private final OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    public List<Order> getAllOrders() {
        return orderRepository.findAll();
    }

    public Order getOrderById(Long id) {
        return orderRepository.findById(id).orElse(null);
    }

    public Order saveOrder(Order order) {
        return orderRepository.save(order);
    }

    public void deleteOrder(Long id) {
        orderRepository.deleteById(id);
    }
}

// Invoice service class
@Service
public class InvoiceService {
    private final InvoiceRepository invoiceRepository;

    public InvoiceService(InvoiceRepository invoiceRepository) {
        this.invoiceRepository = invoiceRepository;
    }

    public Invoice generateInvoice(Order order) {
        // Calculate the invoice amount based on the order details
        double amount = order.getProducts().stream()
                .mapToDouble(Product::getPrice)
                .sum();

        Invoice invoice = new Invoice();
        invoice.setOrder(order);
        invoice.setAmount(amount);

        return invoiceRepository.save(invoice);
    }
}

// Payment service class
@Service
public class PaymentService {
    private final PaymentRepository paymentRepository;

    public PaymentService(PaymentRepository paymentRepository) {
        this.paymentRepository = paymentRepository;
    }

    public Payment processPayment(Invoice invoice, double amount) {
        // Check if the invoice amount matches the payment amount
        if (amount != invoice.getAmount()) {
            throw new IllegalArgumentException("Payment amount does not match the invoice amount.");
        }

        Payment payment = new Payment();
        payment.setInvoice(invoice);
        payment.setAmount(amount);

        return paymentRepository.save(payment);
    }
}

// Controller class for handling REST API endpoints
@RestController
@RequestMapping("/api")
public class BillingSystemController {
    private final CustomerService customerService;
    private final ProductService productService;
    private final OrderService orderService;
    private final InvoiceService invoiceService;
    private final PaymentService paymentService;

    public BillingSystemController(CustomerService customerService, ProductService productService,
                                   OrderService orderService, InvoiceService invoiceService,
                                   PaymentService paymentService) {
        this.customerService = customerService;
        this.productService = productService;
        this.orderService = orderService;
        this.invoiceService = invoiceService;
        this.paymentService = paymentService;
    }

    // Endpoint for getting all customers
    @GetMapping("/customers")
    public List<Customer> getAllCustomers() {
        return customerService.getAllCustomers();
    }

    // Endpoint for getting a specific customer by ID
    @GetMapping("/customers/{id}")
    public Customer getCustomerById(@PathVariable Long id) {
        return customerService.getCustomerById(id);
    }

    // Endpoint for creating a new customer
    @PostMapping("/customers")
    public Customer createCustomer(@RequestBody Customer customer) {
        return customerService.saveCustomer(customer);
    }

    // Endpoint for deleting a customer by ID
    @DeleteMapping("/customers/{id}")
    public void deleteCustomer(@PathVariable Long id) {
        customerService.deleteCustomer(id);
    }

    // Endpoint for getting all products
    @GetMapping("/products")
    public List<Product> getAllProducts() {
        return productService.getAllProducts();
    }

    // Endpoint for getting a specific product by ID
    @GetMapping("/products/{id}")
    public Product getProductById(@PathVariable Long id) {
        return productService.getProductById(id);
    }

    // Endpoint for creating a new product
    @PostMapping("/products")
    public Product createProduct(@RequestBody Product product) {
        returnproductService.saveProduct(product);
    }

    // Endpoint for deleting a product by ID
    @DeleteMapping("/products/{id}")
    public void deleteProduct(@PathVariable Long id) {
        productService.deleteProduct(id);
    }

    // Endpoint for creating a new order
    @PostMapping("/orders")
    public Order createOrder(@RequestBody Order order) {
        return orderService.saveOrder(order);
    }

    // Endpoint for generating an invoice for an order
    @PostMapping("/orders/{orderId}/invoices")
    public Invoice generateInvoice(@PathVariable Long orderId) {
        Order order = orderService.getOrderById(orderId);
        if (order == null) {
            throw new NotFoundException("Order not found.");
        }

        return invoiceService.generateInvoice(order);
    }

    // Endpoint for processing a payment for an invoice
    @PostMapping("/invoices/{invoiceId}/payments")
    public Payment processPayment(@PathVariable Long invoiceId, @RequestParam double amount) {
        Invoice invoice = invoiceService.getInvoiceById(invoiceId);
        if (invoice == null) {
            throw new NotFoundException("Invoice not found.");
        }

        return paymentService.processPayment(invoice, amount);
    }
}

// Main application class
@SpringBootApplication
public class BillingSystemApplication {
    public static void main(String[] args) {
        SpringApplication.run(BillingSystemApplication.class, args);
    }
}
