Berikut adalah panduan untuk membuat aplikasi **CRUD (Create, Read, Update, Delete)** dasar menggunakan **Laravel 12**. Aplikasi ini akan mengelola data **Post** (judul dan konten) dengan fitur tambah, lihat, edit, dan hapus. Saya akan menyertakan kode lengkap dalam format artifact sesuai permintaan.

---

### **Prasyarat**
- PHP 8.2 atau lebih baru.
- Composer terinstal.
- Database (MySQL, PostgreSQL, atau SQLite).
- Laravel 12 terinstal (akan dijelaskan di bawah).

---

### **Langkah-langkah**

#### 1. **Instalasi Laravel 12**
Jalankan perintah berikut di terminal untuk membuat proyek baru:
```bash
composer create-project --prefer-dist laravel/laravel laravel-crud
cd laravel-crud
```

#### 2. **Konfigurasi Database**
Edit file `.env` untuk mengatur koneksi database (contoh menggunakan MySQL):
```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_crud
DB_USERNAME=root
DB_PASSWORD=
```
Buat database `laravel_crud` di MySQL (atau gunakan SQLite untuk kemudahan).

#### 3. **Membuat Model, Migrasi, dan Controller**
Jalankan perintah Artisan untuk membuat model `Post` beserta migrasi dan controller:
```bash
php artisan make:model Post -mcr
```
- `-m`: Membuat file migrasi.
- `-c`: Membuat controller.
- `-r`: Membuat controller dengan method resource (CRUD).

#### 4. **Setup Migrasi**
Edit file migrasi di `database/migrations/xxxx_create_posts_table.php`:
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->id();
            $table->string('title');
            $table->text('content');
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('posts');
    }
};
```

Jalankan migrasi untuk membuat tabel:
```bash
php artisan migrate
```

#### 5. **Setup Model**
Edit file `app/Models/Post.php` untuk menentukan fillable fields:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    use HasFactory;

    protected $fillable = ['title', 'content'];
}
```

#### 6. **Setup Controller**
Edit file `app/Http/Controllers/PostController.php` untuk menangani logika CRUD:
```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\Request;

class PostController extends Controller
{
    public function index()
    {
        $posts = Post::all();
        return view('posts.index', compact('posts'));
    }

    public function create()
    {
        return view('posts.create');
    }

    public function store(Request $request)
    {
        $request->validate([
            'title' => 'required|string|max:255',
            'content' => 'required|string',
        ]);

        Post::create($request->all());
        return redirect()->route('posts.index')->with('success', 'Post created successfully.');
    }

    public function show(Post $post)
    {
        return view('posts.show', compact('post'));
    }

    public function edit(Post $post)
    {
        return view('posts.edit', compact('post'));
    }

    public function update(Request $request, Post $post)
    {
        $request->validate([
            'title' => 'required|string|max:255',
            'content' => 'required|string',
        ]);

        $post->update($request->all());
        return redirect()->route('posts.index')->with('success', 'Post updated successfully.');
    }

    public function destroy(Post $post)
    {
        $post->delete();
        return redirect()->route('posts.index')->with('success', 'Post deleted successfully.');
    }
}
```

#### 7. **Setup Rute**
Edit file `routes/web.php` untuk mendefinisikan rute resource:
```php
<?php

use App\Http\Controllers\PostController;
use Illuminate\Support\Facades\Route;

Route::get('/', function () {
    return view('welcome');
});

Route::resource('posts', PostController::class);
```

#### 8. **Setup Blade Views**
Buat folder `resources/views/posts/` dan tambahkan file Blade berikut:

a. **Index (Daftar Post)** - `resources/views/posts/index.blade.php`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Posts</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container mt-5">
        <h1>Posts</h1>
        @if (session('success'))
            <div class="alert alert-success">
                {{ session('success') }}
            </div>
        @endif
        <a href="{{ route('posts.create') }}" class="btn btn-primary mb-3">Create New Post</a>
        <table class="table">
            <thead>
                <tr>
                    <th>ID</th>
                    <th>Title</th>
                    <th>Content</th>
                    <th>Actions</th>
                </tr>
            </thead>
            <tbody>
                @foreach ($posts as $post)
                    <tr>
                        <td>{{ $post->id }}</td>
                        <td>{{ $post->title }}</td>
                        <td>{{ Str::limit($post->content, 50) }}</td>
                        <td>
                            <a href="{{ route('posts.show', $post) }}" class="btn btn-info btn-sm">View</a>
                            <a href="{{ route('posts.edit', $post) }}" class="btn btn-warning btn-sm">Edit</a>
                            <form action="{{ route('posts.destroy', $post) }}" method="POST" style="display:inline;">
                                @csrf
                                @method('DELETE')
                                <button type="submit" class="btn btn-danger btn-sm" onclick="return confirm('Are you sure?')">Delete</button>
                            </form>
                        </td>
                    </tr>
                @endforeach
            </tbody>
        </table>
    </div>
</body>
</html>
```

b. **Create (Form Tambah Post)** - `resources/views/posts/create.blade.php`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Create Post</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container mt-5">
        <h1>Create Post</h1>
        <form action="{{ route('posts.store') }}" method="POST">
            @csrf
            <div class="mb-3">
                <label for="title" class="form-label">Title</label>
                <input type="text" name="title" id="title" class="form-control" value="{{ old('title') }}">
                @error('title')
                    <div class="text-danger">{{ $message }}</div>
                @enderror
            </div>
            <div class="mb-3">
                <label for="content" class="form-label">Content</label>
                <textarea name="content" id="content" class="form-control">{{ old('content') }}</textarea>
                @error('content')
                    <div class="text-danger">{{ $message }}</div>
                @enderror
            </div>
            <button type="submit" class="btn btn-primary">Save</button>
            <a href="{{ route('posts.index') }}" class="btn btn-secondary">Cancel</a>
        </form>
    </div>
</body>
</html>
```

c. **Show (Detail Post)** - `resources/views/posts/show.blade.php`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Post Details</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container mt-5">
        <h1>{{ $post->title }}</h1>
        <p>{{ $post->content }}</p>
        <a href="{{ route('posts.index') }}" class="btn btn-secondary">Back</a>
    </div>
</body>
</html>
```

d. **Edit (Form Edit Post)** - `resources/views/posts/edit.blade.php`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Edit Post</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container mt-5">
        <h1>Edit Post</h1>
        <form action="{{ route('posts.update', $post) }}" method="POST">
            @csrf
            @method('PUT')
            <div class="mb-3">
                <label for="title" class="form-label">Title</label>
                <input type="text" name="title" id="title" class="form-control" value="{{ $post->title }}">
                @error('title')
                    <div class="text-danger">{{ $message }}</div>
                @enderror
            </div>
            <div class="mb-3">
                <label for="content" class="form-label">Content</label>
                <textarea name="content" id="content" class="form-control">{{ $post->content }}</textarea>
                @error('content')
                    <div class="text-danger">{{ $message }}</div>
                @enderror
            </div>
            <button type="submit" class="btn btn-primary">Update</button>
            <a href="{{ route('posts.index') }}" class="btn btn-secondary">Cancel</a>
        </form>
    </div>
</body>
</html>
```

#### 9. **Jalankan Aplikasi**
Jalankan server Laravel:
```bash
php artisan serve
```
Buka `http://localhost:8000/posts` di browser untuk melihat daftar post. Kamu bisa:
- Menambah post baru (Create).
- Melihat detail post (Read).
- Mengedit post (Update).
- Menghapus post (Delete).

---

### **Penjelasan Tambahan**
- **Bootstrap**: Digunakan untuk styling sederhana. CDN Bootstrap di-include di setiap view.
- **Validasi**: Controller menggunakan validasi untuk memastikan input `title` dan `content` tidak kosong.
- **Flash Message**: Pesan sukses ditampilkan setelah create, update, atau delete.
- **Route Resource**: Menggunakan `Route::resource` untuk membuat rute CRUD secara otomatis (index, create, store, show, edit, update, destroy).

---

### **Langkah Selanjutnya**
- Tambahkan autentikasi dengan `php artisan breeze:install` untuk membatasi akses CRUD hanya untuk pengguna yang login.
- Tambahkan fitur seperti pencarian atau pagination.
- Pelajari Eloquent Relationship jika ingin menambahkan fitur seperti kategori atau komentar.

Jika ada kendala atau kamu ingin menambahkan fitur tertentu (misalnya, styling custom atau fitur tambahan), beri tahu saya!
