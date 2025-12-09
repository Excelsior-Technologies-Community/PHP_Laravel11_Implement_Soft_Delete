# Laravel 11 – Soft Deletes & Restore Functionality  
![Laravel](https://img.shields.io/badge/Laravel-11-orange)
![PHP](https://img.shields.io/badge/PHP-8.2-blue)
![Bootstrap](https://img.shields.io/badge/Bootstrap-5-purple)
![MySQL](https://img.shields.io/badge/Database-MySQL-yellow)

This guide explains how to implement **Soft Deletes**, **Restore Records**, and **Status-based filtering** in a **Product CRUD application** using Laravel 11.

---

# ⭐ Overview  
This project demonstrates:

- Soft delete using Laravel’s built‑in `SoftDeletes` trait  
- Status column (`active`, `deleted`)  
- Hide deleted records from listing  
- Full Product CRUD  
- Single image upload  
- Pagination, price sorting, and search  
- Admin panel layout  
- Laravel Breeze Authentication  

---

# 📦 Folder Structure  
```
project/
│── app/
│   ├── Models/Product.php
│   └── Http/Controllers/ProductController.php
│
│── resources/views/products/
│       ├── index.blade.php
│       ├── create.blade.php
│       └── edit.blade.php
│
│── database/migrations/
│── public/images/
│── routes/web.php
│── README.md
```

---

# 🧱 Step 1 — Install Laravel 11  
```
composer create-project laravel/laravel example-app
```

---

# 🛠 Step 2 — Configure Database  
Edit `.env`:
```
DB_DATABASE=your_db
DB_USERNAME=root
DB_PASSWORD=root
```

---

# 🧱 Step 3 — Create Products Table  
```
php artisan make:migration create_products_table --create=products
```
Columns include: name, details, price, size, color, category, image, timestamps.

---

# 🧱 Step 4 — Add Status Column  
To track soft-deleted records manually.

```
php artisan make:migration add_status_to_products_table --table=products
```

Migration adds:
```
$table->string('status')->default('active');
```

Run migration:
```
php artisan migrate
```

---

# 🧠 Step 5 — Product Model with Soft Deletes  
```php
use Illuminate\Database\Eloquent\SoftDeletes;

class Product extends Model
{
    use HasFactory, SoftDeletes;

    protected $fillable = [
        'name','details','image','size','color','category','price','status'
    ];

    protected $dates = ['deleted_at'];
}
```

---

# 🧠 Step 6 — ProductController Logic

## ✔ Listing Products (Hide Deleted)
```php
$query = Product::where('status', '!=', 'deleted');
```

Supports:
- Search  
- Pagination  
- Sort by price  

---

## ✔ Soft Delete a Record  
```php
public function destroy(Product $product)
{
    $product->update(['status' => 'deleted']);
    $product->delete(); // Soft delete

    return back()->with('success', 'Product deleted successfully.');
}
```

---

## ✔ Restore (If Needed)
```
Product::withTrashed()->find($id)->restore();
```

---

# 🎨 Step 7 — Blade Files

### `/products/index.blade.php`  
Includes:
- Search box  
- Sorting dropdown  
- Pagination  
- Soft delete button  

### `/products/create.blade.php`  
Includes:
- Product form  
- Image upload  

### `/products/edit.blade.php`  
Includes:
- Old + new image preview  
- Update form  

---

# 🌐 Step 8 — Add Routes  
```php
Route::resource('products', ProductController::class);
```

---

# 🔐 Step 9 — Admin Authentication (Laravel Breeze)

```
composer require laravel/breeze --dev
php artisan breeze:install blade
npm install && npm run dev
php artisan migrate
```

Protect CRUD:
```php
Route::middleware(['auth'])->group(function () {
    Route::resource('products', ProductController::class);
});
```

Set login redirect:
```
public const HOME = '/products';
```

---

# 🖼 Step 10 — Admin Layout  
Includes Bootstrap + Navigation + Page container.

---

# ▶ Run Application  
```
php artisan serve
```
Visit:
```
http://localhost:8000/products
```

---<img width="676" height="163" alt="image" src="https://github.com/user-attachments/assets/637f3b4b-c20a-44c7-9872-a7f084a78d92" />
<img width="676" height="111" alt="image" src="https://github.com/user-attachments/assets/8f6e3627-04d6-4386-89dc-37323602a4e6" />

