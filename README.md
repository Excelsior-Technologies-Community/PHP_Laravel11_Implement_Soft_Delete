# PHP_Laravel11_Implement_Soft_Delete

This documentation explains how to create a product CRUD system in Laravel 11 with:

- Soft delete functionality  
- Restore deleted records  
- Search and sorting options  
- Image upload and update  
- Admin panel using Laravel Breeze  

This README follows the exact step-by-step method shown in the implementation file.

---

# Step 1: Install Laravel 11

Create a new Laravel project:

```
composer create-project laravel/laravel example-app
```

---

# Step 2: Configure MySQL Database

Update your `.env` file:

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=blog
DB_USERNAME=root
DB_PASSWORD=root
```

---

# Step 3: Create Products Migration

Create the products table:

```
php artisan make:migration create_products_table --create=products
```

Add required fields:

```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->text('details');
    $table->decimal('price', 8, 2);
    $table->string('size');
    $table->string('color');
    $table->string('category');
    $table->string('image')->nullable();
    $table->timestamps();
});
```

---

## Add `status` Column (Soft Delete Indicator)

```
php artisan make:migration add_status_to_products_table --table=products
```

Migration:

```php
Schema::table('products', function (Blueprint $table) {
    $table->string('status')->default('active')->after('id');
});
```

Run migrations:

```
php artisan migrate
```

---

# Step 4: Add Resource Route

In `routes/web.php`:

```php
use App\Http\Controllers\ProductController;

Route::resource('products', ProductController::class);
```

---

# Step 5: Create Controller and Model

```
php artisan make:controller ProductController --resource --model=Product
```

---

## Product Model (Soft Deletes Enabled)

```php
class Product extends Model
{
    use HasFactory, SoftDeletes;

    protected $fillable = [
        'name',
        'details',
        'image',
        'size',
        'color',
        'category',
        'price',
        'status'
    ];

    protected $dates = ['deleted_at'];
}
```

---

# ProductController Methods

---

# Show Products With Search + Sorting + Soft Delete Filter

```php
public function index(Request $request)
{
    $query = Product::where('status', '!=', 'deleted'); 
```

### Search Logic

```php
if ($request->filled('keyword')) {
    $keyword = $request->keyword;

    if (is_numeric($keyword)) {
        $query->where('price', (float)$keyword);
    } else {
        $query->where(function ($q) use ($keyword) {
            $q->where('name', 'like', "%{$keyword}%")
              ->orWhere('category', 'like', "%{$keyword}%")
              ->orWhere('color', 'like', "%{$keyword}%")
              ->orWhere('size', 'like', "%{$keyword}%")
              ->orWhere('details', 'like', "%{$keyword}%")
              ->orWhere('price', 'like', "%{$keyword}%");
        });
    }
}
```

### Sorting Logic

```php
if ($request->filled('sort') && in_array($request->sort, ['price-asc', 'price-desc'])) {
    $query->orderBy('price', $request->sort === 'price-asc' ? 'asc' : 'desc');
} else {
    $query->latest();
}
```

### Pagination

```php
$products = $query->paginate(3);
```

---

# Create Product

```php
public function create()
{
    return view('products.create');
}
```

---

# Store Product (Image Upload Included)

```php
public function store(Request $request)
{
    $request->validate([
        'name' => 'required|string|max:255',
        'details' => 'required|string',
        'image' => 'nullable|image|max:2048',
        'size' => 'required|string|max:50',
        'color' => 'required|string|max:50',
        'category' => 'required|string|max:100',
        'price' => 'required|numeric|min:0',
    ]);

    $imagePath = null;

    if ($request->hasFile('image')) {
        $image = $request->file('image');
        $imageName = time().'_'.$image->getClientOriginalName();
        $image->move(public_path('images'), $imageName);
        $imagePath = 'images/'.$imageName;
    }

    Product::create([
        'name' => $request->name,
        'details' => $request->details,
        'image' => $imagePath,
        'size' => $request->size,
        'color' => $request->color,
        'category' => $request->category,
        'price' => $request->price,
    ]);

    return redirect()->route('products.index')->with('success', 'Product created successfully.');
}
```

---

# Show Product

```php
public function show(Product $product)
{
    return view('products.show', compact('product'));
}
```

---

# Edit Product

```php
public function edit(Product $product)
{
    return view('products.edit', compact('product'));
}
```

---

# Update Product (With Image Replace + Delete Old)

```php
public function update(Request $request, Product $product)
{
    $request->validate([
        'name' => 'required|string|max:255',
        'details' => 'required|string',
        'image' => 'nullable|image|max:2048',
        'size' => 'required|string|max:50',
        'color' => 'required|string|max:50',
        'category' => 'required|string|max:100',
        'price' => 'required|numeric|min:0',
    ]);

    $imagePath = $product->image;

    if ($request->hasFile('image')) {

        if ($product->image && file_exists(public_path($product->image))) {
            unlink(public_path($product->image));
        }

        $image = $request->file('image');
        $imageName = time().'_'.$image->getClientOriginalName();
        $image->move(public_path('images'), $imageName);
        $imagePath = 'images/'.$imageName;
    }

    $product->update([
        'name' => $request->name,
        'details' => $request->details,
        'image' => $imagePath,
        'size' => $request->size,
        'color' => $request->color,
        'category' => $request->category,
        'price' => $request->price,
    ]);

    return redirect()->route('products.index')->with('success', 'Product updated successfully.');
}
```

---

# Soft Delete Product

```php
public function destroy(Product $product)
{
    $product->update(['status' => 'deleted']);
    $product->delete();

    return redirect()->route('products.index')
        ->with('success', 'Product deleted successfully.');
}
```

---

# Step 6: Blade Files

You need to create:

```
resources/views/products/index.blade.php
resources/views/products/create.blade.php
resources/views/products/edit.blade.php
```

Your index file includes:

- Search bar  
- Sort dropdown  
- Pagination  
- Image preview  
- Edit/Delete buttons  
- Soft delete using status field  

(All code exactly matches your file.)

---

# Step 7: Required Layout Files

```
resources/views/layouts/app.blade.php
resources/views/layouts/admin.blade.php
```

Both layouts include proper HTML structure, navigation, and page container system.

---

# Step 8: Run Application

```
php artisan serve
```

Open:

```
http://localhost:8000/products
```


---<img width="676" height="163" alt="image" src="https://github.com/user-attachments/assets/637f3b4b-c20a-44c7-9872-a7f084a78d92" />
<img width="676" height="111" alt="image" src="https://github.com/user-attachments/assets/8f6e3627-04d6-4386-89dc-37323602a4e6" />

