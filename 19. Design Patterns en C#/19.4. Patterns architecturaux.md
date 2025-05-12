# 19.4. Patterns architecturaux en C#

🔝 Retour au [Sommaire](/SOMMAIRE.md)

Les **patterns architecturaux** sont des modèles de conception à grande échelle qui définissent la structure générale d'une application. Ils organisent le code en couches et définissent comment les différentes parties communiquent entre elles.

## Qu'est-ce qu'un Pattern Architectural ?

Les patterns architecturaux vous aident à :
- **Organiser le code** de manière structurée
- **Séparer les responsabilités** clairement
- **Rendre l'application maintenable** et testable
- **Gérer la complexité** des grandes applications

## 19.4.1. MVC - Model-View-Controller

### Qu'est-ce que MVC ?

MVC est un pattern qui **sépare une application en trois composants** :
- **Model** : Les données et la logique métier
- **View** : L'interface utilisateur
- **Controller** : L'intermédiaire qui gère la logique

### Quand utiliser MVC ?

- Pour les applications web
- Quand vous voulez séparer l'affichage de la logique
- Pour faciliter les tests et la maintenance

### Exemple simple : Application de gestion de tâches

```csharp
// Model - Représente les données
public class Task
{
    public int Id { get; set; }
    public string Description { get; set; }
    public bool IsCompleted { get; set; }
    public DateTime CreatedDate { get; set; }
    public DateTime? DueDate { get; set; }

    public Task()
    {
        CreatedDate = DateTime.Now;
    }
}

// Service pour gérer les tâches (partie du Model)
public class TaskService
{
    private List<Task> tasks = new List<Task>();
    private int nextId = 1;

    public IEnumerable<Task> GetAllTasks()
    {
        return tasks.ToList();
    }

    public Task GetTaskById(int id)
    {
        return tasks.FirstOrDefault(t => t.Id == id);
    }

    public Task CreateTask(string description, DateTime? dueDate = null)
    {
        var task = new Task
        {
            Id = nextId++,
            Description = description,
            DueDate = dueDate
        };

        tasks.Add(task);
        return task;
    }

    public void UpdateTask(Task task)
    {
        var existingTask = GetTaskById(task.Id);
        if (existingTask != null)
        {
            existingTask.Description = task.Description;
            existingTask.IsCompleted = task.IsCompleted;
            existingTask.DueDate = task.DueDate;
        }
    }

    public void DeleteTask(int id)
    {
        var task = GetTaskById(id);
        if (task != null)
        {
            tasks.Remove(task);
        }
    }

    public void ToggleTaskCompletion(int id)
    {
        var task = GetTaskById(id);
        if (task != null)
        {
            task.IsCompleted = !task.IsCompleted;
        }
    }
}

// Controller - Gère la logique de l'application
[Route("api/[controller]")]
[ApiController]
public class TaskController : ControllerBase
{
    private readonly TaskService taskService;

    public TaskController(TaskService taskService)
    {
        this.taskService = taskService;
    }

    // GET: api/task
    [HttpGet]
    public ActionResult<IEnumerable<Task>> GetTasks()
    {
        return Ok(taskService.GetAllTasks());
    }

    // GET: api/task/5
    [HttpGet("{id}")]
    public ActionResult<Task> GetTask(int id)
    {
        var task = taskService.GetTaskById(id);
        if (task == null)
        {
            return NotFound();
        }
        return Ok(task);
    }

    // POST: api/task
    [HttpPost]
    public ActionResult<Task> CreateTask([FromBody] CreateTaskDto dto)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }

        var task = taskService.CreateTask(dto.Description, dto.DueDate);
        return CreatedAtAction(nameof(GetTask), new { id = task.Id }, task);
    }

    // PUT: api/task/5
    [HttpPut("{id}")]
    public IActionResult UpdateTask(int id, [FromBody] Task task)
    {
        if (id != task.Id)
        {
            return BadRequest();
        }

        var existingTask = taskService.GetTaskById(id);
        if (existingTask == null)
        {
            return NotFound();
        }

        taskService.UpdateTask(task);
        return NoContent();
    }

    // DELETE: api/task/5
    [HttpDelete("{id}")]
    public IActionResult DeleteTask(int id)
    {
        var task = taskService.GetTaskById(id);
        if (task == null)
        {
            return NotFound();
        }

        taskService.DeleteTask(id);
        return NoContent();
    }

    // POST: api/task/5/toggle
    [HttpPost("{id}/toggle")]
    public IActionResult ToggleTaskCompletion(int id)
    {
        var task = taskService.GetTaskById(id);
        if (task == null)
        {
            return NotFound();
        }

        taskService.ToggleTaskCompletion(id);
        return NoContent();
    }
}

// DTO pour la création de tâches
public class CreateTaskDto
{
    [Required]
    [StringLength(200, MinimumLength = 1)]
    public string Description { get; set; }

    public DateTime? DueDate { get; set; }
}

// View - Interface utilisateur (simplifié pour l'exemple)
public class TaskView
{
    private readonly TaskService taskService;

    public TaskView(TaskService taskService)
    {
        this.taskService = taskService;
    }

    public void DisplayAllTasks()
    {
        Console.WriteLine("=== Liste des tâches ===");
        var tasks = taskService.GetAllTasks();

        if (!tasks.Any())
        {
            Console.WriteLine("Aucune tâche");
            return;
        }

        foreach (var task in tasks)
        {
            DisplayTask(task);
        }
    }

    private void DisplayTask(Task task)
    {
        var status = task.IsCompleted ? "✓" : " ";
        var dueDate = task.DueDate?.ToString("dd/MM/yyyy") ?? "";

        Console.WriteLine($"[{status}] {task.Id}. {task.Description} {dueDate}");
    }

    public void CreateTaskInteractive()
    {
        Console.Write("Description de la tâche: ");
        var description = Console.ReadLine();

        if (string.IsNullOrWhiteSpace(description))
        {
            Console.WriteLine("La description ne peut pas être vide");
            return;
        }

        Console.Write("Date d'échéance (jj/mm/aaaa) [optionnel]: ");
        var dueDateStr = Console.ReadLine();
        DateTime? dueDate = null;

        if (DateTime.TryParseExact(dueDateStr, "dd/MM/yyyy", null,
            System.Globalization.DateTimeStyles.None, out DateTime parsed))
        {
            dueDate = parsed;
        }

        var task = taskService.CreateTask(description, dueDate);
        Console.WriteLine($"Tâche créée avec l'ID {task.Id}");
    }
}

// Utilisation
var taskService = new TaskService();
var view = new TaskView(taskService);

// Créer quelques tâches de test
taskService.CreateTask("Apprendre les design patterns", DateTime.Now.AddDays(7));
taskService.CreateTask("Terminer le projet", DateTime.Now.AddDays(14));

// Afficher les tâches
view.DisplayAllTasks();

// Interface interactive
view.CreateTaskInteractive();
```

## 19.4.2. MVVM - Model-View-ViewModel

### Qu'est-ce que MVVM ?

MVVM est un pattern pour les applications avec data binding, principalement utilisé en WPF et Xamarin :
- **Model** : Les données et la logique métier
- **View** : L'interface utilisateur (XAML)
- **ViewModel** : La logique de présentation et le binding

### Quand utiliser MVVM ?

- Pour les applications WPF, UWP, Xamarin
- Quand vous utilisez le data binding
- Pour séparer l'interface de la logique de présentation

### Exemple simple : Application de compteur

```csharp
// Model
public class Counter
{
    public int Value { get; set; }

    public void Increment()
    {
        Value++;
    }

    public void Decrement()
    {
        Value--;
    }

    public void Reset()
    {
        Value = 0;
    }
}

// Interface pour notifier les changements
public interface INotifyPropertyChanged
{
    event PropertyChangedEventHandler PropertyChanged;
}

// Base class pour ViewModel
public abstract class ViewModelBase : INotifyPropertyChanged
{
    public event PropertyChangedEventHandler PropertyChanged;

    protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }

    protected bool SetProperty<T>(ref T storage, T value, [CallerMemberName] string propertyName = null)
    {
        if (EqualityComparer<T>.Default.Equals(storage, value))
            return false;

        storage = value;
        OnPropertyChanged(propertyName);
        return true;
    }
}

// Interface pour les commandes
public interface ICommand
{
    event EventHandler CanExecuteChanged;
    bool CanExecute(object parameter);
    void Execute(object parameter);
}

// Implémentation de ICommand
public class RelayCommand : ICommand
{
    private readonly Action<object> execute;
    private readonly Func<object, bool> canExecute;

    public RelayCommand(Action<object> execute, Func<object, bool> canExecute = null)
    {
        this.execute = execute ?? throw new ArgumentNullException(nameof(execute));
        this.canExecute = canExecute;
    }

    public event EventHandler CanExecuteChanged;

    public bool CanExecute(object parameter)
    {
        return canExecute == null || canExecute(parameter);
    }

    public void Execute(object parameter)
    {
        execute(parameter);
    }

    public void RaiseCanExecuteChanged()
    {
        CanExecuteChanged?.Invoke(this, EventArgs.Empty);
    }
}

// ViewModel
public class CounterViewModel : ViewModelBase
{
    private readonly Counter counter;
    private int displayValue;

    public CounterViewModel()
    {
        counter = new Counter();
        DisplayValue = counter.Value;

        // Initialiser les commandes
        IncrementCommand = new RelayCommand(
            execute: _ => Increment(),
            canExecute: _ => DisplayValue < 100
        );

        DecrementCommand = new RelayCommand(
            execute: _ => Decrement(),
            canExecute: _ => DisplayValue > 0
        );

        ResetCommand = new RelayCommand(
            execute: _ => Reset(),
            canExecute: _ => DisplayValue != 0
        );
    }

    public int DisplayValue
    {
        get => displayValue;
        private set
        {
            if (SetProperty(ref displayValue, value))
            {
                // Notifier que la disponibilité des commandes peut avoir changé
                UpdateCommandStates();
            }
        }
    }

    public ICommand IncrementCommand { get; }
    public ICommand DecrementCommand { get; }
    public ICommand ResetCommand { get; }

    private void Increment()
    {
        counter.Increment();
        DisplayValue = counter.Value;
    }

    private void Decrement()
    {
        counter.Decrement();
        DisplayValue = counter.Value;
    }

    private void Reset()
    {
        counter.Reset();
        DisplayValue = counter.Value;
    }

    private void UpdateCommandStates()
    {
        (IncrementCommand as RelayCommand)?.RaiseCanExecuteChanged();
        (DecrementCommand as RelayCommand)?.RaiseCanExecuteChanged();
        (ResetCommand as RelayCommand)?.RaiseCanExecuteChanged();
    }
}

// Configuration pour utilisation dans une View WPF
/*
<Window x:Class="CounterApp.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Counter App" Height="300" Width="400">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>

        <TextBlock Grid.Row="0"
                   Text="{Binding DisplayValue}"
                   FontSize="48"
                   HorizontalAlignment="Center"
                   VerticalAlignment="Center"/>

        <StackPanel Grid.Row="1"
                    Orientation="Horizontal"
                    HorizontalAlignment="Center"
                    Margin="10">
            <Button Content="+"
                    Command="{Binding IncrementCommand}"
                    Margin="5" Padding="10"/>
            <Button Content="-"
                    Command="{Binding DecrementCommand}"
                    Margin="5" Padding="10"/>
            <Button Content="Reset"
                    Command="{Binding ResetCommand}"
                    Margin="5" Padding="10"/>
        </StackPanel>
    </Grid>
</Window>
*/

// Code-behind minimal pour la View
/*
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
        DataContext = new CounterViewModel();
    }
}
*/
```

## 19.4.3. Repository - "Abstraction de l'accès aux données"

### Qu'est-ce que le Repository ?

Le Repository **encapsule la logique d'accès aux données** et fournit une interface plus orientée objet pour accéder au modèle de domaine.

### Quand utiliser Repository ?

- Pour isoler la logique d'accès aux données
- Pour faciliter les tests (mocking)
- Pour changer de source de données facilement

### Exemple simple : Repository pour les produits

```csharp
// Entité
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal Price { get; set; }
    public int Stock { get; set; }
    public string Category { get; set; }
    public DateTime CreatedDate { get; set; }
    public DateTime? UpdatedDate { get; set; }

    public Product()
    {
        CreatedDate = DateTime.Now;
    }
}

// Interface Repository
public interface IProductRepository
{
    Task<IEnumerable<Product>> GetAllAsync();
    Task<Product> GetByIdAsync(int id);
    Task<IEnumerable<Product>> GetByCategoryAsync(string category);
    Task<IEnumerable<Product>> SearchAsync(string searchTerm);
    Task<Product> AddAsync(Product product);
    Task UpdateAsync(Product product);
    Task DeleteAsync(int id);
    Task<bool> ExistsAsync(int id);
}

// Implémentation en mémoire (pour démo)
public class InMemoryProductRepository : IProductRepository
{
    private readonly List<Product> products = new List<Product>();
    private int nextId = 1;

    public InMemoryProductRepository()
    {
        // Ajouter quelques produits de test
        SeedData();
    }

    private void SeedData()
    {
        var categories = new[] { "Électronique", "Vêtements", "Livres", "Sports" };
        var random = new Random();

        for (int i = 0; i < 20; i++)
        {
            products.Add(new Product
            {
                Id = nextId++,
                Name = $"Produit {i + 1}",
                Description = $"Description du produit {i + 1}",
                Price = random.Next(10, 1000),
                Stock = random.Next(0, 100),
                Category = categories[random.Next(categories.Length)]
            });
        }
    }

    public async Task<IEnumerable<Product>> GetAllAsync()
    {
        await Task.Delay(10); // Simuler un accès async
        return products.ToList();
    }

    public async Task<Product> GetByIdAsync(int id)
    {
        await Task.Delay(10);
        return products.FirstOrDefault(p => p.Id == id);
    }

    public async Task<IEnumerable<Product>> GetByCategoryAsync(string category)
    {
        await Task.Delay(10);
        return products.Where(p => p.Category.Equals(category, StringComparison.OrdinalIgnoreCase)).ToList();
    }

    public async Task<IEnumerable<Product>> SearchAsync(string searchTerm)
    {
        await Task.Delay(10);
        var term = searchTerm.ToLower();
        return products.Where(p =>
            p.Name.ToLower().Contains(term) ||
            p.Description.ToLower().Contains(term) ||
            p.Category.ToLower().Contains(term)
        ).ToList();
    }

    public async Task<Product> AddAsync(Product product)
    {
        await Task.Delay(10);
        product.Id = nextId++;
        products.Add(product);
        return product;
    }

    public async Task UpdateAsync(Product product)
    {
        await Task.Delay(10);
        var existing = await GetByIdAsync(product.Id);
        if (existing != null)
        {
            existing.Name = product.Name;
            existing.Description = product.Description;
            existing.Price = product.Price;
            existing.Stock = product.Stock;
            existing.Category = product.Category;
            existing.UpdatedDate = DateTime.Now;
        }
    }

    public async Task DeleteAsync(int id)
    {
        await Task.Delay(10);
        var product = await GetByIdAsync(id);
        if (product != null)
        {
            products.Remove(product);
        }
    }

    public async Task<bool> ExistsAsync(int id)
    {
        await Task.Delay(10);
        return products.Any(p => p.Id == id);
    }
}

// Service utilisant le Repository
public class ProductService
{
    private readonly IProductRepository repository;

    public ProductService(IProductRepository repository)
    {
        this.repository = repository;
    }

    public async Task<IEnumerable<Product>> GetLowStockProductsAsync(int threshold = 10)
    {
        var allProducts = await repository.GetAllAsync();
        return allProducts.Where(p => p.Stock <= threshold);
    }

    public async Task<IEnumerable<Product>> GetProductsByPriceRangeAsync(decimal minPrice, decimal maxPrice)
    {
        var allProducts = await repository.GetAllAsync();
        return allProducts.Where(p => p.Price >= minPrice && p.Price <= maxPrice);
    }

    public async Task<Product> CreateProductAsync(string name, string description, decimal price, int stock, string category)
    {
        var product = new Product
        {
            Name = name,
            Description = description,
            Price = price,
            Stock = stock,
            Category = category
        };

        return await repository.AddAsync(product);
    }

    public async Task<bool> UpdateStockAsync(int productId, int newStock)
    {
        var product = await repository.GetByIdAsync(productId);
        if (product == null)
        {
            return false;
        }

        product.Stock = newStock;
        await repository.UpdateAsync(product);
        return true;
    }

    public async Task<decimal> CalculateCategoryTotalValueAsync(string category)
    {
        var categoryProducts = await repository.GetByCategoryAsync(category);
        return categoryProducts.Sum(p => p.Price * p.Stock);
    }
}

// Test et utilisation
public async Task DemoRepository()
{
    IProductRepository repository = new InMemoryProductRepository();
    var productService = new ProductService(repository);

    // Récupérer tous les produits
    var allProducts = await repository.GetAllAsync();
    Console.WriteLine($"Total produits: {allProducts.Count()}");

    // Rechercher par catégorie
    var electronics = await repository.GetByCategoryAsync("Électronique");
    Console.WriteLine($"Produits électroniques: {electronics.Count()}");

    // Rechercher avec terme
    var searchResults = await repository.SearchAsync("produit 1");
    Console.WriteLine($"Résultats recherche: {searchResults.Count()}");

    // Créer un nouveau produit
    var newProduct = await productService.CreateProductAsync(
        "Nouvel ordinateur",
        "Un ordinateur puissant",
        1200,
        5,
        "Électronique"
    );
    Console.WriteLine($"Produit créé: {newProduct.Id} - {newProduct.Name}");

    // Produits avec stock bas
    var lowStock = await productService.GetLowStockProductsAsync(5);
    Console.WriteLine($"Produits avec stock bas: {lowStock.Count()}");

    // Valeur totale d'une catégorie
    var categoryValue = await productService.CalculateCategoryTotalValueAsync("Électronique");
    Console.WriteLine($"Valeur totale électronique: {categoryValue:C}");
}
```

### Exemple avancé : Repository avec Entity Framework

```csharp
// Configuration Entity Framework
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    public DbSet<Product> Products { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Product>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Name).IsRequired().HasMaxLength(200);
            entity.Property(e => e.Description).HasMaxLength(1000);
            entity.Property(e => e.Price).HasColumnType("decimal(18,2)");
            entity.Property(e => e.Category).HasMaxLength(100);
            entity.HasIndex(e => e.Category);
        });
    }
}

// Repository EF Core
public class EFProductRepository : IProductRepository
{
    private readonly ApplicationDbContext context;

    public EFProductRepository(ApplicationDbContext context)
    {
        this.context = context;
    }

    public async Task<IEnumerable<Product>> GetAllAsync()
    {
        return await context.Products.ToListAsync();
    }

    public async Task<Product> GetByIdAsync(int id)
    {
        return await context.Products.FindAsync(id);
    }

    public async Task<IEnumerable<Product>> GetByCategoryAsync(string category)
    {
        return await context.Products
            .Where(p => p.Category == category)
            .ToListAsync();
    }

    public async Task<IEnumerable<Product>> SearchAsync(string searchTerm)
    {
        return await context.Products
            .Where(p =>
                EF.Functions.Like(p.Name, $"%{searchTerm}%") ||
                EF.Functions.Like(p.Description, $"%{searchTerm}%") ||
                EF.Functions.Like(p.Category, $"%{searchTerm}%"))
            .ToListAsync();
    }

    public async Task<Product> AddAsync(Product product)
    {
        context.Products.Add(product);
        await context.SaveChangesAsync();
        return product;
    }

    public async Task UpdateAsync(Product product)
    {
        context.Entry(product).State = EntityState.Modified;
        await context.SaveChangesAsync();
    }

    public async Task DeleteAsync(int id)
    {
        var product = await context.Products.FindAsync(id);
        if (product != null)
        {
            context.Products.Remove(product);
            await context.SaveChangesAsync();
        }
    }

    public async Task<bool> ExistsAsync(int id)
    {
        return await context.Products.AnyAsync(p => p.Id == id);
    }
}

// Configuration DI dans Startup.cs
/*
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));

    services.AddScoped<IProductRepository, EFProductRepository>();
    services.AddScoped<ProductService>();

    services.AddControllers();
}
*/
```

## 19.4.4. Unit of Work - "Transaction logique"

### Qu'est-ce que l'Unit of Work ?

L'Unit of Work **maintient une liste des objets affectés** par une transaction et coordonne l'écriture des changements.

### Quand utiliser Unit of Work ?

- Pour gérer des transactions complexes
- Pour optimiser les accès à la base de données
- Pour garantir la cohérence des données

### Exemple simple : Unit of Work

```csharp
// Interface pour les entités
public interface IEntity
{
    int Id { get; set; }
}

// Order entité
public class Order : IEntity
{
    public int Id { get; set; }
    public string CustomerName { get; set; }
    public DateTime OrderDate { get; set; }
    public decimal TotalAmount { get; set; }
    public ICollection<OrderItem> Items { get; set; } = new List<OrderItem>();

    public Order()
    {
        OrderDate = DateTime.Now;
    }
}

// OrderItem entité
public class OrderItem : IEntity
{
    public int Id { get; set; }
    public int OrderId { get; set; }
    public string ProductName { get; set; }
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
    public decimal TotalPrice => Quantity * UnitPrice;
}

// Repositories
public interface IOrderRepository
{
    Task<Order> GetByIdAsync(int id);
    Task<IEnumerable<Order>> GetByCustomerAsync(string customerName);
    void Add(Order order);
    void Update(Order order);
    void Delete(Order order);
}

public interface IOrderItemRepository
{
    Task<IEnumerable<OrderItem>> GetByOrderIdAsync(int orderId);
    void Add(OrderItem item);
    void Update(OrderItem item);
    void Delete(OrderItem item);
}

// Interface Unit of Work
public interface IUnitOfWork : IDisposable
{
    IOrderRepository Orders { get; }
    IOrderItemRepository OrderItems { get; }

    Task<int> SaveChangesAsync();
    Task BeginTransactionAsync();
    Task CommitTransactionAsync();
    Task RollbackTransactionAsync();
}

// Implémentation Entity Framework
public class EFUnitOfWork : IUnitOfWork
{
    private readonly ApplicationDbContext context;
    private IDbContextTransaction transaction;

    // Repositories lazy loaded
    private IOrderRepository orderRepository;
    private IOrderItemRepository orderItemRepository;

    public EFUnitOfWork(ApplicationDbContext context)
    {
        this.context = context;
    }

    public IOrderRepository Orders
    {
        get
        {
            return orderRepository ??= new EFOrderRepository(context);
        }
    }

    public IOrderItemRepository OrderItems
    {
        get
        {
            return orderItemRepository ??= new EFOrderItemRepository(context);
        }
    }

    public async Task<int> SaveChangesAsync()
    {
        return await context.SaveChangesAsync();
    }

    public async Task BeginTransactionAsync()
    {
        transaction = await context.Database.BeginTransactionAsync();
    }

    public async Task CommitTransactionAsync()
    {
        try
        {
            await SaveChangesAsync();
            await transaction?.CommitAsync();
        }
        catch
        {
            await RollbackTransactionAsync();
            throw;
        }
    }

    public async Task RollbackTransactionAsync()
    {
        await transaction?.RollbackAsync();
        transaction?.Dispose();
        transaction = null;
    }

    public void Dispose()
    {
        transaction?.Dispose();
        context?.Dispose();
    }
}

// Service utilisant Unit of Work
public class OrderService
{
    private readonly IUnitOfWork unitOfWork;

    public OrderService(IUnitOfWork unitOfWork)
    {
        this.unitOfWork = unitOfWork;
    }

    public async Task<Order> CreateOrderWithItemsAsync(string customerName, List<(string ProductName, int Quantity, decimal UnitPrice)> items)
    {
        await unitOfWork.BeginTransactionAsync();

        try
        {
            // Créer la commande
            var order = new Order
            {
                CustomerName = customerName,
                TotalAmount = 0
            };

            unitOfWork.Orders.Add(order);
            await unitOfWork.SaveChangesAsync();  // Pour obtenir l'ID

            // Ajouter les items
            decimal total = 0;
            foreach (var (productName, quantity, unitPrice) in items)
            {
                var item = new OrderItem
                {
                    OrderId = order.Id,
                    ProductName = productName,
                    Quantity = quantity,
                    UnitPrice = unitPrice
                };

                unitOfWork.OrderItems.Add(item);
                total += item.TotalPrice;
            }

            // Mettre à jour le total
            order.TotalAmount = total;
            unitOfWork.Orders.Update(order);

            await unitOfWork.CommitTransactionAsync();

            return order;
        }
        catch
        {
            await unitOfWork.RollbackTransactionAsync();
            throw;
        }
    }

public async Task<bool> UpdateOrderTotalAsync(int orderId)
{
    await unitOfWork.BeginTransactionAsync();

    try
    {
        var order = await unitOfWork.Orders.GetByIdAsync(orderId);
        if (order == null)
        {
            return false;
        }

        var items = await unitOfWork.OrderItems.GetByOrderIdAsync(orderId);
        order.TotalAmount = items.Sum(item => item.TotalPrice);

        unitOfWork.Orders.Update(order);
        await unitOfWork.CommitTransactionAsync();

        return true;
    }
    catch
    {
        await unitOfWork.RollbackTransactionAsync();
        throw;
    }
}

    public async Task<bool> CancelOrderAsync(int orderId)
    {
        await unitOfWork.BeginTransactionAsync();

        try
        {
            var order = await unitOfWork.Orders.GetByIdAsync(orderId);
            if (order == null)
            {
                return false;
            }

            // Supprimer tous les items
            var items = await unitOfWork.OrderItems.GetByOrderIdAsync(orderId);
            foreach (var item in items)
            {
                unitOfWork.OrderItems.Delete(item);
            }

            // Supprimer la commande
            unitOfWork.Orders.Delete(order);

            await unitOfWork.CommitTransactionAsync();

            return true;
        }
        catch
        {
            await unitOfWork.RollbackTransactionAsync();
            throw;
        }
    }
}

// Utilisation
public async Task DemoUnitOfWork()
{
    using var unitOfWork = serviceProvider.GetService<IUnitOfWork>();
    var orderService = new OrderService(unitOfWork);

    // Créer une commande avec plusieurs items
    var items = new List<(string, int, decimal)>
    {
        ("Ordinateur", 1, 999.99m),
        ("Souris", 2, 29.99m),
        ("Clavier", 1, 79.99m)
    };

    var order = await orderService.CreateOrderWithItemsAsync("Jean Dupont", items);
    Console.WriteLine($"Commande créée: {order.Id}, Total: {order.TotalAmount:C}");

    // Annuler la commande
    bool cancelled = await orderService.CancelOrderAsync(order.Id);
    Console.WriteLine($"Commande annulée: {cancelled}");
}
```

## 19.4.5. Dependency Injection - "Inversion de contrôle"

### Qu'est-ce que la Dependency Injection ?

La Dependency Injection est une technique pour **fournir les dépendances à un objet** plutôt que de le laisser les créer lui-même.

### Quand utiliser DI ?

- Pour rendre le code testable
- Pour gérer le cycle de vie des objets
- Pour configurer des implémentations différentes selon l'environnement

### Exemple simple : System de notification

```csharp
// Abstraction
public interface INotificationService
{
    Task SendAsync(string recipient, string subject, string message);
}

// Implémentations concrètes
public class EmailNotificationService : INotificationService
{
    private readonly IConfiguration configuration;
    private readonly ILogger<EmailNotificationService> logger;

    public EmailNotificationService(IConfiguration configuration, ILogger<EmailNotificationService> logger)
    {
        this.configuration = configuration;
        this.logger = logger;
    }

    public async Task SendAsync(string recipient, string subject, string message)
    {
        var smtpSettings = configuration.GetSection("SmtpSettings");

        logger.LogInformation($"Sending email to {recipient}: {subject}");

        // Simuler l'envoi d'email
        using var client = new SmtpClient(smtpSettings["Host"], int.Parse(smtpSettings["Port"]));
        // Configuration SMTP...

        await Task.Delay(100); // Simuler l'envoi
        logger.LogInformation("Email sent successfully");
    }
}

public class SmsNotificationService : INotificationService
{
    private readonly IConfiguration configuration;
    private readonly ILogger<SmsNotificationService> logger;

    public SmsNotificationService(IConfiguration configuration, ILogger<SmsNotificationService> logger)
    {
        this.configuration = configuration;
        this.logger = logger;
    }

    public async Task SendAsync(string recipient, string subject, string message)
    {
        var smsSettings = configuration.GetSection("SmsSettings");

        logger.LogInformation($"Sending SMS to {recipient}: {subject}");

        // Simuler l'envoi de SMS
        await Task.Delay(50);
        logger.LogInformation("SMS sent successfully");
    }
}

// Service utilisant les dépendances
public interface IUserService
{
    Task<User> RegisterUserAsync(string email, string password);
    Task<bool> SendWelcomeMessageAsync(User user);
}

public class UserService : IUserService
{
    private readonly IUserRepository userRepository;
    private readonly IPasswordHasher passwordHasher;
    private readonly INotificationService notificationService;
    private readonly ILogger<UserService> logger;

    public UserService(
        IUserRepository userRepository,
        IPasswordHasher passwordHasher,
        INotificationService notificationService,
        ILogger<UserService> logger)
    {
        this.userRepository = userRepository;
        this.passwordHasher = passwordHasher;
        this.notificationService = notificationService;
        this.logger = logger;
    }

    public async Task<User> RegisterUserAsync(string email, string password)
    {
        logger.LogInformation($"Registering user: {email}");

        var hashedPassword = passwordHasher.Hash(password);

        var user = new User
        {
            Email = email,
            PasswordHash = hashedPassword,
            CreatedDate = DateTime.UtcNow,
            IsActive = true
        };

        await userRepository.AddAsync(user);
        await SendWelcomeMessageAsync(user);

        return user;
    }

    public async Task<bool> SendWelcomeMessageAsync(User user)
    {
        try
        {
            await notificationService.SendAsync(
                user.Email,
                "Bienvenue dans notre application!",
                $"Bonjour {user.Email}, merci de vous être inscrit!"
            );

            return true;
        }
        catch (Exception ex)
        {
            logger.LogError(ex, "Failed to send welcome message to {Email}", user.Email);
            return false;
        }
    }
}

// Configuration dans Startup.cs ou Program.cs
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // Configuration de base de données
        services.AddDbContext<ApplicationDbContext>(options =>
            options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));

        // Repositories
        services.AddScoped<IUserRepository, EFUserRepository>();

        // Services
        services.AddScoped<IPasswordHasher, BCryptPasswordHasher>();

        // Services de notification - choisir selon la configuration
        if (Configuration.GetValue<bool>("UseEmailNotification"))
        {
            services.AddScoped<INotificationService, EmailNotificationService>();
        }
        else
        {
            services.AddScoped<INotificationService, SmsNotificationService>();
        }

        services.AddScoped<IUserService, UserService>();

        // Logging
        services.AddLogging();

        // API
        services.AddControllers();
    }
}
```

### Exemple avancé : Factory Pattern avec DI

```csharp
// Enumération pour les types de notification
public enum NotificationType
{
    Email,
    Sms,
    Push,
    Slack
}

// Factory pour créer les services de notification
public interface INotificationServiceFactory
{
    INotificationService Create(NotificationType type);
}

public class NotificationServiceFactory : INotificationServiceFactory
{
    private readonly IServiceProvider serviceProvider;

    public NotificationServiceFactory(IServiceProvider serviceProvider)
    {
        this.serviceProvider = serviceProvider;
    }

    public INotificationService Create(NotificationType type)
    {
        return type switch
        {
            NotificationType.Email => serviceProvider.GetService<EmailNotificationService>(),
            NotificationType.Sms => serviceProvider.GetService<SmsNotificationService>(),
            NotificationType.Push => serviceProvider.GetService<PushNotificationService>(),
            NotificationType.Slack => serviceProvider.GetService<SlackNotificationService>(),
            _ => throw new ArgumentException($"Invalid notification type: {type}")
        };
    }
}

// Service qui utilise la factory
public class MultiChannelNotificationService
{
    private readonly INotificationServiceFactory factory;
    private readonly IUserPreferencesRepository preferencesRepository;

    public MultiChannelNotificationService(
        INotificationServiceFactory factory,
        IUserPreferencesRepository preferencesRepository)
    {
        this.factory = factory;
        this.preferencesRepository = preferencesRepository;
    }

    public async Task SendNotificationAsync(string userId, NotificationContent content)
    {
        var preferences = await preferencesRepository.GetUserPreferencesAsync(userId);

        // Envoyer selon les préférences de l'utilisateur
        foreach (var preferredType in preferences.PreferredChannels)
        {
            var notificationService = factory.Create(preferredType);
            await notificationService.SendAsync(preferences.ContactInfo[preferredType], content.Subject, content.Body);
        }
    }
}

// Configuration
services.AddScoped<EmailNotificationService>();
services.AddScoped<SmsNotificationService>();
services.AddScoped<PushNotificationService>();
services.AddScoped<SlackNotificationService>();
services.AddScoped<INotificationServiceFactory, NotificationServiceFactory>();
services.AddScoped<MultiChannelNotificationService>();
```

## 19.4.6. CQRS - Command Query Responsibility Segregation

### Qu'est-ce que CQRS ?

CQRS **sépare les opérations de lecture (Query) des opérations d'écriture (Command)**, permettant d'optimiser chacune indépendamment.

### Quand utiliser CQRS ?

- Pour des applications avec des besoins très différents en lecture/écriture
- Pour optimiser la performance
- Pour simplifier des modèles complexes

### Exemple simple : Système de commandes

```csharp
// Séparation des modèles
// Write Model (Commands)
public class Order
{
    public int Id { get; private set; }
    public string CustomerId { get; private set; }
    public DateTime OrderDate { get; private set; }
    public OrderStatus Status { get; private set; }
    private readonly List<OrderLine> lines = new List<OrderLine>();

    public IReadOnlyList<OrderLine> Lines => lines.AsReadOnly();

    // Constructor for creation
    private Order() { } // For EF

    public Order(string customerId)
    {
        CustomerId = customerId;
        OrderDate = DateTime.UtcNow;
        Status = OrderStatus.Draft;
    }

    public void AddLine(string productId, int quantity, decimal unitPrice)
    {
        if (Status != OrderStatus.Draft)
            throw new InvalidOperationException("Cannot modify non-draft order");

        var existingLine = lines.FirstOrDefault(l => l.ProductId == productId);
        if (existingLine != null)
        {
            existingLine.UpdateQuantity(existingLine.Quantity + quantity);
        }
        else
        {
            lines.Add(new OrderLine(productId, quantity, unitPrice));
        }
    }

    public void Submit()
    {
        if (Status != OrderStatus.Draft)
            throw new InvalidOperationException("Only draft orders can be submitted");

        if (!lines.Any())
            throw new InvalidOperationException("Cannot submit empty order");

        Status = OrderStatus.Submitted;
    }

    public decimal CalculateTotal()
    {
        return lines.Sum(l => l.Total);
    }
}

// Read Model (Queries)
public class OrderSummaryDto
{
    public int Id { get; set; }
    public string CustomerName { get; set; }
    public DateTime OrderDate { get; set; }
    public string Status { get; set; }
    public decimal Total { get; set; }
    public int ItemCount { get; set; }
}

public class OrderDetailsDto
{
    public int Id { get; set; }
    public string CustomerName { get; set; }
    public string CustomerEmail { get; set; }
    public DateTime OrderDate { get; set; }
    public string Status { get; set; }
    public decimal Total { get; set; }
    public List<OrderLineDto> Lines { get; set; }
}

// Commands
public interface ICommand
{
    DateTime Timestamp { get; }
}

public class CreateOrderCommand : ICommand
{
    public DateTime Timestamp { get; } = DateTime.UtcNow;
    public string CustomerId { get; set; }
}

public class AddOrderLineCommand : ICommand
{
    public DateTime Timestamp { get; } = DateTime.UtcNow;
    public int OrderId { get; set; }
    public string ProductId { get; set; }
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
}

public class SubmitOrderCommand : ICommand
{
    public DateTime Timestamp { get; } = DateTime.UtcNow;
    public int OrderId { get; set; }
}

// Command Handlers
public interface ICommandHandler<TCommand> where TCommand : ICommand
{
    Task HandleAsync(TCommand command);
}

public class CreateOrderCommandHandler : ICommandHandler<CreateOrderCommand>
{
    private readonly IOrderRepository repository;
    private readonly IEventPublisher eventPublisher;

    public CreateOrderCommandHandler(IOrderRepository repository, IEventPublisher eventPublisher)
    {
        this.repository = repository;
        this.eventPublisher = eventPublisher;
    }

    public async Task HandleAsync(CreateOrderCommand command)
    {
        var order = new Order(command.CustomerId);
        await repository.SaveAsync(order);

        await eventPublisher.PublishAsync(new OrderCreatedEvent
        {
            OrderId = order.Id,
            CustomerId = order.CustomerId,
            OrderDate = order.OrderDate
        });
    }
}

public class SubmitOrderCommandHandler : ICommandHandler<SubmitOrderCommand>
{
    private readonly IOrderRepository repository;
    private readonly IEventPublisher eventPublisher;

    public SubmitOrderCommandHandler(IOrderRepository repository, IEventPublisher eventPublisher)
    {
        this.repository = repository;
        this.eventPublisher = eventPublisher;
    }

    public async Task HandleAsync(SubmitOrderCommand command)
    {
        var order = await repository.GetByIdAsync(command.OrderId);
        if (order == null)
            throw new OrderNotFoundException(command.OrderId);

        order.Submit();
        await repository.SaveAsync(order);

        await eventPublisher.PublishAsync(new OrderSubmittedEvent
        {
            OrderId = order.Id,
            Total = order.CalculateTotal(),
            SubmittedAt = DateTime.UtcNow
        });
    }
}

// Queries
public interface IQuery<TResult>
{
}

public class GetOrdersByCustomerQuery : IQuery<IEnumerable<OrderSummaryDto>>
{
    public string CustomerId { get; set; }
    public DateTime? FromDate { get; set; }
    public DateTime? ToDate { get; set; }
}

public class GetOrderDetailsQuery : IQuery<OrderDetailsDto>
{
    public int OrderId { get; set; }
}

// Query Handlers
public interface IQueryHandler<TQuery, TResult> where TQuery : IQuery<TResult>
{
    Task<TResult> HandleAsync(TQuery query);
}

public class GetOrdersByCustomerQueryHandler : IQueryHandler<GetOrdersByCustomerQuery, IEnumerable<OrderSummaryDto>>
{
    private readonly IOrderReadModelRepository repository;

    public GetOrdersByCustomerQueryHandler(IOrderReadModelRepository repository)
    {
        this.repository = repository;
    }

    public async Task<IEnumerable<OrderSummaryDto>> HandleAsync(GetOrdersByCustomerQuery query)
    {
        return await repository.GetOrderSummariesByCustomerAsync(
            query.CustomerId,
            query.FromDate,
            query.ToDate);
    }
}

// Dispatcher pour coordonner les handlers
public interface ICommandDispatcher
{
    Task DispatchAsync<TCommand>(TCommand command) where TCommand : ICommand;
}

public interface IQueryDispatcher
{
    Task<TResult> DispatchAsync<TResult>(IQuery<TResult> query);
}

public class Dispatcher : ICommandDispatcher, IQueryDispatcher
{
    private readonly IServiceProvider serviceProvider;

    public Dispatcher(IServiceProvider serviceProvider)
    {
        this.serviceProvider = serviceProvider;
    }

    public async Task DispatchAsync<TCommand>(TCommand command) where TCommand : ICommand
    {
        var handler = serviceProvider.GetService<ICommandHandler<TCommand>>();
        if (handler == null)
            throw new InvalidOperationException($"No handler found for {typeof(TCommand).Name}");

        await handler.HandleAsync(command);
    }

    public async Task<TResult> DispatchAsync<TResult>(IQuery<TResult> query)
    {
        var handlerType = typeof(IQueryHandler<,>).MakeGenericType(query.GetType(), typeof(TResult));
        dynamic handler = serviceProvider.GetService(handlerType);

        if (handler == null)
            throw new InvalidOperationException($"No handler found for {query.GetType().Name}");

        return await handler.HandleAsync((dynamic)query);
    }
}

// Utilisation
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly ICommandDispatcher commandDispatcher;
    private readonly IQueryDispatcher queryDispatcher;

    public OrdersController(ICommandDispatcher commandDispatcher, IQueryDispatcher queryDispatcher)
    {
        this.commandDispatcher = commandDispatcher;
        this.queryDispatcher = queryDispatcher;
    }

    // Command endpoint
    [HttpPost]
    public async Task<IActionResult> CreateOrder([FromBody] CreateOrderCommand command)
    {
        await commandDispatcher.DispatchAsync(command);
        return Accepted();
    }

    // Query endpoint
    [HttpGet("customer/{customerId}")]
    public async Task<IActionResult> GetCustomerOrders(string customerId)
    {
        var query = new GetOrdersByCustomerQuery { CustomerId = customerId };
        var result = await queryDispatcher.DispatchAsync(query);
        return Ok(result);
    }
}
```

## 19.4.7. Event Sourcing - "Stocker les événements"

### Qu'est-ce que l'Event Sourcing ?

Event Sourcing **stocke tous les changements d'état comme une séquence d'événements**, permettant de reconstruire l'état actuel et les états historiques.

### Quand utiliser Event Sourcing ?

- Pour un audit complet des changements
- Pour pouvoir reconstruire l'historique
- Pour implémenter des fonctionnalités de temps-réel
- Pour des domaines complexes nécessitant une traçabilité

### Exemple simple : Compte bancaire

```csharp
// Événements
public abstract class DomainEvent
{
    public DateTime OccurredAt { get; }
    public string AggregateId { get; }

    protected DomainEvent(string aggregateId)
    {
        AggregateId = aggregateId;
        OccurredAt = DateTime.UtcNow;
    }
}

public class AccountCreatedEvent : DomainEvent
{
    public string AccountHolder { get; }
    public decimal InitialBalance { get; }

    public AccountCreatedEvent(string aggregateId, string accountHolder, decimal initialBalance)
        : base(aggregateId)
    {
        AccountHolder = accountHolder;
        InitialBalance = initialBalance;
    }
}

public class MoneyDepositedEvent : DomainEvent
{
    public decimal Amount { get; }
    public string Description { get; }

    public MoneyDepositedEvent(string aggregateId, decimal amount, string description)
        : base(aggregateId)
    {
        Amount = amount;
        Description = description;
    }
}

public class MoneyWithdrawnEvent : DomainEvent
{
    public decimal Amount { get; }
    public string Description { get; }

    public MoneyWithdrawnEvent(string aggregateId, decimal amount, string description)
        : base(aggregateId)
    {
        Amount = amount;
        Description = description;
    }
}

public class AccountClosedEvent : DomainEvent
{
    public string Reason { get; }

    public AccountClosedEvent(string aggregateId, string reason)
        : base(aggregateId)
    {
        Reason = reason;
    }
}

// Aggregate
public class BankAccount
{
    public string Id { get; private set; }
    public string AccountHolder { get; private set; }
    public decimal Balance { get; private set; }
    public bool IsActive { get; private set; }
    public List<DomainEvent> Changes { get; private set; } = new List<DomainEvent>();

    // Pour la reconstruction depuis les événements
    private BankAccount()
    {
    }

    // Constructor pour création
    public BankAccount(string accountId, string accountHolder, decimal initialDeposit)
    {
        if (initialDeposit < 0)
            throw new ArgumentException("Initial deposit cannot be negative");

        var createdEvent = new AccountCreatedEvent(accountId, accountHolder, initialDeposit);
        Apply(createdEvent);
        Changes.Add(createdEvent);
    }

    // Méthodes business
    public void Deposit(decimal amount, string description = null)
    {
        if (!IsActive)
            throw new InvalidOperationException("Account is closed");

        if (amount <= 0)
            throw new ArgumentException("Deposit amount must be positive");

        var depositEvent = new MoneyDepositedEvent(Id, amount, description);
        Apply(depositEvent);
        Changes.Add(depositEvent);
    }

    public void Withdraw(decimal amount, string description = null)
    {
        if (!IsActive)
            throw new InvalidOperationException("Account is closed");

        if (amount <= 0)
            throw new ArgumentException("Withdrawal amount must be positive");

        if (Balance < amount)
            throw new InvalidOperationException("Insufficient funds");

        var withdrawnEvent = new MoneyWithdrawnEvent(Id, amount, description);
        Apply(withdrawnEvent);
        Changes.Add(withdrawnEvent);
    }

    public void Close(string reason)
    {
        if (!IsActive)
            throw new InvalidOperationException("Account is already closed");

        var closedEvent = new AccountClosedEvent(Id, reason);
        Apply(closedEvent);
        Changes.Add(closedEvent);
    }

    // Reconstruction depuis les événements
    public static BankAccount FromEvents(IEnumerable<DomainEvent> events)
    {
        var account = new BankAccount();
        foreach (var @event in events)
        {
            account.Apply(@event);
        }
        return account;
    }

    // Application des événements
    private void Apply(DomainEvent @event)
    {
        switch (@event)
        {
            case AccountCreatedEvent created:
                Id = created.AggregateId;
                AccountHolder = created.AccountHolder;
                Balance = created.InitialBalance;
                IsActive = true;
                break;

            case MoneyDepositedEvent deposited:
                Balance += deposited.Amount;
                break;

            case MoneyWithdrawnEvent withdrawn:
                Balance -= withdrawn.Amount;
                break;

            case AccountClosedEvent closed:
                IsActive = false;
                break;
        }
    }

    // Nettoyer les changements après sauvegarde
    public void ClearChanges()
    {
        Changes.Clear();
    }
}

// Event Store
public interface IEventStore
{
    Task SaveEventsAsync(string aggregateId, IEnumerable<DomainEvent> events, int expectedVersion);
    Task<IEnumerable<DomainEvent>> GetEventsAsync(string aggregateId);
    Task<IEnumerable<DomainEvent>> GetEventsAsync(string aggregateId, int fromVersion);
}

// Repository pour les aggregates
public class BankAccountRepository
{
    private readonly IEventStore eventStore;

    public BankAccountRepository(IEventStore eventStore)
    {
        this.eventStore = eventStore;
    }

    public async Task<BankAccount> GetByIdAsync(string accountId)
    {
        var events = await eventStore.GetEventsAsync(accountId);

        if (!events.Any())
            return null;

        return BankAccount.FromEvents(events);
    }

    public async Task SaveAsync(BankAccount account)
    {
        var uncommittedEvents = account.Changes;

        if (!uncommittedEvents.Any())
            return;

        // Pour simplification, on n'utilise pas de version ici
        await eventStore.SaveEventsAsync(account.Id, uncommittedEvents, -1);
        account.ClearChanges();
    }
}

// Projection pour les vues
public class AccountBalanceProjection
{
    public string AccountId { get; set; }
    public string AccountHolder { get; set; }
    public decimal Balance { get; set; }
    public bool IsActive { get; set; }
    public DateTime LastUpdated { get; set; }
}

// Handler pour mettre à jour les projections
public class AccountBalanceProjectionHandler :
    IEventHandler<AccountCreatedEvent>,
    IEventHandler<MoneyDepositedEvent>,
    IEventHandler<MoneyWithdrawnEvent>,
    IEventHandler<AccountClosedEvent>
{
    private readonly IAccountBalanceProjectionRepository repository;

    public AccountBalanceProjectionHandler(IAccountBalanceProjectionRepository repository)
    {
        this.repository = repository;
    }

    public async Task Handle(AccountCreatedEvent @event)
    {
        var projection = new AccountBalanceProjection
        {
            AccountId = @event.AggregateId,
            AccountHolder = @event.AccountHolder,
            Balance = @event.InitialBalance,
            IsActive = true,
            LastUpdated = @event.OccurredAt
        };

        await repository.SaveAsync(projection);
    }

    public async Task Handle(MoneyDepositedEvent @event)
    {
        var projection = await repository.GetByIdAsync(@event.AggregateId);
        if (projection != null)
        {
            projection.Balance += @event.Amount;
            projection.LastUpdated = @event.OccurredAt;
            await repository.SaveAsync(projection);
        }
    }

    public async Task Handle(MoneyWithdrawnEvent @event)
    {
        var projection = await repository.GetByIdAsync(@event.AggregateId);
        if (projection != null)
        {
            projection.Balance -= @event.Amount;
            projection.LastUpdated = @event.OccurredAt;
            await repository.SaveAsync(projection);
        }
    }

    public async Task Handle(AccountClosedEvent @event)
    {
        var projection = await repository.GetByIdAsync(@event.AggregateId);
        if (projection != null)
        {
            projection.IsActive = false;
            projection.LastUpdated = @event.OccurredAt;
            await repository.SaveAsync(projection);
        }
    }
}

// Utilisation
public async Task DemoEventSourcing()
{
    var eventStore = new InMemoryEventStore();
    var repository = new BankAccountRepository(eventStore);

    // Créer un compte
    var account1 = new BankAccount("ACC001", "Jean Dupont", 1000m);
    await repository.SaveAsync(account1);

    // Faire des transactions
    account1.Deposit(500m, "Salaire");
    account1.Withdraw(200m, "Courses");
    account1.Withdraw(100m, "Restaurant");
    await repository.SaveAsync(account1);

    // Récupérer le compte depuis les événements
    var loadedAccount = await repository.GetByIdAsync("ACC001");
    Console.WriteLine($"Balance: {loadedAccount.Balance}");

    // Afficher l'historique des événements
    var events = await eventStore.GetEventsAsync("ACC001");
    foreach (var @event in events)
    {
        Console.WriteLine($"{@event.GetType().Name} - {FormatEvent(@event)}");
    }
}

private static string FormatEvent(DomainEvent @event)
{
    return @event switch
    {
        AccountCreatedEvent created => $"Created for {created.AccountHolder} with {created.InitialBalance:C}",
        MoneyDepositedEvent deposited => $"Deposited {deposited.Amount:C} - {deposited.Description}",
        MoneyWithdrawnEvent withdrawn => $"Withdrawn {withdrawn.Amount:C} - {withdrawn.Description}",
        AccountClosedEvent closed => $"Closed - {closed.Reason}",
        _ => @event.ToString()
    };
}
```

## Résumé des Patterns Architecturaux

| Pattern | But principal | Cas d'utilisation typique |
|---------|--------------|--------------------------|
| **MVC** | Séparation interface/logique | Applications web |
| **MVVM** | Binding et présentation | Applications WPF/Xamarin |
| **Repository** | Abstraction accès données | Isolation couche données |
| **Unit of Work** | Transaction business | Opérations multi-tables |
| **Dependency Injection** | Inversion du contrôle | Testabilité, configuration |
| **CQRS** | Séparation lecture/écriture | Optimisation performance |
| **Event Sourcing** | Historique des changements | Audit, reconstruction |

## Bonnes pratiques

### 1. MVC
- Garde les controllers minces
- Place la logique business dans des services
- Utilise des ViewModels pour la présentation

### 2. MVVM
- Évite le code dans les views
- Utilise les commandes pour les actions
- Implémenta INotifyPropertyChanged correctement

### 3. Repository/UoW
- Garde les repositories focalisés sur l'accès aux données
- Un repository par aggregate
- Utilise Unit of Work pour les transactions complexes
- Évite les requêtes N+1
- Implémente des interfaces pour faciliter les tests

### 4. Dependency Injection
- Configure la durée de vie appropriée (Singleton, Scoped, Transient)
- Évite les dépendances circulaires
- Utilise des interfaces pour découpler
- Ne résoud pas les services dans les constructeurs
- Documente les dépendances transitives

### 5. CQRS
- Sépare clairement les modèles de lecture/écriture
- Optimise chaque côté selon ses besoins
- Utilise des événements pour synchroniser
- Évite la sur-ingénierie pour les cas simples
- Documente la cohérence éventuelle

### 6. Event Sourcing
- Rend les événements immuables
- Stocke les événements dans l'ordre
- Implémente des projections pour les requêtes
- Planifie la gestion des snapshots
- Gère les migrations d'événements

## Exercices pratiques

### Exercice 1 : MVC Application
Créez une application de gestion de bibliothèque avec :
- Modèles pour Livre, Auteur, Emprunt
- Controllers pour chaque entité
- Views pour les opérations CRUD

### Exercice 2 : MVVM Calculator
Développez une calculatrice en WPF avec :
- ViewModel gérant la logique
- Commands pour les opérations
- Binding pour l'affichage
- Support de l'historique

### Exercice 3 : Repository avec EF Core
Implémentez un système de blog avec :
- Repository pour Post, Comment, User
- Unit of Work pour les opérations transactionnelles
- Recherche et pagination

### Exercice 4 : Système de notification avec DI
Créez un système de notification supportant :
- Différents canaux (Email, SMS, Slack)
- Configuration par environnement
- Logging intégré
- Tests unitaires

### Exercice 5 : E-commerce avec CQRS
Construisez un système de commande avec :
- Commands pour créer/modifier des commandes
- Queries optimisées pour l'affichage
- Projections pour les rapports
- Gestion d'inventaire

## Combinaison de Patterns

Les patterns architecturaux fonctionnent souvent mieux ensemble :

```csharp
// Exemple : MVC + Repository + DI + CQRS
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly ICommandDispatcher commandDispatcher;
    private readonly IQueryDispatcher queryDispatcher;

    public ProductsController(
        ICommandDispatcher commandDispatcher,
        IQueryDispatcher queryDispatcher)
    {
        this.commandDispatcher = commandDispatcher;
        this.queryDispatcher = queryDispatcher;
    }

    // Command (Write)
    [HttpPost]
    public async Task<IActionResult> CreateProduct([FromBody] CreateProductCommand command)
    {
        await commandDispatcher.DispatchAsync(command);
        return CreatedAtAction(nameof(GetProduct), new { id = command.ProductId }, null);
    }

    // Query (Read)
    [HttpGet("{id}")]
    public async Task<IActionResult> GetProduct(int id)
    {
        var query = new GetProductByIdQuery { ProductId = id };
        var result = await queryDispatcher.DispatchAsync(query);

        if (result == null)
            return NotFound();

        return Ok(result);
    }
}

// Handler avec Repository et Unit of Work
public class CreateProductCommandHandler : ICommandHandler<CreateProductCommand>
{
    private readonly IUnitOfWork unitOfWork;
    private readonly IEventPublisher eventPublisher;

    public CreateProductCommandHandler(IUnitOfWork unitOfWork, IEventPublisher eventPublisher)
    {
        this.unitOfWork = unitOfWork;
        this.eventPublisher = eventPublisher;
    }

    public async Task HandleAsync(CreateProductCommand command)
    {
        await unitOfWork.BeginTransactionAsync();

        try
        {
            var product = new Product(command.Name, command.Price);
            unitOfWork.Products.Add(product);

            await unitOfWork.CommitTransactionAsync();

            // Publier un événement pour Event Sourcing
            await eventPublisher.PublishAsync(new ProductCreatedEvent
            {
                ProductId = product.Id,
                Name = product.Name,
                Price = product.Price
            });
        }
        catch
        {
            await unitOfWork.RollbackTransactionAsync();
            throw;
        }
    }
}
```

## Architecture moderne : Clean Architecture

```csharp
// Couches de Clean Architecture

// 1. Domain Layer (Core)
public class Order
{
    public int Id { get; private set; }
    public string CustomerId { get; private set; }
    public DateTime OrderDate { get; private set; }
    public OrderStatus Status { get; private set; }

    // Business logic ici
    public void AddItem(Product product, int quantity)
    {
        // Validation et logique business
    }
}

// 2. Application Layer
public interface IOrderService
{
    Task<OrderDto> CreateOrderAsync(CreateOrderDto dto);
    Task<OrderDto> GetOrderAsync(int id);
}

public class OrderService : IOrderService
{
    private readonly IOrderRepository repository;
    private readonly IMapper mapper;

    // Implémentation sans dépendances externes
}

// 3. Infrastructure Layer
public class EFOrderRepository : IOrderRepository
{
    private readonly ApplicationDbContext context;

    // Implémentation EF Core
}

// 4. Presentation Layer
[ApiController]
public class OrdersController : ControllerBase
{
    private readonly IOrderService orderService;

    // API endpoints
}
```

## Microservices Architecture

```csharp
// Exemple de microservice avec tous les patterns

// API Gateway
public class ApiGateway
{
    private readonly IServiceDiscovery serviceDiscovery;
    private readonly ILoadBalancer loadBalancer;

    public async Task<IActionResult> RouteRequest(HttpRequest request)
    {
        var service = await serviceDiscovery.FindServiceAsync(request.Path);
        var instance = await loadBalancer.GetInstanceAsync(service);

        // Forward request to microservice
        return await ForwardToService(instance, request);
    }
}

// Service individuel
[ApiController]
[Route("api/[controller]")]
public class OrderMicroserviceController : ControllerBase
{
    private readonly IEventBus eventBus;
    private readonly ICommandDispatcher commandDispatcher;
    private readonly IQueryDispatcher queryDispatcher;

    // Logique du microservice

    [HttpPost]
    public async Task<IActionResult> CreateOrder([FromBody] CreateOrderCommand command)
    {
        await commandDispatcher.DispatchAsync(command);

        // Publier événement pour autres services
        await eventBus.PublishAsync(new OrderCreatedEvent(command.OrderId));

        return Accepted();
    }
}
```

## Tendances modernes

### 1. Domain-Driven Design (DDD)
- Aggregate patterns
- Value objects
- Domain events
- Bounded contexts

### 2. Event-Driven Architecture
- Messages asynchrones
- Eventual consistency
- Saga pattern
- CQRS + Event Sourcing

### 3. Reactive Programming
- Observable streams
- Backpressure handling
- Reactive Extensions (Rx)

### 4. Serverless Architecture
- Functions as a Service
- Event-driven triggers
- Stateless design

## Choix du bon pattern

```csharp
// Guide de décision
public class ArchitectureDecisionGuide
{
    public static string SuggestPattern(ApplicationRequirements requirements)
    {
        if (requirements.IsSimpleWebApp && requirements.TeamSize < 5)
            return "MVC avec Repository simple";

        if (requirements.IsDesktopApp && requirements.HasComplexUI)
            return "MVVM avec PRISM ou ReactiveUI";

        if (requirements.HasComplexBusiness && requirements.NeedsAudit)
            return "CQRS + Event Sourcing";

        if (requirements.IsDistributed && requirements.ScalabilityNeeded)
            return "Microservices avec Event Bus";

        if (requirements.HasMultipleDataSources)
            return "Repository + Unit of Work";

        return "Commencer simple, évoluer selon les besoins";
    }
}
```

## Résumé final

Les patterns architecturaux vous aident à :

1. **Organiser le code** de manière logique
2. **Gérer la complexité** des grandes applications
3. **Faciliter la maintenance** et les évolutions
4. **Améliorer la testabilité** du code
5. **Optimiser les performances** selon les besoins

### Points clés à retenir :

- **Commencez simple** - N'utilisez pas tous les patterns d'un coup
- **Évoluez selon les besoins** - Ajoutez de la complexité quand nécessaire
- **Documentez vos choix** - Expliquez pourquoi tel pattern est utilisé
- **Mesurez l'impact** - Surveillez les performances et la maintenabilité
- **Adaptez au contexte** - Chaque projet est unique

### Pour aller plus loin :

1. **Pratiquez avec des projets réels** pour maîtriser les patterns
2. **Lisez les livres classiques** (Clean Code, DDD, Enterprise Patterns)
3. **Participez à des projets open source** pour voir les patterns en action
4. **Suivez les évolutions technologiques** - les patterns évoluent aussi
5. **Formez-vous continuellement** sur les nouvelles approches architecturales

N'oubliez pas que **les patterns sont des outils, pas des solutions universelles**. Le but est de résoudre des problèmes réels, pas d'appliquer des patterns pour le plaisir. Choisissez toujours la solution la plus simple qui répond à vos besoins.

⏭️
